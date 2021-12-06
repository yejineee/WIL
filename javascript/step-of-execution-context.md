# 실행 컨텍스트의 생성 과정

실행 컨텍스트가 생성되는 과정을 알아본다. `[[Scopes]]` 프로퍼티, 변수 호이스팅, 함수 호이스팅에 대해 알아본다.

아래의 코드로 어떻게 실행 컨텍스트가 생성되는지 알아보자.

```javascript
var x = 'xxx';

function foo () {
  var y = 'yyy';

  function bar () {
    var z = 'zzz';
    console.log(x + y + z);
  }
  bar();
}

foo();
```

## 📌 1. 전역 코드 \(Global Code\)로 진입

* 컨트롤이 실행 컨텍스트에 진입하기 전에 **전역객체\(Global Object/GO\)가 생성**된다. 
* 전역객체는 유일하며, 객체의 프로퍼티는 코드에서 접근할 수 없다.
* 초기 상태에는 빌트인 객체인 Math, String, Array 등과 BOM, DOM이 설정되어 있다. ![](https://i.imgur.com/vyeqaEe.png)
* 전역 코드로 컨트롤이 진입하면, 전역 실행 컨텍스트가 생성되고, 실행 컨텍스트 스택에 쌓인다. ![](https://i.imgur.com/uO2SdGc.png)

실행 컨텍스트에는 스코프 체인, 변수 객체, this value가 있다. 이 실행 컨텍스트를 바탕으로 다음의 처리가 순서대로 실행된다. \(1 -&gt; 2 -&gt; 3\)

> 1. 스코프체인 생성과 초기화
> 2. 변수 객체화 실행
> 3. this value 결정

### 1-1. 스코프체인 생성과 초기화

`스코프 체인`은 **실행 컨텍스트가 참조할 수 있는 변수 객체 참조의 리스트**이다. 실행 컨텍스트가 생성된 후, 가장 먼저 스코프체인의 생성과 초기화가 실행된다. 전역 컨텍스트의 스코프 체인은 **전역 객체\(GO\)의 레퍼런스를 담고 있는 리스트**가 된다. ![](https://i.imgur.com/siA2Tfb.png)

### 1-2. 변수 객체화\(Variable Instantiation\) 실행

* 변수 객체화는 변수 객체\(VO\)에 프로퍼티와 값을 추가하는 것이다. 
* 전역 코드의 VO는 전역 객체 Global Object\(GO\)를 가리킨다. 
* **매개변수와 인수 정보, 함수 선언, 변수를 VO에 추가하여 객체화**한다. 

![](https://i.imgur.com/xYt1WAC.png)

* **변수 객체화 순서**

  > 1. **매개 변수와 인수 정보 추가**
  >
  >    함수 코드의 경우, VO의 프로퍼티로 매개변수\(parameter\), 값으로 argument가 설정된다.
  >
  >    전역 코드의 경우, 매개변수가 없으므로 이 과정은 실행되지 않는다.
  >
  > 2. **함수 호이스팅**
  >
  >    코드 내의 함수 선언\(함수 표현식 제외\)을 대상으로 VO의 프로퍼티로 함수명, 그 값으로 생성된 _함수 객체_ 가 설정된다.
  >
  > 3. **변수 호이스팅**
  >
  >    코드 내의 변수 선언을 대상으로 VO의 프로퍼티로 변수명, 값으로 undefined가 설정된다.

전역코드이므로, 1은 실행되지 않고, 전역 함수인 foo 선언이 처리되고, 그 후 전역 변수인 x의 선언이 처리된다.

#### 1-2-1 함수 foo 처리 - foo 함수 생성

* foo 함수 생성으로 VO의 프로퍼티로 함수 이름인 foo가, 값으로 함수 객체가 설정된다. 

  ![](https://i.imgur.com/EtCQvdm.png)

* 이 때, foo 함수의 `[[Scopes]]`는 다음과 같다.

  > foo.\[\[Scopes\]\] = \[ globalContext.VO \]

* 함수 객체의 `[[Scopes]]`프로퍼티
  * 함수 객체는 내부 프로퍼티인 `[[Scopes]]` 프로퍼티를 갖고 있다. 
  * 현재 실행 컨텍스트의 **스코프 체인이 참조하고 있는 객체**를 가리킨다. 
  * `[[Scopes]]` 프로퍼티는 자신의 실행 환경 Lexical Environment와 자신을 포함하는 외부 함수의 실행 환경인 AO와 전역 객체\(Global Object\)를 가리키게 된다. 
  * 이 때, 외부 함수의 실행 컨텍스트가 소멸하여도 `[[Scopes]]`는 외부 함수의 실행 환경을 여전히 참조할 수 있다. 이를 **클로저**라고 한다. 
* **함수 호이스팅** `함수 선언식` 의 경우 변수 객체에 함수명을 프로퍼티로, 함수 객체가 값으로 할당된다. 따라서, 함수 선언식은 선언문 이전에 함수를 호출할 수 있다.

```javascript
hello(); // hello
function hello() { // 함수 선언식
  console.log("hello");
}
```

`함수 표현식`은 함수 객체에 함수명을 프로퍼티로 추가하지만, 그 방식은 _일반 변수의 방식_ 을 따른다. 따라서, 함수 표현식은 선언문 이전에 함수를 호출할 수 없다.

#### 1-2-2 변수 x 처리

변수는 변수명이 VO의 프로퍼티로, undefined가 값으로 설정된다. 변수 처리는 다음 세 단계로 나뉘어서 이루어진다.

> **1. 선언 단계**
>
> * **변수 객체에 변수를 등록**한다. 이 변수는 스코프가 참조할 수 있는 대상이 된다.
>
>   **2. 초기화 단계**
>
> * 변수 객체에 등록된 변수를 **메모리에 할당**한다.
> * 이 단계에서 변수는 undefined로 초기화된다.
>
>   **3. 할당 단계**
>
> * 실제 값을 할당한다.

**var 키워드로 선언된 변수는 스코프 초기에 선언 단계와 초기화 단계가 한 번에 이루어진다**. 따라서, 변수 선언문 이전에 변수에 접근하여도 VO에 변수가 있고, 메모리에도 할당되어 있으므로 에러가 발생하지 않는다.

```javascript
// 스코프의 초기에 선언 단계(1)와 초기화 단계(2)가 실행된다.
console.log(hello); // undefined
var hello;          // 변수 선언문
console.log(hello); // undefined

hello = "hello";    // 할당문 -> 할당 단계(3)가 실행된다.
console.log(hello); // hello
```

다만, 아직 변수 x는 'xxx'로 초기화되지 않았고, 변수 할당문에 도달하면 값의 할당이 이루어진다. ![](https://i.imgur.com/U3Rrj7C.png)

위에서 `함수 표현식`은 일반 변수의 방식을 따른다고 하였다. 따라서, var 키워드로 선언한 함수 표현식을 선언문 이전에 호출하면, undefined가 반환된다.

```javascript
console.log(bye); // undefined
var bye = function () {
  // 함수 표현식
  console.log("bye");
};
```

* **const, let 키워드의 변수 호이스팅**

  함수 표현식과 변수가 const나 let으로 할당된다면, 변수나 함수 표현식 선언문 이전에 호출하려고 하면, Reference Error가 발생한다. 이는 const나 let으로 할당한 변수는 스코프 초기에 선언 단계만 일어나고, 선언문에 도달해야 초기화 단계가 일어나기 때문이다. 따라서 선언문 이전에 접근하는 것은 메모리에 해당 변수나 함수 표현식이 없기 때문에 Reference Error가 발생하는 것이다. 

```javascript
console.log(goodBye); // ReferenceError: Cannot access 'goodBye' before initialization

const goodBye = function () {
  // 함수 표현식
  console.log("goodBye");
};
```

### 1-3 this value 결정

변수 선언 처리 후, this value가 결정된다.

* **this value**가 결정되기 전에 this는 전역 객체를 가리키고 있다가, 함수 호출 패턴에 의해 this에 할당되는 값이 결정된다.
* 전역 코드의 경우, VO, 스코프 체인, this는 항상 전역 객체를 가리킨다. 

  ![](https://i.imgur.com/5nGKcVL.png)

## 📌 2. 전역 코드 실행

지금까지는 프로그램 실행을 위한 준비 과정이었다. 지금부터 코드가 시작된다.

```javascript
var x = 'xxx';

function foo () {
  var y = 'yyy';

  function bar () {
    var z = 'zzz';
    console.log(x + y + z);
  }
  bar();
}

foo()
```

위 예제에서는 변수 x에 'xxx'할당과 함수 foo의 호출이 실행된다.

### 2-1. 변수 x에 값 할당

현재 실행 컨텍스트의 스코프 체인은 참조 가능한 VO를 리스트로 갖고 있다. 0번째 VO부터 검색하여 변수명 x에 해당하는 프로퍼티가 발견되면, 그 프로퍼티에 값 'xxx'를 할당한다.

![](https://i.imgur.com/6PmQzPQ.png)

### 2-2. 함수 foo 호출

* 함수 foo가 호출되면, 새로운 함수 실행 컨텍스트가 생기고, 실행 컨텍스트 스택에 push된다. 
* 컨트롤이 함수 foo의 실행 컨텍스트로 이동하면, 마찬가지로 1. 스코프 체인의 생성과 초기화, 2. 변수 객체화 실행 3. this value 결정이 순차적으로 실행된다. 
* 단, 이 코드는 전역코드가 아닌 함수 코드이므로, 함수 코드의 룰이 적용된다. 

![](https://i.imgur.com/19jIeog.png)

#### 2-2-1. 스코프 체인의 생성 및 초기화

함수 컨텍스트의 스코프 체인은 `Scope = AO + [[Scopes]]`로 정의할 수 있으며, 보통 배열로 나타낸다는 것을 기억해야 한다. foo함수의 `[[Scopes]]`는 \[1\]전역 코드로 진입 시 결정되었으며, 그 값은 다음과 같았다.

> foo.\[\[Scopes\]\] = \[ globalContext.VO \]

1. 이 함수의 변수 객체인 AO가 생성된다.
2. 함수 foo의 스코프 체인은 먼저 자신의 실행 환경인 foo의 Activation Object\(AO\)를 가리킨다. ![](https://i.imgur.com/CfikAd7.png)
3. AO는 arguments 프로퍼티의 초기화를 실행한다.
4. foo함수의 `[[Scopes]]`가 스코프 체인에 push된다. 즉, foo의 스코프 체인은 AO와 GO를 순차적으로 담은 리스트를 가리키게 된다.

   > fooContext.Scope = fooContext.AO + foo.\[\[Scopes\]\]
   >
   > fooContext.Scope = \[ fooContext.AO, globalContext.VO \]

![](https://i.imgur.com/Dp48nxY.png)

#### 2-2-2. 변수 객체화 \(Variable Instantiation\) 실행

Function Code는 AO에 대해 변수 객체화를 실행한다. 1. bar 함수 처리 - bar 함수 생성

![](https://i.imgur.com/r0n7rMo.png)

* AO에 함수 이름 bar를 프로퍼티로, bar의 함수 객체를 값으로 하여 함수를 추가한다.
* bar 함수 객체의 내부 프로퍼티인 `[[Scopes]]` 프로퍼티는 스코프 체인이 가리키는 값을 가리킨다. 

> bar.\[\[Scope\]\] = \[ fooContext.AO, globalContext.VO \]

1. 변수 y 처리 - 변수 호이스팅

   ![](https://i.imgur.com/MmcGxgK.png)

변수명 y를 프로퍼티로, undefined를 값으로 하여 AO에 추가한다.

#### 2-2-3. this value 결정

this에 할당되는 값은 함수 호출 패턴에 의해 결정된다.

* 내부함수의 경우, this의 값은 전역 객체이다. 

  ![](https://i.imgur.com/4eoI0N7.png)

## 📌 3. foo 함수 코드 실행

```javascript
var x = 'xxx';

function foo () {
  var y = 'yyy';

  function bar () {
    var z = 'zzz';
    console.log(x + y + z);
  }
  bar();
}

foo();
```

먼저 y 변수에 'yyy'를 할당하고, 함수 bar가 실행된다.

### 3-1 변수 값 할당

변수 y에 값을 할당하기 위해, 스코프 체인을 따라 가며 VO에 변수명 y를 검색한다. y를 발견하면, 그 프로퍼티에 값을 'yyy'로 할당한다.

![](https://i.imgur.com/Et85LOf.png)

### 3-2 함수 bar 호출

함수 bar가 호출되면, bar의 실행 컨텍스트가 생성되고, 실행 컨텍스트 스택에 푸쉬된다. 실행 컨텍스트의 1. 스코프 체인의 생성과 초기화 2. 변수 객체화 실행 3. this 결정이 순차적으로 이루어진다.

![](https://i.imgur.com/YIozAga.png)

스코프 체인 생성시, bar의 스코프 체인은 bar의 VO인 AO-2를 참조하게 된다. 그 후 bar의 `[[Scopes]]`를 스코프 체인에 push한다. 결과적으로 bar 함수 컨텍스트의 스코프 체인은 다음처럼 구성된다.

> barContext.Scope = \[ barContext.AO fooContext.AO, globalContext.VO \]

## 📌 4. bar 함수 코드 실행

`console.log(x + y + z);`의 결과는 xxxyyyzzz인데, bar 함수 컨텍스트의 스코프 체인에서 변수를 검색하여 찾을 수 있다.

> barContext.Scope = \[ barContext.AO - z : 'zzz' fooContext.AO, - y : 'yyy' globalContext.VO - x : 'xxx' \]

## 📌 정리

* 실행 컨텍스트는 함수 호출시 생성되며, 실행 컨텍스트 스택에 푸쉬된다.
* 실행 컨텍스트 생성시 실행 컨텍스트의 세 가지 객체에 대한 처리가 이루어진다.

  > 1. 스코프체인 생성과 초기화
  >    * 함수 컨텍스트의 경우 스코프 체인은 `Scope = AO + [[Scopes]]`이다.
  > 2. 변수 객체화
  >    1. 매개변수와 인수 정보 추가
  >    2. 함수 호이스팅
  >       * property : 함수 이름
  >       * value : 함수 객체
  >    3. 변수 호이스팅
  >       * property : 변수명
  >       * value : undefined
  > 3. this value 결정

* 함수의 `[[Scopes]]` 결정
  * 함수 객체 생성 시 결정된다.
  * 함수 객체를 생성한 실행 컨텍스트의 스코프 체인이 가리키는 객체를 가리킨다.

## 참조

* [실행 컨텍스트와 자바스크립트의 동작 원리-poiemaweb](https://poiemaweb.com/js-execution-context)
* [dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/#function-creation)

