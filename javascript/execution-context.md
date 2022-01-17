# 실행 컨텍스트 (Execution Context)

## 🪵 정의

> Execution context (abbreviated form — EC) is the abstract concept used by ECMA-262 specification for typification and differentiation of an executable code.

실행컨텍스트는 실행 가능한 코드가 실행되기 위해 필요한 환경이라고 할 수 있다.

개념적으로 active 실행 컨텍스트의 집합은 _스택_ 을 구성한다. 스택의 bottom에는 항상 `global context`가 있고, top에는 현재 active 실행 컨텍스트가 있다.

## 🪵 Executable code의 종류

실행 가능한 코드에는 네 종류가 있다.

1. global code
2. function code
3. eval code
4. module code

실행 컨텍스트의 스택을 배열로 정의해보자.

```javascript
ECStack = [];
```

스택은 함수에 들어갈 때마다 push되고, 내장된 eval 함수가 실행될 때도 push된다.

### Global Code

* 글로벌 코드는 `Program`레벨에서 처리된다. 예를 들어 로드된 외부의 .js 파일이나, 내부의 인라인 코드 (`<script></script>`)에서 처리된다.
* 글로벌 코드는 함수 내부에 있는 어떠한 코드도 포함하지 않는다.
* 프로그램이 시작할 때, ECStack은 다음과 같다.

```javascript
ECStack = [
    globalContext
];
```

### Function Code

* 함수 코드에 들어갈 때마다, ECStack에 새로운 요소가 푸쉬된다. 이 때, 함수 내부에 있는 함수는 포함되지 않는다. 예를 들어, 다음과 같은 재귀함수가 있을 때,

```javascript
(function foo(flag) {
  if (flag) {
    return;
  }
  foo(true);
})(false);
```

ECStack은 다음과 같이 수정된다.

```javascript
// first activation of foo
ECStack = [
  <foo> functionContext
  globalContext
];
  
// recursive activation of foo
ECStack = [
  <foo> functionContext – recursively 
  <foo> functionContext
  globalContext
];
```

처음 foo함수가 호출되면, ECStack에 foo 함수 컨텍스트가 푸쉬되고, 그 내부의 foo가 호출되면 다시 foo 함수 컨텍스트가 푸쉬된다.

함수에서 리턴될 때마다, 현재 실행 컨텍스트에서 빠져나오게 되고, ECStack은 pop이 된다 (순차적으로 혹은 그 반대로). 프로그램이 끝나면, ECStack에는 글로벌 컨텍스트만 남게 된다.

* 예외가 throw되었지만, 잡히지는 않은 경우에도 하나 이상의 실행 컨텍스트를 빠져나오게 된다.

```javascript
(function foo() {
  (function bar() {
    throw 'Exit from bar and foo contexts';
  })();
})();
```

### Eval Code

* eval 함수가 호출된 컨텍스트인 _calling context_라는 것이 있다.
* eval 함수에 의해 만들어진 액션은 calling context에 영향을 준다.

```javascript
// influence global context
eval('var x = 10');
 
(function foo() {
  // and here, variable "y" is
  // created in the local context
  // of "foo" function
  eval('var y = 20');
})();
  
alert(x); // 10
alert(y); // "y" is not defined
```

이 코드의 ECStack의 변경은 다음과 같다.

```javascript
ECStack = [
  globalContext
];
  
// eval('var x = 10');
ECStack.push({
  context: evalContext,
  callingContext: globalContext
});
 
// eval exited context
ECStack.pop();
 
// foo funciton call
ECStack.push(<foo> functionContext);
 
// eval('var y = 20');
ECStack.push({
  context: evalContext,
  callingContext: <foo> functionContext
});
 
// return from eval 
ECStack.pop();
 
// return from foo
ECStack.pop();
```

## 🪵 요약

개념적으로 실행 컨텍스트는 콜스택처럼 _스택_으로 유지된다.

> **Def. 8: Execution context stack**: An execution context stack is a LIFO structure used to maintain control flow and order of execution.

실행 컨텍스트 스택은 LIFO 구조를 갖고 있으며, 이는 control flow와 실행 순서를 유지하는데 사용된다.

ECMAScript code에는 여러 타입이 존재한다 : global code, function code, eval code, module code. 각 코드는 그의 해당하는 실행 컨텍스트에서 처리된다. 각 코드 타입과 그것들에 적합한 객체는 실행 컨텍스트의 구조에 영향을 준다.

예를 들어, 전역 실행 컨텍스트의 변수 객체(VO)는 전역 객체 Global Object(GO)라고 하며, 인수에 대한 정보를 담지 않는다. 반면 함수 코드에서 생성된 함수 실행 컨텍스트의 변수 객체(VO) Activation Object(AO)라고 하며, 파라미터와 인수에 대한 정보를 담는다.

다음 재귀 함수 호출을 살펴보자.

```
function recursive(flag) {
 
  // Exit condition.
  if (flag === 2) {
    return;
  }
 
  // Call recursively.
  recursive(++flag);
}
 
// Go.
recursive(0);
```

![](https://i.imgur.com/ztpL6MQ.png)

* _글로벌 컨텍스트_는 항상 스택의 bottom에 위치한다. 글로벌 컨텍스트는 다른 컨텍스트가 생성되기 이전에 생성된다는 것이다. 전역 컨텍스트는 애플리케이션이 종료될 때까지 유지된다.
* 새로운 함수가 호출되면, 새로운 실행 컨텍스트가 생기고, 스택에 push된다. 이 시점에서 이 실행 컨텍스트는 active 실행 컨텍스트가 된다.
* 함수가 리턴되었을 때, 컨텍스트는 스택에서 pop된다.
* 다른 컨텍스트를 호출한 컨텍스트를 _caller_라고 하며, 호출된 컨텍스트를 _callee_라고 한다. 위의 예시에 있는 recursive 함수는 caller이자 callee가 된다.

### 참조

* [ECMA-262-3 in detail. Chapter 1. Execution Contexts.](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)
* [실행 컨텍스트와 자바스크립트의 동작 원리 - poiemaweb](https://poiemaweb.com/js-execution-context)
