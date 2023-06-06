---
title: Typescript 간단 정리 (ongoing)
summary: 
publishedAt: 2023-06-06 19:23
thumbnail: 
---

## 개요

블로그 유지보수, 사이드 프로젝트 등에서 TypeScript 를 쓸 일이 많이 생겨서 제대로 알고 쓰는게 좋을 것 같아 간단하게 정리하고 boiler template 까지 만드는게 목표이다. 

요즘 자꾸 흥미 위주로만 공부한다는 생각이 들어서 짧게 3시간 이내로 정리하는 것도 부가적인 목표.

- 참고 자료
  - [The Typescript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

## Tutorials

### Basics

```js
// Accessing the property 'toLowerCase'
// on 'message' and then calling it
message.toLowerCase();
// Calling 'message'
message();
```

위 자바스크립트 코드에서는 `toLowerCase` 라는 property를 접근한 후 호출했고 그 후 `message` 를 직접 호출했다

하지만 코드를 보는 사람들의 입장에서, message가 어떤 값인지 추측할 수 없다. 따라서 이 코드는 어떤 결과를 만들어낼지 확실하게 말할 수 없다. 모든 동작은 맨 처음 message가 어떻게 생성되었는지에 의존하여 달라진다.

```js
const message = "Hello World!";
```

`toLowerCase()` 를 통해 message가 string이라고 추측해볼 수 있다. 하지만 바로 뒤에서 `message()` 를 실행하려고 하면 `message is not a function` 이라는 에러가 출력될 것이다.

```js
function fn(x) {
  return x.flip();
}
```

이 함수를 읽어보면 fn 은 parameter 로 전달받은 x 의 `flip` property 에 접근하여 호출하는 것을 알 수 있다. 

하지만 fn이 어떤 파라미터를 받는 것을 의도하고 작성되었는지 알 수 없다. 또한 의도한 대로 x.flip 을 하면 어떤 결과를 출력하는지도 알 수 없다.

결국 위는 javascript 가 dynamic type을 사용하면서 발생하는 문제들의 나열이고, dynamic type 을 사용하는 장점도 물론 있지만 개인적으로는 위의 단점이 치명적이라고 생각한다. 따라서 대안은 static type 을 사용하는 것이 된다.

#### Static Type Checking

Typescript는 type checking 을 함으로서 dynamic type 이 불편한 사람들의 needs를 충족한다

```ts
const message = "hello!";
 
message();
^^^^^^^ This expression is not callable. Type 'String' has no call signatures.
// 컴파일 타임 or 그 전에 위와 같은 에러가 뜰 것
```

런타임에 message 에 알 수 없는 type의 값이 지정되고 직접 호출된다면 비즈니스 로직 동작 중에 알 수 없는 에러를 뱉게 된다. 이상적으로는 이런 사소한 버그를 compile 되는 시점에 알 수 있는 것이 좋고, static type checking 이 이런 일을 해준다.

#### Non-exception Failures

```ts
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```

[the ECMAScript specification](https://tc39.github.io/ecma262/) 에서는 'callable 하지 않은 것을 call 하려 하면 에러를 발생시켜야 한다' 라고 한다. 이 항목은 명백한 에러처럼 보일 수 도 있지만, property가 없는 경우에 해당 property에 접근하는 것도 에러를 내야하는 것이 아닌가? 라는 생각을 하게 될 것이다. 이런 경우에 js 는 위 코드처럼 undefined 를 return할 뿐이다.

```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
user.location;
     ^^^^^^^^ Property 'location' does not exist on type '{ name: string; age: number; }'.
```

궁극적으로 static type system은 위와 같은 잘못된 접근을 허용하지 않고 compile time 에 잡아줌으로서 다음과 같은 아찔한 에러들을 자동으로 잘 잡아준다. 

```js
const announcement = "Hello World!";
 
// How quickly can you spot the typos?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();
 
// We probably meant to write this...
announcement.toLocaleLowerCase();
```

#### Types for Tooling

위 예제에 포함된 실수들을 감지하는 것도 좋지만, ts 는 이를 예방해줄 수도 있다. type-checker 가 어떤 properties 에 접근할 수 있는지 알고 있기 때문에 접근할 수 있는 properties 를 제안해줄 수 있다. 

개인적으로는 js를 사용하면서 dynamic type을 사용하는 것 보다는 ts 를 사용하면서 static type 을 사용하고 suggestion 을 받는게 훨씬 생산성이 좋았다.

#### tsc - TypeScript Compiler

`npm install -g typescript` 으로 tsc 를 다운받을 수 있다. tsc 는 ts 코드를 js 로 컴파일 해준다.

#### Explicit Types

```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```

위 함수처럼 person, date 라는 이름으로 넘겨받는 파라미터의 타입을 명시해줄 수 있다. 타입이 적절하지 않은 경우에는 compiler 에서 error를 적절하게 뱉어준다

```ts
// Date() 는 js 로 구현되어 있고, 반환값은 string 이다
greet("Maddison", Date());
                  ^^^^ Argument of type 'string' is not assignable to parameter of type 'Date'.
```

#### Strictness

ts 를 사용하는 사람들 중 js 에서 migration 하는 사람들은 좀 더 느슨한 타입 체킹을 필요로 할 수 있다.(잠재적인 null, undefined 들의 허용을 위해)

대조적으로 많은 사람들이 ts 가 좀 더 strict한 체크를 해주는걸 원할 수 있다. ts 는 몇가지 strictness flag 를 가지고 있고, 이것들을 tsconfig.json 에 설정함으로서 켜고 끌 수 있다. 대표적으로 많이 사용되는 두가지 flag는 다음과 같다

- noImplicitAny
  - ts 는 타입을 추론할 수 없는 경우 `any` 라는 포괄적인 타입으로 추론해버리는 경우가 있다.
    - 이 경우에는 js 와 거의 동일한 사용 경험을 할 수 있다 (타입 체킹이 안됨..)
  - 옵션을 켜면 ts 는 `any` 타입에 대해 추론하지 않고 에러를 발생시킨다
- strictNullChecks
  - 기본적으로 null, undefined들은 다른 타입으로 할당될 수 없다. 하지만 null, undefined 가 발생할 수 있는 시나리오에서 적절한 핸들링을 해주지 않으면 버그 천국에 빠질 수 있다.
  - 이 옵션을 켜면 null, undefined 에 대한 핸들링을 좀 더 명시적으로 만들고, 핸들링하는 것을 까먹지 않도록 돕는다.

### Types

#### Primitives

- string
  - `"Hello world"` 같은 값
- number
  - `42` 와 같은 값. js runtime은 integer, float을 구분하지 않으므로 둘 다 number 이다
- boolean
  - `true, false`

> String, Number, Boolean 을 사용할 수도 있지만 이것들은 primitive 타입이 아니라 reference type 이다.

```ts
var s1 = new String("Avoid newing things where possible");
// var s2 = String("A string, in TypeScript of type 'string'") 과 같다
var s2 = "A string, in TypeScript of type 'string'";
var s3: string;

// 결론
var str: String = new String("Hello world"); // Uses the JavaScript String object
var str: string = String("Hello World"); // Uses the TypeScript string type
```
출처: https://stackoverflow.com/questions/14727044/what-is-the-difference-between-types-string-and-string

#### Arrays

- `[1, 2, 3]` 같은 Array 의 타입을 명시할 때에는 `number[]` 와 같이 사용한다.
- `Array<number>` 로 사용할 수도 있다.

#### Any

```ts
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

- ts 의 특별한 타입이다. 이 타입은 특정한 값에 대한 타입 체킹을 하고 싶지 않을 때 사용한다.
- any로 설정된 타입에서는 존재하지 않는 property에 접근할 수도 있고, 호출하거나 할당할 수도 있다
- js 와 동일한 경험을 준다.
- js 로 작성된 모듈들이 보통 Any 타입을 사용하고 있다. 
  - 해당 모듈의 type 을 따로 의존성으로 추가하면 타입 체킹을 해준다. 다만 없는 경우에는 그냥 any 로 사용해야 할 때도 있다..
  - ex. `npm i --save-dev @types/swagger-jsdoc`

#### Type Annotations on Variables

- const, var, let 을 사용해서 변수를 선언할 때에 명시적으로 어떤 타입인지 표시할 수 있다.

```ts
let myName: string = "Alice";

// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
```

- 보통 위와 같은 것들은 필요하지는 않다. 타입스크립트가 코드상의 타입을 추측해보기 때문.
- 개인적으로는 의도한 타입을 잘 명시해두는 편이다.
  - type infer 가 의도치 않게 동작하는 경우를 막기 위해 (값 할당을 잘못한다던지..)

#### Functions

```ts
// Parameter type annotation
function greet(name: string): number {
  console.log("Hello, " + name.toUpperCase() + "!!");
  return 23;
}
```

- ts의 functions 는 js 의 값을 전달하고 output 을 뱉는 구조와 동일하다
- 대신 paramter에 타입을 정의해줄 수 있다.
  - parameter에 타입을 정의해주면 argument 의 타입이 잘 체크된다
- 또한 return type 도 명시해줄 수 있다.

```ts
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

- Anonymous Functions
  - 함수 선언과는 조금 다르게 ts에서 어떻게 호출되는지 파악할 수 있다면, parameter 들의 타입이 자동으로 부여된다
  - 위에서 s는 따로 타입을 지정해주지 않았지만, names 를 순회하고 있으므로 string 타입인 것을 알 수 있다.

#### Object Types

```ts
// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

- 보통 가장 자주 마주치는 타입은 이것이다.
- property를 가진 js value 들을 뜻한다

```ts
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
                       ^^^^^^^^^^^ 'obj.last' is possibly 'undefined'.

  // 위와 같은 에러가 발생하니 아래처럼 에러 핸들링을 해줘야 한다.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```

- 위의 `last?` 처럼 optional한 값이라는 표시를 할 수도 있다.
- js 에서 값이 없거나 존재하지 않는 property에 접근하면 undefined 가 튀어나오는데, 이 이후에 특정 함수나 property를 접근하면 에러가 날 것이다.
- 따라서 이런 optional value 들을 핸들링해주는 과정이 필요하다.
  - ts 는 잠재적으로 undefined 가 될 수 있는 값들을 체크해두고, 에러 핸들링을 잘 할 수 있게 돕는다.

#### Union types

- ts 의 타입 시스템은 하나의 값이 여러 타입일 수 있다는 추론을 가능하게 해준다.

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
        ^^^^^^^^^^^^^^^ Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
```

- 여러 타입으로 명시해주는건 쉽지만, 다음과 같은 상황에서는 좀 곤란하다.

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
                 ^^^^^^^^^^^
                  Property 'toUpperCase' does not exist on type 'string | number'.
                  Property 'toUpperCase' does not exist on type 'number'.
}
```

- 위와 같은 상황에서는 string 에만 `toUpperCase` 가 존재하기 때문에 에러가 난다.
  - `narrowing` 을 통해서 해결할 수 있다.

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

- id 의 타입을 체킹하면서 어떤 타입인지 좁혀가는 방식으로 해결할 수 있다.

#### Type Aliases

```ts
type Point = {
  x: number;
  y: number;
};

type Id = number | string;
 
// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

- Point, Id 처럼 특정 object 나 union type을 하나의 이름으로 alias 할 수 있다.
- alias 는 alias 일 뿐이다. alias 된 타입과 별개로 취급되는 type은 아니다.

```ts
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// Create a sanitized input
let userInput = sanitizeInput(getInput());
 
// Can still be re-assigned with a string though
userInput = "new input";
```

#### Interfaces

- object type에 이름을 지어 선언하는 방법 중 하나이다

```ts
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

- type alias 처럼 anonymous object 에 이름을 지어줄 수 있다. 
- ts 에서는 어떤 property에 접근할 수 있는지 그 구조에 대해서만 체크한다.
  - ts 가 왜 structurally typed system 이라고 불리는지에 대한 이유

```ts
// interface

interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;

// type

type Animal = {
  name: string;
}

type Bear = Animal & { 
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```

- type 키워드와 다른 점
  - 확장할 때에 extends 를 사용한다 (type은 intersaction 한다)
  - interface는 런타임에서 재정의, 확장이 가능하다 (type은 재정의할 시 에러가 발생한다)

```ts
// interface
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});

// type

type Window = {
  title: string;
}

type Window = {
  ts: TypeScriptAPI;
}

 // Error: Duplicate identifier 'Window'.
```

#### Type Assertions

- 어떤 값의 타입에 대한 정보를 타입스크립트가 알 수 없는 경우가 있다.
  - document.getElementById 에 대해 ts 는 HTMLElement 로만 추론한다.
  - 하지만 작성자는 HTMLCanvasElement 가 들어와야 하는걸 아는 경우가 있다.
- 위와 같은 경우에 `as` 를 사용해 Type Assertion 을 사용할 수 있다.

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

> Type Assertion 은 runtime 에서는 작동하지 않는다. 따라서 런타임에 type assertion 과 다른 타입이 들어와도 에러가 나지 않고 null 이 생성된다.

- 대신 type assertion 은 불가능한 상황에서는 컴파일 타임 에러를 만들어준다.

```ts
const x = "hello" as number;
                  ^^^^^^^^^ 
                  Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
```

#### Literal Types

- string, number 를 사용하는 대신, 특정 문자열과 숫자를 타입으로서 사용할 수 있다.

```ts
let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
// let changingString: string
changingString;
 
const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
// const constantString: "Hello World"
constantString;
```

- changingString은 어떤 값이난 들어갈 수 있으므로 기본적인 string 이 된다.
- constantString 은 "Hello World" 라는 값 1개만 가능한 상태가 되므로 literal type 으로 설정된다.

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
```

- 정리하면 literal Type 은 특정 값이 특정 값만 할당될 수 있다는 것이다.
- 이 자체로는 쓸데 없다고 생각할 수 있지만, union type 과 같이 사용하면 굉장히 가치있는 문법이 된다.

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
                          ^^^^^^
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

Numeric literal type 도 비슷하게 동작하고 선언할 수 있다.

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

물론 non-literal type 과도 혼용할 수 있다.

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
           ^^^^^^^^^
           Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

#### Literal inference

```ts
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

- object 안에 변수를 초기화하면, ts 는 그 값이 이후에 바뀔 수 있다고 추측한다.
  - 따라서 위와 같은 코드가 가능한 것이다.
- literal type 을 사용할 때에도 비슷하게 동작한다

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
 
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

- req.method 는 string으로 취급되고, "GET" 이라는 literal로 취급되지 않는 문제가 있다.
  - 이를 해결하는 두가지 방법이 있다.
    - method 가 literal 임을 명시해준다
    - req 가 const 임을 명시해준다.

```ts
// req.method as literal

const req = { url: "https://example.com", method: "GET" as "GET" };
handleRequest(req.url, req.method as "GET");

// req as const

const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

#### null and undefined

- js 에서는 두 타입을 존재하지 않는 값이거나 초기화되지 않은 값으로 다룬다
- ts는 같은 이름의 null, undefined 가 존재하지만, strictNullChecks 옵션에 따라 다르게 동작한다

- strictNullChecks: off
  - null, undefined는 접근 가능하고 any 타입으로서 할당될 수 있다. 
  - null checking이 불가능한 언어들과 비슷하게 동작한다
  - 이러한 null checking 의 부재는 버그 발생의 큰 요인으로서 동작한다.
  - 따라서 이 옵션을 켜는 것을 권장한다
- strictNullChecks: on
  - 어떤 값이 null, undefined 일 때, 이 값들에 대한 체킹을 해야 한다. (narrowing)

```ts
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

- Non-null Assertion Operator
  - `!` 를 씀으로서 null 이나 undefined 인 값이 더이상 null 이 아니라고 선언할 수 있다.

### Narrowing

- types 에서 언급했던 narrowing의 디테일한 설명이다.
- union type 같은 곳에서 범위를 좁혀가며 특정 값에 대한 타입을 추론할 수 있는 상태로 만드는 것

#### typeof guarding

- js 는 runtime 시점에 특정 값의 기본적인 정보를 알아낼 수 있는 typeof 를 지원한다
- string 을 return 한다. 
  - "string"
  - "number"
  - "bigint"
  - "boolean"
  - "symbol"
  - "undefined"
  - "object"
  - "function"

하지만 typeof 는 null을 구분해주지 않는다. `typeof null` 을 실행하면 "object" 가 튀어나온다... 따라서 typeof 만으로 null 을 다루는 것은 힘들고, 'truthiness checking' 을 알아보는 것이 좋다.

#### Truthiness narrowing

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

- 위처럼 if-else 를 통해 특정한 조건을 boolean 로 만들고 값에 따라 분기를 태울 수 있는데, 
- 항상 false 인 녀석들이 있다.
  - 0
  - NaN
  - "" (the empty string)
  - 0n (the bigint version of zero)
  - null
  - undefined
- (의견) 위 typeof 에서 커버하지 못한 null, undefined 들은 if-else 에서 잘 처리하라는 이야기같다.


결국 null 인 경우에는 typeof guard 를 달아도 object로 판단되니, 다음처럼 처리하면 좋다.

```ts
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

#### Equality narrowing

- Truthiness narrowing 과 비슷한 맥락으로 타입의 범위를 좁히며 추론할 때, `==`, `!=` 을 사용하자

```ts
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
                            
    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

- 위처럼 null 인지 먼저 판단하여 분기를 태워두는 것이 좋다

#### ETC

- 더 이상 narrowing 에 대한 항목을 보는 것은 시간 낭비인 것 같고, 좀 더 간단하게 요약한다.
  - 타입을 추론하는 방법론들에 대한 것이라 한번만 읽고 넘어가면 될 것 같은 정도라고 생각한다.


**in**

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```
`value in x` 로 property 가 특정 value 에 존재하는지 알 수 있다.

**instanceof**

`x instanceof Foo` 는 x가 Foo.prototype 을 포함하는지 확인한다. 여기서는 더 나아가 알아보지 않지만, 그냥 특정 변수가 클래스의 instance 인지 확인하는 키워드라고 생각할 수 있다.

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
  } else {
    console.log(x.toUpperCase());
  }
}
```

**Assignments**

```ts
// 이 상태에서는 number | string 으로 추론된다
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
// 할당 이후 number 로 추론된다
x = 1; 
console.log(x);

// 할당 이후 string 으로 추론된다
x = "goodbye!";
console.log(x);

// union type 에 포함되지 않은 타입이기 때문에 에러
x = true;
^ Type 'boolean' is not assignable to type 'string | number'.
```

**Using type predicates**

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

코드가 돌면서 타입이 어떻게 변하는지에 대해 직접적으로 통제하고 싶을 때가 있다. 위처럼 type predicate 을 return type 으로 지정하면 된다.

```ts
compatible.

// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();
 
if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

위와 같은 코드를 작성하면 isFish 가 호출될 때 마다 원래 타입이 적절하다면 자동으로 narrow 하여 타입을 추론할 것이다.

```ts
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];
 
// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

Array 에서는 filter를 사용해 구현할 수 있다.

**discriminated unions**

```ts
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

위 인터페이스에서 circle에는 항상 radius가 있음을 전제로, 면적을 구하는 함수를 구현하고 싶은 경우에는 다음과 같이 할 수 있다.

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

하지만 위 코드에서 radius 가 혹시라도 없으면 어떡하지? 라는 찝찝함이 남는다. circle 이라는 kind 를 가진 Shape 가 생성될 때 radius가 꼭 같이 들어가는지를 보장할 수 없기 때문이다.

```ts
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;
```

이럴 때에는 위처럼 kind 를 고정해버리고 두가지 interface를 생성해 Shape 의 type alias로 지정해버릴 수 있다.

```ts
// if-else

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
}

// switch-case

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

그럼 위처럼 circle 이라는 kind 를 가진 Shape 는 radius 값을 항상 가지고 있음이 보장된다

추가로 위에 `_exhaustiveCheck: never` 를 끼워넣었는데, 모든 가능성을 제외하고 일어날 수 없는 케이스를 이처럼 표현한다. kind, sideLength와 같이 적절한 필드를 가진 object 가 파라미터로 전달되면 함수에서 요구하는 property 가 전부 잘 있기때문에 에러가 발생하지 않는다. 이 때 `_exhaustiveCheck: never` 를 처리해주면 케이스에 해당하지 않는 의도치 않은 케이스를 감지하고 에러를 잘 발생시켜 준다.


(2시간 반정도 지나서 일단 stop)

### More on Functions

### Object Types

### Type Manipulation

### Classes

### Modules

## 개발 환경 세팅

### TsConfig

### Good patterns