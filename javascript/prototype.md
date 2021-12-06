# 프로토타입

## 생성자 안에서 메서드를 정의하는 방식의 문제점

```javascript
function Circle(center, radius) {
  this.center = center;
  this.radius = radius;
  this.area = function () {
    return Math.PI * this.radius ** 2;
  };
}
const c1 = new Circle({ x: 0, y: 0 }, 2.0);
const c2 = new Circle({ x: 0, y: 1 }, 4.0);
const c3 = new Circle({ x: 1, y: 0 }, 5.0);
```

Circle 안에 면적을 구하는 area 메서드를 정의하였다. Circle 생성자로 생성한 인스턴스 c1,c2,c3는 같은 메서드를 포함하게 된다.

![](https://i.imgur.com/SHXSN6D.png)

이는 같은 작업을 하는 메서드를 인스턴스 갯수만큼 생성하는 것이므로, 그만큼 메모리 낭비가 일어나게 된다.

## 프로토타입 객체

이러한 문제는 프로토타입 객체에 메서드를 정의하여 해결할 수 있다.

```javascript
function Circle(center, radius) {
  this.center = center;
  this.radius = radius;
}
Circle.prototype.area = function () {
  return Math.PI * this.radius ** 2;
};
const c1 = new Circle({ x: 0, y: 0 }, 2.0);
const c2 = new Circle({ x: 0, y: 1 }, 4.0);
const c3 = new Circle({ x: 1, y: 0 }, 5.0);

console.log(c1.area()); // 12.566370614359172
```

`__proto__` 프로퍼티는 그 객체에게 상속을 해 준 부모 객체를 가리킨다. 따라서 객체는 `__proto__` 프로퍼티가 가리키는 부모 객체의 프로퍼티를 사용할 수 있다.

![](https://i.imgur.com/0nuQQqR.png)

인스턴스 c1, c2, c3는 생성자로 Circle을 사용하였고, `__proto__`는 생성자인 Circle의 prototype object를 가리키고 있다. 따라서, 인스턴스들은 Circle 프로토타입 객체가 갖고 있는 area 메서드를 사용할 수 있게 된다. 이렇게 함으로써 인스턴스에 메서드를 추가하지 않고도, 인스턴스들은 메서드를 사용할 수 있다. 결과적으로 메모리 낭비를 막을 수 있다.

## 프로토타입 객체의 프로퍼티

함수가 정의될 때 `함수 객체`와 `Prototype Object` 이 생긴다.

![](https://i.imgur.com/wwNdDLL.png)

1. **함수 객체**는 `prototype` 프로퍼티를 갖고 있다. 
   * prototype은 **Prototype Object를 가리킨다**. 
   * 생성된 Prototype Object는 함수 객체의 `prototype`  프로퍼티를 통해 접근할 수 있다. 
   * 이 때, prototype object는 함수 객체만 갖고 있다. 단일 객체가 만들어질 때는 생기지 않는다. 
2. Prototype Object는 `constructor` 와 `__proto__` 를 갖고 있다. 
   * constructor는 **생성된 함수 객체의 참조**를 갖고 있다. 
   * `__proto__` \(`[[Prototype]]`\)는 `Prototype Link` 로서 생성자의 Prototype Object를 가리킨다.
   * 프로토타입 객체의 `__proto__`는 기본적으로 Object.prototype을 가리킨다. 즉, 프로토타입 객체의 프로토타입은 Object.prototype이다. 

## 프로토타입 체인

프로토타입 체인이란 위에서 봤던 프로토타입을 상속해서 만들어지는 객체들관의 연관관계를 의미한다. 그림에서 `__proto__` 프로퍼티들간 이어진 점선을 타고 가다보면 최종적으로 Object 객체의 prototype Object에 다다르는 것을 알수 있다. 그렇기 때문에 **자바스크립트의 모든 객체는 Object 객체에서부터 파생되어 나온 자식들**이라고 하는 것이다

프로토타입 체인을 따라 최상위 객체까지 도달할때까지 해당 메소드를 찾지 못한다면 `undefined` 이 되는 것이다.

## 프로토타입 객체 교체

* 함수 객체의 프로토타입 프로퍼티를 교체할 때는 교체할 객체에 constructor 프로퍼티로 생성자의 참조를 대입해야 한다.

  \`\`\`javascript

  function Circle\(center, radius\) {

  this.center = center;

  this.radius = radius;

  }

  Circle.prototype = {

  constructor: Circle,

  area\(\) {

    return Math.PI  _this.radius \*_ 2;

  },

  };

const c1 = new Circle\({ x: 0, y: 0 }, 2.0\); console.log\(c1.area\(\)\); // 12.566370614359172

```text
## 인스턴스가 프로토타입을 상속받는 시기
- 인스턴스의 프로토타입은 생성자가 인스턴스를 생성할 때 갖고 있던 프로토타입 객체이다. 
- 인스턴스 생성 후, 생성자의 prototype 프로퍼티를 다른 객체로 교체하여도 인스턴스의 프로토타입은 바뀌지 않는다.

```javascript
function Circle(center, radius) {
  this.center = center;
  this.radius = radius;
}
const c1 = new Circle({ x: 0, y: 0 }, 2.0);
Circle.prototype = {
  constructor: Circle,
  area() {
    return Math.PI * this.radius ** 2;
  },
};

console.log(c1.area()); // TypeError: c1.area is not a function
```

c1 인스턴스의 프로퍼티는 **Circle이 생성되는 시점의 프로퍼티를 상속받게 된다**. c1인스턴스 생성 후, Circle의 프로토타입 프로퍼티를 교체하여도 교체한 객체로부터는 프로퍼티를 상속받지 않는다.

* 인스턴스 생성 후 프로토타입 객체에 프로퍼티를 추가한다면, 인스턴스는 추가된 프로퍼티를 사용할 수 있다. 

  \`\`\`javascript

  function Circle\(center, radius\) {

  this.center = center;

  this.radius = radius;

  }

  const c1 = new Circle\({ x: 0, y: 0 }, 2.0\);

  Circle.prototype.area = function \(\) {

  return Math.PI  _this.radius \*_ 2;

  };

console.log\(c1.area\(\)\); // 12.566370614359172

\`\`\`

## 출처

* 모던 자바스크립트 입문
* [프로토타입 이해하기](https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)
* [자바스크립트 기초 - Object prototype 이해하기](http://insanehong.kr/post/javascript-prototype/)

