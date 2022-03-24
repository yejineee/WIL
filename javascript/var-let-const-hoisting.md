# var, let, const와 호이스팅

## 🌲 var

#### **함수 레벨 스코프**

**var는 함수 레벨 스코프를** 갖기 때문에, for문 안에서 선언된 변수인 test를 for문 밖에서도 참조할 수 있게 된다.

```javascript
function func() {
for (var i = 0; i < 5; i++) {
  var test = "test";
}
console.log(test); // test
}
func();
```

#### **변수 중복 선언 허용**

다음처럼 같은 이름의 변수를 중복선언하여도 문제가 없다.

let이나 const로 변수를 중복선언한다면, SyntaxError가 발생하게 된다.

```javascript
var hi = "hi";
var hi = "hello";
console.log(hi) // hello
```

#### **변수 호이스팅**

변수를 선언하기 이전에 참조할 수 있다.

hi를 선언하기 전에 hi를 참조하려고 하면 에러가 발생하지 않고 undefined를 출력한다.

이를 통해 실행 컨텍스트에 변수가 변수 객체에 등록되어 있고, 메모리를 할당받았음을 알 수 있다.

```javascript
function sayHi() {
console.log(hi); // undefined
var hi = "hi";
}
sayHi();
```

#### **전역 객체와 var**

#### `전역객체`는 모든 객체의 유일한 최상위 객체이다

브라우저 환경에서는 window, node에서는 global이 전역객체이다.

var로 선언한 변수를 전역 스코프에 선언하면,**전역객체의 프로퍼티**가 된다.

```javascript
var declared = 1;  // Explicit global variable (new variable)

window.hasOwnProperty('declared')    // true
```

## 🌲 let

#### **블록레벨 스코프**

* 자바스크립트는 **함수 레벨 스코프**를 따른다.
* ES6에서 제공하는 `let`은 **블록 레벨 스코프**를 따른다.

#### 변수 중복 선언 금지

let으로는 동일한 이름을 갖는 변수를 선언하면 SyntaxError가 발생하게 된다.

```javascript
let hi = 10;
let hi = 100; // SyntaxError: Identifier 'hi' has already been declared
```

#### 전역객체와 let

var로 선언한 변수를 전역변수로 사용하면, 전역객체의 프로퍼티가 되었다.

그러나, let으로 선언된 변수를 전역변수로 사용되어도 let 전역 변수는 _전역 객체의 프로퍼티가 되지 않는다._

let은 보이지 않는 개념적인 블록 내에서만 존재하기 때문이다.

```javascript
let letvariable = 1;
window.hasOwnProperty('letvariable') // false
```

## 🌲 const

#### const는 재할당이 금지된다

const는 상수를 위해 사용되기에 재할당을 시도하면 타입에러가 발생한다.

```javascript
const a = 10;
a = 200; // TypeError: Assignment to constant variable.
```

#### 전역객체와 const

let과 마찬가지로 const로 선언된 변수가 전역 변수이더라도, 전역객체의 프로퍼티가 되지 않는다.

```javascript
const constvariable = 1;
window.hasOwnProperty('constvariable'); // false
```

#### const는 선언과 동시에 할당이 이루어져야 한다.

```javascript
const bar; // SyntaxError: Missing initializer in const declaration
```

#### 블록 레벨 스코프를 갖는다.

```javascript
{
  const constVar = 10;
}
console.log(constVar); // ReferenceError: constVar is not defined
```

#### const와 객체

객체를 const 변수의 타입으로 할당을 하면, 객체에 대한 참조를 변경하지 못한다.

```javascript
const obj = { foo: 123 };
obj = { bar: 456 }; // TypeError: Assignment to constant variable.
```

그러나, **객체의 프로퍼티는 보호되지 않는다.**

따라서 할당된 객체의 내용을 추가, 삭제, 수정할 수 있다.

```javascript
const user = { name: "name" };
user.age = 10; // 프로퍼티 추가
user.name = "new name"; // 프로퍼티 변경
console.log(user); // { name: 'new name', age: 10 }
```

#### const와 배열

배열 또한 const 변수의 타입으로 할당을 한다면, 재할당은 불가능하다.

그러나, 배열의 요소를 변경하는 것은 가능하다.

```javascript
const number = [];

for (let i = 0; i < 3; i++) {
  number.push(i);
}

console.log(number); // [0,1,2]
```

## 🌲 호이스팅

자바스크립트는 **let, const를 포함하여 모든 선언(var, let, const, function, class)를 호이스팅**한다.

그러나 let으로 선언된 변수를 선언문 이전에 참조하면 ReferenceError가 발생한다.

그 이유를 알아보려면, 변수가 어떻게 생성되는지 알아야 한다.

### **변수 생성 과정**

#### 1. 선언 단계 (Declaration phase)

* 스코프에 변수를 등록한다.
* 변수를 실행 컨텍스트의 **변수 객체 (Variable Object)에 등록**한다.
* 이 변수 객체는 스코프가 참조하는 대상이 된다.

#### 2. 초기화 단계 (Initialization phase)

* 변수 객체에 등록된 변수를 위한 공간을 **메모리에 확보**한다.
* 이 단계에서 변수는 undefined로 초기화 된다.

#### 3. 할당 단계 (Assignment phase)

* undefined로 초기화된 변수에 실제 값을 할당한다.

**var**로 선언된 변수는 **선언단계(1)와 초기화 단계(2)가 한 번에 이루어진다**.

따라서 변수 선언문 이전에 변수에 접근하여도 스코프에 변수가 존재하므로, 에러가 발생하지 않고 undefined를 반환하게 된다.

변수 할당문에 도달하면 값이 할당된다.

이러한 현상을 **변수 호이스팅**이라고 한다.

```javascript
var hello; // 변수 선언문 console.log(hello); // undefined
hello = "hello"; // 할당문 -> 할당 단계(3)가 실행된다. 
sconsole.log(hello); // hello
```

![](https://i.imgur.com/oVJT5JM.png)

let으로 선언된 변수는 (1)선언 단계와 (2)초기화 단계가 나뉘어서 진행된다.

따라서,초기화되기 전에 변수에 접근하려고 하면 Reference Error가 발생한다.

이는 초기화 단계가 진행되지 않아 **메모리가 확보되지 않은 공간에 접근하려고 하였기 때문에 발생하는 에러**이다.

스코프의 시작 지점부터 초기화 시작 지점까지의 구간을 **일시적 사각 지대 (Temporal Dead Zone; TDZ)** 라고 한다.

```javascript
// let은 스코프의 선두에서 선언단계(1)만 실행된다.
// 이 지점부터 초기화 단계 시작지점까지 TDZ이므로, 여기서 변수를 참조할 수 없다.
console.log(hello); // ReferenceError: Cannot access 'hello' before initialization
let hello;          // 변수 선언문 -> 초기화 단계(2)가 실행된다.
console.log(hello); // undefined

hello = "hello";    // 할당문 -> 할당 단계(3)가 실행된다.
console.log(hello); // hello
```

## 출처

* [let, const와 블록 레벨 스코프 - poiemaweb.com](https://poiemaweb.com/es6-block-scope)
