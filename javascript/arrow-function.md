# 화살표함수

## ✔️단축 문법

1. function을 생략해도 된다.
2. 함수에 매개변수가 하나라면 괄호도 생략 가능하다.
3. 함수 바디가 표현식이 하나라면, 중괄호와 return문도 생략할 수 있다.
4. 객체 반환시 소괄호를 사용해야 한다. 

```jsx
// exp1
setTimeout(function() {
    console.log("settimeout") ;
}, 1000) ;

//arrow function으로 만들기
setTimeout(() => {
    console.log("settimeout") ;
}, 1000) ;

// exp2
const newArr = [1,2,3,4,5].map(function(value, index, object){
    return value*2 ; 
})
// arrow function
const newArr = [1,2,3,4,5].map((value) => value*2 ); 
)

//exp4
() => ({ a : 1 });
```

## ✔️this

### 일반 함수의 this

* this는 함수 호출 방식에 의해 this에 바인딩할 객체가 **동적**으로 바인딩된다.
* 생성자 함수와 메서드를 제외한 모든 함수\(내부함수, 콜백함수\) 내부의 this는 전역객체 window를 가리킨다.

```jsx
function Prefixer(prefix){
  this.prefix = prefix;
}

Prefixer.prototype.prefixArray = function (arr) {
//(A)
  return arr.map(function(x){
    //(B)
    return this.prefix + ' ' + x;
  });
};

const pre = new Prefixer('hi');
console.log(pre.prefixArray(['lee', 'kim'])); // [ 'undefined lee', 'undefined kim' ]
```

* \(A\)지점에서의 this는 생성자 함수의 인스턴스인 pre이다.

생성자의 prototype 메서드 안에 있는 this는 그 생성자로 생성한 객체를 가리키기 때문이다.

```jsx
// A 지점의 this
Prefixer { prefix: 'hi' }
```

* \(B\)지점에서의 this는 전역객체인 global이다\(브라우저에서는 window이고 node.js 환경에서는 global 객체\).

  this는 생성자 함수와 객체의 메서드를 제외한 모든 함수 내부의 this는 전역객체를 가리키기 때문이다.

```jsx
// B 지점에서의 this
Object [global] {
  global: [Circular],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  setInterval: [Function: setInterval],
  setTimeout: [Function: setTimeout] { [Symbol(util.promisify.custom)]: [Function] },
  queueMicrotask: [Function: queueMicrotask],
  clearImmediate: [Function: clearImmediate],
  setImmediate: [Function: setImmediate] {
    [Symbol(util.promisify.custom)]: [Function]
  }
}
```

### 콜백함수 내부의 this 설정

콜백함수 내부의 this가 생성자 함수의 인스턴스를 가리키게 하는 법은 bind를 사용하는 것이다.

* bind

```jsx
func.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind의 첫 번째 인수는 대상 함수의 this값이다.

```jsx
function Prefixer(prefix){
  this.prefix = prefix;
}

Prefixer.prototype.prefixArray = function (arr) {
  return arr.map(function(x){
    return this.prefix + ' ' + x;
  }.bind(this));
};

const pre = new Prefixer('hi');
console.log(pre.prefixArray(['lee', 'kim'])); //[ 'hi lee', 'hi kim' ]
```

### 화살표함수의 this

* 화살표 함수는 함수를 선언할 때 this에 바인딩할 객체가 **정적**으로 결정된다.
* 화살표함수의 this는 언제나 **상위 스코프의 this**를 가리킨다. → Lexical this라고 한다.

  → 화살표 함수 바깥의 this 값이 화살표 함수의 this 값이 된다.

* 화살표 함수는 bind로 this를 설정하는 것의 syntactic sugar이다.

```jsx
function Prefixer(prefix){
  this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
  // this는 상위 스코프인 prefixArray 메소드 내의 this를 가리킨다.
  return arr.map(x => `${this.prefix}  ${x}`);
};

const pre = new Prefixer('Hi');
console.log(pre.prefixArray(['Lee', 'Kim'])); //[ 'Hi  Lee', 'Hi  Kim' ]
```

## ✔️화살표 함수를 사용하면 안되는 경우

### 1. 메서드

```jsx
const person = {
  name: 'yejin',
  sayHi: () => console.log(`hi ${this.name}`),
}

person.sayHi(); // hi undefined
```

sayHi 메서드 내부의 this는 상위 스코프의 this이므로 전역객체 window를 가리킨다.

따라서 화살표 함수로 메서드를 정의하는 것은 바람직하지 않다.

\(이제서야 객체 안에서 화살표 함수를 사용하면 this로 같은 객체 내부의 어떠한 것도 가져오지 못하는 이유를 알았다.\)

이 경우 ES6의 축약 메서드 표현을 사용하는 것이 좋다. 이 표현을 사용하면 funtion 키워드를 생략하면서, this는 동적으로 결정된다.

```jsx
const person = {
  name: 'Lee',
  sayHi() { // === sayHi: function() {
    console.log(`Hi ${this.name}`);
  }
};

person.sayHi(); // Hi Lee
```

sayHi 내부의 this는 person객체이다.

```jsx
{ name: 'Lee', sayHi: [Function: sayHi] }
```

### 2. prototype

화살표 함수로 정의된 메서드를 prototype에 할당하는 경우에도 같은 문제가 발생한다.

```jsx
const person = {
  name: 'Lee',
};

Object.prototype.sayHi = () => console.log(`Hi ${this.name}`);

person.sayHi(); // Hi undefined
```

이 때에도 sayHi 메서드 내부가 가리키는 this는 상위 스코프인 전역객체 window이다.

### 3. 생성자 함수

화살표 함수는 prototype 프로퍼티를 갖고 있지 않다. 따라서 생성자 함수로 사용할 수 없다.

```jsx
const Foo = () => {};

// 화살표 함수는 prototype 프로퍼티가 없다
console.log(Foo.hasOwnProperty('prototype')); // false

const foo = new Foo(); // TypeError: Foo is not a constructor
```

### 4. addEventListener 함수의 콜백 함수

addEventListener 함수의 콜백 함수를 화살표 함수로 정의하면 this가 상위 컨텍스트인 전역객체 this를 가리킨다.

```jsx
var button = document.getElementById('myButton');

button.addEventListener('click', () => {
  console.log(this === window); // => true
  this.innerHTML = 'Clicked button';
});
```

* addEventListener 함수의 콜백 함수 내 일반함수 사용하면, 그 콜백함수 내부의 this는 이벤트 리스너에

바인딩된 요소를 가리킨다.

```jsx
// Good
var button = document.getElementById('myButton');

button.addEventListener('click', function() {
  console.log(this === button); // => true
  this.innerHTML = 'Clicked button';
});
```

#### 참고문서

* [https://poiemaweb.com/es6-arrow-function](https://poiemaweb.com/es6-arrow-function)

