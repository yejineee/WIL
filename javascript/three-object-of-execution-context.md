# 실행 컨텍스트의 세 가지 객체 \(VO, SC, this\)

실행 컨텍스트는 물리적으로 객체의 형태를 가지며, 3가지 프로퍼티를 갖고 있다.

1. 변수 객체 \(variable object\)
2. 스코프 체인 \(scope chain\)
3. this value 

## 💫 변수 객체 \(VO\)

실행 컨텍스트가 생성되면, 자바스크립트 엔진은 이 실행 컨텍스트에 필요한 정보를 담을 객체를 생성하는데 이를 **변수 객체**라고 한다.

* 변수 객체는 코드가 실행될 때, 엔진이 참조한다.
* 코드에서는 접근할 수 없다. 

변수객체는 아래의 정보를 담는다.

> 1. 매개변수 \(parameters\)와 인수 정보\(arguments\) - 함수 컨텍스트의 경우
> 2. 함수 선언 - 함수 표현식은 제외
> 3. 변수

* 변수 객체는 실행 컨텍스트의 프로퍼티이고, 이 변수 객체는 객체를 가리킨다. 
* 실행 컨텍스트에 따라 가리키는 객체가 다르다.

  전역 코드 \(Global Code\)와 함수 코드 \(Function Code\) 실행시 생성되는 실행 컨텍스트에 따라 가리키는 객체가 다르다. 이는 전역 코드와 함수 코드의 내용이 다르기 때문이다. 예를 들어 전역 코드에는 매개변수가 없고, 함수 코드에는 매개변수가 있다.

### 전역 컨텍스트의 경우

* **전역 객체**\(Global Object / GO\)를 가리킨다.
* 변수 객체는 _유일_하고, _최상위_에 위치한다.
* 모든 **전역 변수**와, **전역 함수**를 프로퍼티로 갖는다.

![](https://i.imgur.com/xp8905O.png)

### 함수 컨텍스트의 경우

* **활성 객체**\(Activation Object / AO\)를 가리킨다. 
* **지역 변수**와 **내부 함수**를 프로퍼티로 갖는다. 
* 프로퍼티에 인수들의 정보를 배열로 담고 있는 객체인 **arguments object**가 추가된다.

![](https://i.imgur.com/5U5BWj4.png)

## 💫 스코프 체인 \(SC\)

![](https://i.imgur.com/BCKB4Du.png)

* 스코프체인은 **해당 컨텍스트가 참조할 수 있는 변수 객체\(전역 객체 또는 활성 객체\)의 레퍼런스를 담고 있는 리스트**이다. 
* 먼저 현재 실행 컨텍스트의 활성 객체를 가리키고, 상위 컨텍스트의 활성 객체를 가리키다가, 마지막에는 전역 객체를 가리킨다.
* 엔진은 스코프 체인을 통해 **렉시컬 스코프**를 파악한다. 
* 스코프 체인은 _variable lookup_ 에 사용된다.

  변수를 파악하기 위해서 먼저 현재 스코프, 즉 Activation Object에서 검색하고, 존재하지 않으면 스코프 체인에 담긴 순서대로 검색을 하게 된다. 

  \(예\) AO -&gt; AO -&gt; ... -&gt; GO \(Global Object\)

* 스코프 체인을 통해 검색을 하다가, 검색에 실패하면 _Reference Error_를 발생시킨다.

```javascript
function testNotFoundVar() {
  console.log(notDefVar);
}
testNotFoundVar(); // ReferenceError: notDefVar is not defined
```

* 함수 컨텍스트의 스코프체인은 _함수 호출_ 시 생성되며, activation object와 이 함수의 내부 프로퍼티인 `[[Scope]]`로 구성되어 있다.

> Scope = AO + `[[Scope]]`

* 함수 객체의 `[[Scopes]]`프로퍼티
  * 함수 객체는 `[[Scopes]]` 프로퍼티를 갖고 있다. 
  * `[[Scopes]]` 프로퍼티는 함수 객체의 내부 프로퍼티로 **함수 객체가 실행되는 환경**을 가리킨다. 
  * 즉 현재 실행 컨텍스트의 **스코프 체인이 참조하고 있는 객체**를 가리킨다. 
  * `[[Scopes]]` 프로퍼티는 자신의 실행 환경 Lexical Environment와 자신을 포함하는 외부 함수의 실행 환경인 AO와 전역 객체\(Global Object\)를 가리키게 된다. 
  * 이 때, 외부 함수의 실행 컨텍스트가 소멸하여도 `[[Scopes]]`는 외부 함수의 실행 환경을 여전히 참조할 수 있다. 이를 **클로저**라고 한다. 

#### `[[Scopes]]` vs 스코프 체인

* ❗️ 함수의 생명 주기는 _생성\(creation\)_ 과 _호출\(call\)_ 로 구분된다. 
* `[[Scopes]]` 는 _함수 객체_ 의 프로퍼티이며, 함수 _생성_ 단계에서 결정된다.
* 스코프 체인 \(SC\)은 _함수 컨텍스트_ 의 프로퍼티이고, 함수 _호출_ 시 함수 컨텍스트가 생성 될 때, 제일 먼저 결정된다.

#### 스코프 체인 vs 프로토타입 체인

> Scope chain is related with an execution context a chain of variable objects which is used for variables lookup at _identifier_ resolution.

* 스코프 체인 : 식별자 중에서 _변수_ 를 검색하는 메커니즘이다. 단, 전역 객체는 객체의 프로퍼티를 검색할 수 있다.
* 프로토타입 체인 : 식별자중에서 _객체의 프로퍼티_ 를 검색하는 메커니즘이다. 

## 💫 this value

* this값이 할당되며, this는 함수 호출 패턴에 의해 결정된다.

## 참조

* [실행 컨텍스트와 자바스크립트의 동작 원리 - poiemaweb](https://poiemaweb.com/js-execution-context)
* [dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/#function-creation)

