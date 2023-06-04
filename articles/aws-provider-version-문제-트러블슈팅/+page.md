---
title: aws provider version 문제 트러블슈팅
summary: 
publishedAt: 2023-06-04 17:13
thumbnail: 
---

## 문제 상황

- 사내 인프라 관리를 terraform 으로 하고 있는데, 갑자기 deprecated 된 keyword, property 들이 우수수 쏟아져 나왔다..

## 원인 파악

- 문제 상황 당시 8시간 전에 aws provider 버전이 5.0.0 이상으로 올라간 것을 확인했다.
  - 아마도 버전이 올라가면서 deprecate 된 것을 사용한 것 같다.
- 그럼 우리 terraform runner는 왜 버전이 올라간 것인가..?
  - .terraform.lock.hcl 에는 3.48.0 버전으로 명시되어 있다.
    - 3.48.0 버전으로 init 되고 있는 걸까?
    - 하지만 이 lock 파일은 특정 모듈 하나에서만 동작하고 있어서 전역적인 설정이 아니다.
  - remote 저장소에 들어있는 tfstate 는 aws provider 를 `registry.terraform.io/hashicorp/aws` 로 사용하고 있는데, 버전이 따로 명시되어있지 않다
    - 아마 항상 latest 버전을 사용한 것 같고, deprecate 된 것들이 생김에 따라 문제가 발생한 듯 하다

## 문제

- infra 코드상에서 모듈별로 aws provider 를 따로 불러와 사용하는데, 버전이 하나도 명시된 곳이 없다
  - aws provider 를 하나로 모아서 사용하게 하거나 버전만 따로 고정시켜주는게 필요할 듯 하다
- terraform 의 0.13.0 버전 문법과 0.12.0 문법이 혼용되어 있다
  - `provider` 블록과 `required_providers` 이 섞여서 사용되고 있음

## 해결 방안

- 일단 aws provider 버전을 고정하거나 항상 latest 버전으로 사용하는 것 중 택일해야 한다
  - 항상 latest 버전을 사용하는 것은 유지보수 코스트가 좀 많이 들어가는 것 같아, 버전을 고정하기로 했다
- 현재 코드에 호환 가능한 aws provider 버전 중 가장 높은 버전은 4.67.0 이였다. 따라서 aws provider 블럭들에 버전을 명시해 주었다

## TODO

- 지금은 버전이 모듈마다 전부 다른데, 한 곳으로 모아 version을 명시해두어 하나의 버전으로 돌아가게 만들어야겠다.