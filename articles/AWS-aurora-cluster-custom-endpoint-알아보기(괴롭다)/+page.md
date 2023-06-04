---
title: AWS aurora cluster custom endpoint 알아보기(괴롭다)
summary: Aurora custom endpoint
publishedAt: 2023-06-04 16:59
thumbnail: 
---

## 상황

서버에 장애가 발생했다. 원인은 사내에서 사용하고 있던 aurora cluster의 automatic minor update 옵션이 켜져있었고, minor update 가 진행되면서 writer instance 가 중지된 것이다.
	- 다만 이 상황에서는 read replica 메커니즘이나 failover 가 정상적으로 동작했어야 하지만 모든 read 요청이 fail 했다. (minor update 완료 이후에도 계속)
	- 이 상황에서 예상되는 시나리오는 다음과 같았다
		- (전제) 우리 서버는 read replica 를 custom endpoint 를 통해 접속하고 있다
		1. read instance 가 writer instance 가 되면서 custom endpoint 에서 빠졌다
		2. custom endpoint 설정에서 exclusion list 로 기존 writer instance를 설정해둔 상태였다
		3. 따라서 read instance 로 역할이 변경된 writer 였던 instance 가 endpoint 에서 제외되면서 endpoint 에 연결된 instance 가 없어져 모든 요청을 처리할 수 없었다
	- 하지만 위 시나리오일 확률이 높지 않다고 생각했던 이유들이 몇가지 있는데,
		1. custom endpoint 에서 reader로 지정된 instance들만 가져오는게 맞는지 모른다
		2. failover된 이후에는 그럼 writer였던 instance가 reader로 바뀌는 경우에도 해당 instance를 엔드포인트에 넣어주는지, 그렇지 않은지 모른다 
			- cluster endpoint 는 이런 기능을 지원해주는 것으로 알고 있었는데, custom endpoint 는 이걸 안해주는게 이상하다고 생각했다

## Custom endpoint 관련 문서 정독

- custom endpoint는 cluster db instance 의 연결을 제어하고 간편하게 사용하기 위한 목적으로 사용한다
  - 엔드포인트에 등록된 특정 인스턴스들의 그룹을 대상으로 로드밸런싱을 해준다
- 어떤 인스턴스들이 endpoint와 연관되는지 결정할 수 있는 Type이 있다
  - (READER, WRITER, ANY)
- Type은 AWS management console 에서 지정할 수 없으며 management console에서 생성하는 모든 엔드포인트는 ANY 타입으로 생성된다
  - AWS CLI/ RDS API 를 통해서만 가능
  - READER 타입은 **read-only Replica 만** 포함된다
    - 그럼 writer로 변경된 instance는 자동으로 endpoint 에서 사라지는건가? 
      - (추가) 그렇다. 다만 static list, exclusion list 에 정의된 것들은 어떤 방법으로 custom endpoint를 만들었느냐에 따라 다르다
  - ANY 에서는 RO, RW 가 포함된다. 하지만 복제본의 원본 데이터 베이스에 연결하는지 미리 지정할 수 없으니 RO 상황일 때에만 이 타입을 사용해야 한다
  - replication config에 알맞지 않은 type을 사용하면 에러가 난다
    - replication을 사용하지 않는 클러스터에서 ANY 타입의 커스텀 엔드포인트를 만드는 경우
- static list, exclusion list 를 사용해서 포함하거나 제거할 instance를 지정할 수 있다
- Management console 에서 설정할 수 있는 것들
  - **Attach future instances added to this cluster**
    - 체크하지 않으면 white list 방식으로, 체크하면 black list 방식으로 돌아간다
    - 체크하여 exclusion list 를 사용할 경우, 이후에 failover를 통해 ReadReplica가 새롭게 뜨는 것 같은 경우에 자동으로 custom endpoint에 해당 instance가 추가된다
    - static list 를 사용하면 failover 되거나 새로운 instance 가 생성되는 경우 endpoint 의 member가 업데이트 되지 않는다
  - management console에서 만든 커스텀 엔드포인트는 failover, promotion으로 인해 instance의 타입이 RR, writer 간에 변경되면 static list 나 exclusion list 를 업데이트한다
- CLI, RDS API 로 할 수 있는 것들
  - Management console 에서와는 다르게, failover, promotion으로 인해 instance의 타입이 RR, writer 간에 변경되면 static list 나 exclusion list 를 업데이트 하지 않는다

## 문제

- 문서에 명시된 항목이 서로 모순된다고 생각했다.
  - management console 에서는 ANY 타입의 custom endpoint만 생성할 수 있는데, custom endpoint 가 READER 인 경우를 예시로 들고 있다
    - 일단 만들고 RDS API나 AWS CLI 를 사용해서 변경할 수 있는 것 같기도..
  - `READER 타입은 **read-only Replica 만** 포함된다` 라는 문장이 있는데, AWS CLI, RDS API 로 만들어진 custom endpoint 에서는 failover 가 되어도 static list, exclude list 가 변경되지 않는다고 한다. 모순이 아닌가?
    - (추가) *시간이 지나서 다시 생각해 보니, static list, exclusion list 만 변경하지 않는 것이지, 여기에 등록되지 않은 것들은 type 에 결정된 rule 을 따라 잘 변경될 것 같다 (예상...)*
