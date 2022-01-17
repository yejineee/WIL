# undefined vs null vs undeclared 변수

## undefined

* 선언되지 않았을 경우

```text
console.log(typeof v) // undefined
```

* 변수가 선언되었으나, 할당은 되지 않았을 경우, 자바스크립트 엔진에 의해 자동으로 할당되는 값이다.

```javascript
let a;
console.log(a); // undefined
console.log(typeof a); //undefined
```

## null

* 어떠한 참조값도 존재하지 않음을 의미한다. 
* 객체 참조의 연결을 해지하는 것을 말한다. 

```javascript
let b = null;
console.log(b); // null
console.log(typeof b); // Object
```

## null, undefined 검증

선언되지 않은 변수는 undefined이나, 선언되지 않은 변수를 사용하는 것은 문법 오류이다.

```text
console.log(null == undefined); // true
console.log(null === undefined); // false
console.log(qwer == undefined); // ReferenceError: qwer is not defined
```

## undeclared 변수

* 전역 스코프 

undeclared 변수란 선언하지 않은 변수로, 어떠한 키워드도 지정하지 않은 변수이다. 키워드를 지정하여 변수를 선언하는 것은, _그 변수의 유효범위를 지정하는 것_ 이다. undeclared 변수는 유효범위를 지정하지 않았으므로, 어떤 scope에서 선언을 했던지간에, **전역 객체의 프로퍼티**가 된다.

키워드를 지정하여, 변수의 유효범위를 지정한 declared 변수와 비교해보자. var 키워드로 선언된 변수는 함수 범위 스코프를 갖기 때문에, 선언된 함수 내에서만 참조할 수 있다. 전역 스코프에서 var 키워드로 선언된 변수를 참조하려고 하면 에러가 발생한다.

```javascript
function func() {
  var varKeyword = "var 로 선언된 변수";
  console.log(varKeyword); // var 로 선언된 변수
}

func();
console.log(varKeyword); // ReferenceError: varKeyword is not defined
```

선언하지 않은 변수는 전역 변수이므로, 전역 스코프에서 선언되지 않은 변수를 참조할 수 있다.

```text
function fn() {
  undeclared = "선언되지 않은 변수";
}

fn();
console.log(undeclared); // 선언되지 않은 변수
```

* undeclared 변수는 할당문을 만나기 전에 참조하려고 하면, 오류가 발생한다.

  ```text
  console.log(undc); // ReferenceError: undc is not defined
  undc = "선언되지 않은 변수";
  ```

## 정리

undefined는 그 변수가 참조하는 객체를 아직 지정하지 않았음을 뜻한다. undefined를 갖는 변수가 할당을 통해 값을 갖게 되고, 이 값을 해제할 때, null이 된다.

```javascript
let value; // 할당받지 않은 변수는 자동으로 undefined
console.log(value); // undefined

value = "value"; // 문자열을 할당 
console.log(value); // value;

value = null; // 문자열 객체를 해제하였으므로 null
console.log(value); // null
```

## 출처

* [What is the difference between null and undefined in JavaScript? - stackoverflow](https://stackoverflow.com/questions/5076944/what-is-the-difference-between-null-and-undefined-in-javascript)
* [front-end 개발자 인터뷰 문제 - javascript 영역](http://insanehong.kr/post/front-end-developer-interview-javascript/)
* [Effect of declared and undeclared variables Ask Question](https://stackoverflow.com/questions/15985875/effect-of-declared-and-undeclared-variables)

