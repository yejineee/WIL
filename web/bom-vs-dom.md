# BOM vs DOM

전역 객체의 초기 상태에는 DOM, BOM, 빌트인 객체가 설정되어있다. 여기서 DOM과 BOM이 무엇인지 알아보자.

## Document Object Model\(DOM\)

* DOM은 HTML과 XML 도큐먼트에 접근하고 조작할 수 있는 API라고 할 수 있다. 
* DOM은 도큐먼트의 구조를 노드의 계층적 구조로 정의하고 있다. 
* DOM은 XML 파일이나 HTML을 노드의 계층적 구조인 **트리 구조**로 변환한다. 
* 이 트리는 **도큐먼트의 내용과 노드 간의 관계**를 나타낸다. 

![](https://i.imgur.com/2Wcud2z.png)

```text
<!DOCTYPE html>
<html>
 <head>
  <title>My title</title>
 </head>
 <body>
   <h1>My header</h1>
   <a href='#'>My Link</a>
 </body>
</html>
```

![](https://i.imgur.com/JDxZUUV.png)

### Node Types

DOM은 여러 타입을 정의하여, 각각의 인터페이스를 다르게 구성하였다.

* Document : 모든 도큐먼트의 루트노드이다.
* DocumentType : doctype 태그로 나타낸 document type definition을 나타낸다.
* Element : 태그를 나타낸다.
* Attribute : tag의 attribute를 나타내는 key-value 쌍이다.
* Text: 노드의 내용을 나타낸다. 
* Comment : 도큐먼트의 주석이다. 

## Browser Object Model\(BOM\)

브라우저 창에 접근하고, 조작할 수 있도록 하는 것이 BOM이다. 자바스크립트는 BOM을 통해 브라우저와 상호작용할 수 있다. 공식적인 standard는 없지만, 브라우저는 거의 같은 특징을 갖도록 구현되었다.

![](https://i.imgur.com/rfqGskP.png)

### 전역 변수와 전역 메서드

* **`window` 객체는 root element**이다. 다른 모든 것을은 window 객체에 직간접적으로 붙어있다. 
* 브라우저의 **각 탭마다 고유의 window객체**가 있다. 즉, 같은 윈도우여도 탭끼리 window 객체는 공유되지 않는다.
* 전역 스코프에서 선언된 모든 변수와 함수는 전역 객체에 속하게 된다\(let, const 제외\). 즉, 전역 변수는 window 객체의 프로퍼티가 되며, 전역 함수는 window 객체의 메소드가 된다. 

```text
var foo = 'foo';
console.log(window.foo) // foo
```

* 전역 객체의 메서드와 프로퍼티는 [MDN](https://medium.com/@fknussel/dom-bom-revisited-cf6124e2a816)에서 확인할 수 있다. 

### location 객체

* 브라우저 주소창에 있는 URL을 읽고 조작할 수 있게 해주는 객체이다. 
* [location의 주요 메서드와 프로퍼티](https://developer.mozilla.org/en-US/docs/Web/API/Location)는 MDN에서 자세히 살펴볼 수 있다.
  * location.href
  * location.host
  * location.port
  * location.replace\(url\)
  * location.assign\(url\)

### history 객체

* 브라우저의 세션 히스토리를 조작할 수 있게 해주는 객체이다. 즉, 탭에서 방문했던 페이지나, 현재 페이지의 기록을 조작할 수 있게 한다. 
* history.length
* history.go\(integer\)
* history.back\(interger\)
* history.forward\(integer\)

### navigator 객체

* 유저가 사용하고 있는 브라우저나 OS 즉, user agent에 대한 상태나 정보를 제공하는 객체이다. 
* navigator.userAgent
* navigator.languages
* navigator.onLine
* navigator.getBattery\(\)

## 출처

* [What is the DOM and BOM in JavaScript?](https://stackoverflow.com/questions/4416317/what-is-the-dom-and-bom-in-javascript)
* [DOM & BOM Revisited](https://medium.com/@fknussel/dom-bom-revisited-cf6124e2a816)