- RDS API, AWS CLI 를 사용한 custom endpoint 와 Web Console 을 사용해 만든 custom endpoint의 동작이 다르다
  - 가장 혼란스러운 부분인데, failover, promotion 을 통해 변경된 instance가 endpoint 에 존재하는지에 대한 여부가 완전히 반대로 동작한다
    - (추가) *아마도 RDS API, AWS CLI 는 테라폼과 같은 IaaC 를 지원하기 위해 최대한 인프라 관련 변수가 변하지 않도록 설계한 것 같다*
  - 문서에서는 RDS API, AWS CLI 를 사용해 만들어진 custom endpoint 는 역할이 변경되더라도 제거되지 않으며, 대신 endpoint의 type을 사용하여 동작을 제어한다고 하는데, RW 모드가 된 instance 에 READ 요청이 도달하여 정상적으로 동작하는지 그렇지 않은지 명확하게 작성되어 있지 않다
    - 이 부분이 가장 문제가 된다. 만약 writer 로 역할이 변경된 instance 에도 요청/응답을 받을 수 있는 상황이였다면 custom endpoint 관련해서 발생한 문제와 예상했던 시나리오가 완전히 어긋나기 때문
    - (추가) 아마도 위에서 언급한 것 처럼 static, exclusion list 만 그대로 존재하고 writer 로 변경된 instance 는 custom endpoint type 이 READER 였으니 read replica 만 존재할 수 있는 rule 에 따라 제외된 것으로 생각하면 설명이 된다.

## 결론

- 문서는 영어로 읽는게 덜 헷갈린다
- 시간이 지나고 다시 생각해보니 RDS API, AWS CLI 를 통해 만들어진 custom endpoint 들은 instance 의 역할이 바뀌더라도 연결된 instance 를 변경하지 않는다고 이해했었는데, custom endpoint 에 등록된 static list, exclusion list 를 바꾸지 않는다는 말이니, 기존 type 에 설정된 rule 에 따라 writer 가 잘 제외되었을 것 같고 예상했던 장애 시나리오가 맞을 확률이 높아졌다.
- AWS 는 대충 다 계획이 있구나..

## TODO
- 그래도 아직 장애 시나리오가 맞는지 확신이 안서니 test 서버에서 cluster를 띄워보고 failover 시나리오를 재현해볼 예정이다

### 장애 재현
- 방법
  - cluster를 하나 만들고 writer, read replica 를 생성한다
  - writer를 장애 조치한다
- failover test 에서 확인해야 할 것들
  1. READER로 설정한 경우 writer가 죽고 reader가 승격된 상황에서 얼마나 큰 다운타임이 생기는가?
    - (결과) 장애 유형에 따라 다른 것 같아서 측정 불가
  2. READER로 설정한 경우 RR 이 생기면 자동으로 endpoint에 잘 추가되는가?
    - (결과) ReadReplica 가 writer로 변경되고, failover 된 instance(writer 였던 instance)가 ReadReplica로 작동하도록 변경된다
  3. ANY로 설정한 경우 writer가 죽고 reader가 승격된 상황에서 요청이 정상적으로 전달되는 상태인가?
    - (결과) 사실 걱정할 필요가 없던 부분이다. 2개의 instance 가 동시에 죽는 상황이 아니면 괜찮을 것
  
## 앞으로 어떻게 대처할 것인가?

- 전제) 현재는 Read endpoint 를 가져와 writer(provisioning 될 때 당시의 writer instance)와 다른 데이터베이스 하나를 더 제외한다
  - READER 타입이라 reader만 가져오는데, 존재하지도 않는 writer 를 제외하고 있기는 하다...
- 전제1) RR은 하나만 뜨고 있는 상태이다

1. 만든 endpoint 를 ANY 타입으로 변경하자
  - 장애가 발생하면 read replica 가 writer로 승격되고, writer가 read replica가 되는데, 이 때 read replica로 변경된 것이 제외될 것이다 (RDS API로 만들었기 때문에)
    - 이 때문에 장애 발생 이후의 디버깅이 어려워지지는 않을까 두려웠다 (나의 기우였던걸로...)
  - spring 에서 특정 트랜잭션에 annotation을 붙이면 read replica 로만 요청을 전송하게 하고 있는데, 이렇게 되면 장애 발생 이후에는 read replica 로만 전송하는게 아닌 것이 되어버려서 코드로는 알 수 없는 암묵적인 동작이 생겨버리기 때문에 마음에 들지 않는 상태
2. 굳이 writer 를 제외하지 말자
  - WRITER가 죽고 RR이 WRITER로 승격된 상황에서 data, WRITER(RR이였던 것) 만 남는 상황에는 failover될 동안 다운타임이 생길 수 있다.
  - RR이 2개 이상 존재한다면 이 방법이 적절한 것 같다.

이번에는 일단 오는 주말에 aurora cluster update 가 예정되어 있다고 메일이 온 탓에 가장 안정적으로 클러스터를 굴릴 수 있도록 1번 방법을 선책했다. 다만 tfstate 와 현재 infra 상태가 어긋난 상태가 있어서 일단은 cli로 변경하고 tfstate 부터 복구하기로 했다... IaaC 에 최적화된 인프라 관리에 좀 더 익숙해져야 할 듯