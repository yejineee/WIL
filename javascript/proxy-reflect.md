# Proxy와 Reflect

## Proxy와 Reflect

## 🍊 Proxy Objects

`Proxy` 객체는 다른 객체의 proxy를 만드는데, 그 객체의 기초적인 operation을 가로채서 재정의한다.

### Description

* constructor로 호출되면, **Proxy exotic objecr를 생성하고 초기화**한다.
* function으로서 호출되도록 의도되지 않았다. 만약, proxy를 함수처럼 호출하면 exeption이 던져진다.

### Proxy Constructor

* `[[Prototype]]`이 내부적으로 있고, 그 값은 `Function.prototype`이다.
* `prototype` 프로퍼티를 갖고 있지 않다. Proxy exotic objects가 초기화하는데 필요한 `[[Prototype]]`을 내부적으로 갖고 있지 않기 때문이다.
* 참고 : `[[Prototype]]` vs prototype 프로퍼티
  * `[[Prototype]]` : `__proto__`
    * 객체는 생성자의 프로토타입 객체를 가리킨다.
    * 함수 객체는 Function.prototype을 가리킨다.
  * prototype 프로퍼티
    * 함수객체만 갖고 있는 프로퍼티이다.
    * 프로토타입 객체\(Prototype Object\)를 가리킨다.

```javascript
function Person(name) {
  this.name = name;
}

var foo = new Person('Lee');
console.log(Person.__proto__ === Function.prototype); // true

console.log(Person.prototype === foo.__proto__); // true
```

### Proxy \( target, handler \)

Proxy는 두 파라미터로 만들어진다.

* target : proxy하고 싶은 원래의 **객체**
* handler : 어떠한 operation을 가져와서, 어떻게 재정의할 것인지를 정의한 **객체**

프록시 핸들러 메서드는 [Proxy Handler - MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy) 문서를 참고하자.

#### handler.construct\(\)

handler.construct\(\)메서드는 `new` operator를 가로챈다.

```javascript
const handler = {
  construct(target, args, newTarget){
    console.log(`called : ${args.join(', ')}`);
    return {value: args[0] * 10}
  }
}
const prox = new Proxy(function(){}, handler)

const proxItem = new prox(1);
console.log(proxItem.value) // 10
```

#### handler.apply\(\)

handler.apply\(\)는 함수 호출을 가로챈다.

```javascript
function sum(a, b) {
  return a + b;
}

const handler = {
  apply: function(target, thisArg, argumentsList) {
    console.log(`Calculate sum: ${argumentsList}`);
    // expected output: "Calculate sum: 1,2"

    return target(argumentsList[0], argumentsList[1]) * 10;
  }
};

const proxy1 = new Proxy(sum, handler);

console.log(sum(1, 2));
// expected output: 3
console.log(proxy1(1, 2));
// expected output: 30
```

#### 예제

```javascript
const target = {
  msg1: 'hello',
  msg2:'everyone'
};

const handler1 = {};
const proxy1 = new Proxy(target, handler1);

console.log(proxy1.msg1); // hello
console.log(proxy1.msg2); // world

const handler2 = {
  get(target, prop, receiver){
    return "world"
  }
};

const proxy2 = new Proxy(target, handler2);
console.log(proxy2.msg1); // world
console.log(proxy2.msg2); // world


const handler3 = {
  get(target, prop, reciever){
    if(prop === 'msg2'){
      return'world';
    }
    return Reflect.get(...arguments);
  }
}

const proxy3 = new Proxy(target, handler3);

console.log(proxy3.msg1); // hello
console.log(proxy3.msg2); // world
```

Proxy를 사용할 때, `this binding`이 까다로운 부분이다. 어떠한 메소드이건 this가 Proxy에 바인딩 되어, intercept할 수 있기를 바란다. ES6에서 `Reflect`라는 새로운 기능이 추가되었고, 이것이 이 문제를 해결해준다.

```javascript
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)
```

## 🍊 Reflect Object

Reflect는 자바스크립트 operation을 가로챌 수 있는 메소드를 제공하는 빌트인객체이다. [proxy handler](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)와 같은 메소드를 제공한다.

### Description

* Reflect는 일반적인 객체이다.
* 함수 객체가 아니다 =&gt; constructible 하지 않ㄴ다.
* Reflect 내부에는 `[[Construct]]` method가 내부에 없다 =&gt; `new` operator를 사용하여 constructor로 사용할 수 없다. 
* Reflect는 `[[Call]]` internal method가 없다 =&gt; 함수처럼 호출될 수 없다. 
* 모든 Reflect의 메소드와 프로퍼티는 static하다. 

### [Static Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect#static_methods)

### 출처

* [Proxy와 Reflect - 모던 자바스크립트](https://ko.javascript.info/proxy)
* [Proxy - MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
* [Proxy - ECMA 2022](https://tc39.es/ecma262/#sec-proxy-objects)
* [Reflection - ECMA 2022](https://tc39.es/ecma262/#sec-reflection)
* [Reflect - MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
* [Proxy Handler - MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)

