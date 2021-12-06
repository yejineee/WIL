# 호이스팅이란?

실행 컨텍스트는 생성과 실행 단계로 나뉜다. 호이스팅은 자바스크립트 엔진이 실행 컨텍스트를 생성하는 단계에서, 실행 컨텍스트의 scope를 정의할 때, 변수나 함수 선언문을 스코프의 최상단으로 끌어올리는 것을 말한다.

다음 구문이 있을 때,

```javascript
var variable = "variable";
```

자바스크립트 엔진은 컴파일 과정에서 이를 선언문과 할당문으로 나누어서 해석한다.

```text
var variable; // 선언문
variable = "variable"; // 할당문
```

호이스팅은 이 선언문을 스코프의 최상단으로 끌어올리는 것이다.

## 변수 호이스팅

```javascript
console.log(a); // undefined
var a = 'hi';
console.log(a); // hi
```

변수는 '컴파일'과정에서 스코프의 최상단으로 끌어올려져서, 변수 객체에 추가된다. 이 때 프로퍼티는 변수 이름, 값은 undefined로 초기화된다. 따라서 변수 선언문 이전에, 변수를 참조할 수 있다. 함수가 실행되고 할당문에서 비로소 a는 'hi'로 할당된다.

### let, const keyword

let과 const의 경우에도 호이스팅이 일어나지만, 선언문 이전에 참조하려고 하면 Reference Error가 발생된다. 변수 선언문이 스코프 최상단으로 끌어올려지기는 한다. 하지만, 함수가 실행되며 _선언문에 도달해야_ 변수를 위한 메모리가 확보되고 undefined로 초기화가 된다. 스코프 최상단부터 선언문 전까지를 일시적 사각 지대\(Temporal Dead Zone; **TDZ**\) 라고 한다.

```javascript
{
  console.log(apple); //ReferenceError: Cannot access 'apple' before initialization
  let apple; 
  apple = "apple";
}
```

`ReferenceError: Cannot access 'apple' before initialization` 이 에러를 통해 'apple'은 heap memory에 이미 존재하지만, 초기화는 이루어지지 않았음을 알 수 있다.

## 함수 호이스팅

함수 선언문의 경우에도 호이스팅이 된다. 컴파일 과정에서 함수는 변수 객체에 프로퍼티로 함수 이름이, 값으로는 함수 객체가 등록이 된다. 따라서, 함수 선언문 이전에도 함수 호출을 할 수 있게 된다.

```javascript
func(); //func 함수 호출
function func() {
  console.log("func 함수 호출");
}
```

## 함수 표현식

이 때, 주의할 점은 **함수 표현식은 변수 호이스팅으로 처리**된다는 점이다. 즉, 컴파일 될 때 함수 표현식은 변수 객체에 프로퍼티는 함수이름, 값은 undefined로 등록된다.

```javascript
console.log(hello); // undefined
hello(); // TypeError: hello is not a function
var hello = function () {
  console.log("hello 함수 호출");
};
```

함수 표현식은 함수 선언문 이전에 호출하여도 호이스팅 되므로 참조는 할 수 있지만, 그 값은 undefined인 것을 확인할 수 있다.

## 함수 호이스팅 &gt; 변수 호이스팅

함수 호이스팅이 일어나고, 그 다음 변수 호이스팅이 일어난다.

```javascript
{
  quit(); // 함수 호이스팅 - A
  var quit = function () { // B
    console.log("변수 호이스팅");
  };
  function quit() {
    console.log("함수 호이스팅");
  }
  quit(); // 변수 호이스팅 - C
}
```

컴파일 과정에서, 함수 선언문이 스코프의 최상단으로 끌어올려져서, 변수 객체에 함수이름인 quit과 quit 함수 객체가 등록된다. quit이라는 이름으로 선언문이 끌어올려졌으므로, quit 변수 호이스팅은 일어나지 않게 된다. 따라서 A지점에서 함수 호출을 하면, 함수 객체가 실행되어 '함수 호이스팅'을 출력한다. 함수가 실행되면서, B지점에 이르게 되면, quit이라는 이름에 새로운 함수가 할당된다. 따라서 C지점에서는 '변수 호이스팅'을 출력하게 된다.

## 참조

* [JavaScript Hoisting -  javascript tutorial](https://www.javascripttutorial.net/javascript-hoisting/)

