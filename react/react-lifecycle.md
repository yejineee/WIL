# 리액트 생명주기

## 🍑 class형 컴포넌트로 알아보는 생명주기

매초마다 화면에 현재 시간이 업데이트되는 클래스형 컴포넌트이다.

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

* 호출 순서 요약 1. render에 `<ClocK />`이 전달되었을 때, React는 Clock의 constructor를 호출한다. constructor는 this.state를 현재 시간으로 초기화한다.
  1. 리액트가 render 메서드를 호출한다. **render 메소드로 인하여 리액트는 현재 화면에 출력해야할 것이 무엇인지를 알게 된다**. 그 때 리액트는 Clock의 렌더링 출력값을 DOM에 업데이트한다.
  2. Clock의 출력값이 DOM에 추가되었다는 것은 **마운트**가 되었다는 것이다. 이 때 `componentDidMount()` 생명주기 메서드를 호출한다. 이 메소드에서는 브라우저에 setInterval로 타이머를 설정하여, 매초 tick\(\) 메소드를 호출하게 한다.
  3. 매초 브라우저는 tick\(\)메서드를 호출한다. Clock 컴포넌트는 setState\(\)로 state를 변경시킨다. **리액트는 setState\(\)로 인하여 state가 변경된 것을 감지한다.** 리액트는 화면에 표시될 내용을 알아내기 위해, render\(\) 메소드를 호출한다. 브라우저는 변경된 this.state.date를 갖고 있는 엘리먼트를 DOM에 업데이트한다.
  4. Clock 컴포넌트가 DOM에서 한 번이라도 삭제된 적이 있다면, 리액트는 `componentWillUnmount()` 메소드를 호출하여 타이머를 삭제한다.

## 🍑 this binding 다시 짚고 넘어가자

componentDidMount 메소드에서는 setInterval 안에 콜백함수를 등록한다. 여기서는 화살표 함수로 감싸서 this.tick\(\)을 호출한다.

그냥 this.tick을 등록하면, setTimeout의 콜백이 실행될 때의 this는 전역객체이므로, 그때의 tick 메소드가 없어서 에러가 뜰 것이다.

this를 바인딩하는 다른 방법에 대해 생각해보았다. 이 예제에서 화살표 함수로 this.tick\(\)을 호출한 것은, 함수 스코프를 만들어 메소드를 호출하는 객체가 Clock Component가 되게 하여, 그 때의 this를 이 Clock 컴포넌트에 바인딩하게 하기 위함이라고 생각했다.

마찬가지로 function\(\){this.tick\(\)}을 콜백함수로 등록하여도, this가 Clock에 바인드되어 잘 동작하지 않을까 생각했다.

그러나, 예상과는 달리 에러가 발생하였다. tick은 함수가 아니라나... 왤까?

또다시 깨닫는다. function 키워드로 만든 함수의 this와 화살표함수의 this는 다르다... 또한, 일단 function 키워드로 만든 콜백함수 내부의 this는 전역객체이다. 따라서 전역개체의 this가 바인드되어, tick 메소드를 찾지 못한 것이다.

```javascript
  componentDidMount() {
    this.timerID = setInterval(
      function(){

        console.log(this) // Window
      },
      1000
    );
  }
```

화살표함수의 this는 정적으로 결정된다. 외부 스코프의 this를 가져오게 된다. 따라서, 즉, componentDidMount\(\) 함수의 this를 가져오게 되는데, 이는 메소드이므로, this가 Clock에 바인딩 된다. 따라서, 화살표함수로 정의된 콜백함수의 내부도 Clock을 가리킨 것이었다.

```javascript
  componentDidMount() {
    console.log(this); // Clock
    this.timerID = setTimeout(
    () => {
      this.tick(); 
      console.log(this); // Clock
    },
      1000
    );
  }
```

정말 this binding에 대해 잘 모르면, 에러를 만들어낼 수 밖에 없을 것이다... this binding에 대한 개념을 알고 있다고 생각했는데, 막상 예제를 보니 내가 잘 모르고있구나를 알았다. 이렇게 하나하나 깨닫고 가는거지 뭐..!

## 🍑 컴포넌트 생명주기

모든 컴포넌트가 `생명주기 메소드`를 갖고 있으며, 이 메소드를 오버라이딩하여 특정 시점에 코드가 실행되도록 할 수 있다.

![](https://i.imgur.com/aeWDNdH.png)

### ✨ 마운트

마운트가 될 때는 **컴포넌트의 리액트 인스턴스가 생성되어 DOM에 추가될 때**를 말한다.

* 마운트시 메소드 호출 순서 1. **constructor\(\)** 2. static getDerivedStateFormProps\(\) 3. **render\(\)** 4. **componentDidMount\(\)**
* **constructor\(\)** constructor는 마운트 되기 이전에 호출된다. 주로 constructor에서는 다음 두 가지 일을 처리한다.
  1. this.state 초기화
     * 생성자에서만 this.state를 직접 할당할 수 있고, 그 외의 메서드에서는 this.setState를 사용해야 한다.
  2. 인스턴스에 이벤트 처리 메서드 바인딩

```javascript
constructor(props) {
  super(props); // 반드시 호출해야 함.
  // 여기서 this.setState()를 호출하면 안 됩니다!
  this.state = { counter: 0 }; // state 초기화
  this.handleClick = this.handleClick.bind(this); // 이벤트처리 메서드 바인딩
}
```

이 때, state에 props를 복사하면 안된다. 여기서 color props가 변하여도, state에는 반영되지 않는다.

```javascript
constructor(props) {
 super(props);
 // 이렇게 하지 마세요!
 this.state = { color: props.color };
}
```

* **render\(\)**

  render\(\) 메서드는 클래스 컴포넌트에서 반드시 구현해야하는 유일한 메소드이다.

  render\(\) 함수는 **순수**해야 한다.

  * 컴포넌트의 props와 state를 변경하지 않아야 한다.
  * 호출될 때마다 동일한 결과를 반환해야 한다. 
  * 브라우저와 직접적인 상호작용을 하지 않는다. -&gt; VDOM 사용한다는 것인가?

    이 메소드는 `props`나 `state`를 활용하여, 아래의 것 중 하나를 반환해야 한다.

  * 리액트 엘리먼트 : 주로 JSX로 만들고, React.createElemenet로 만들 수 있다.
  * 배열과 Fragment : 여러 개의 엘리먼트를 반환할 수 있다.
  * portal : 별도의 DOM 하위 트리에 자식 엘리먼트를 렌더링한다.
  * 문자열과 숫자 : DOM에서 텍스트 노드로 렌더링된다.
  * Boolean, Null : 아무것도 렌더링하지 않는다.

```javascript
// Fragment 
render() {
  return (
    <React.Fragment> // 단축문법인 <>를 사용할 수 있음.
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

* **componentDidMount\(\)**

  componentDidMount는 **컴포넌트가 마운트된 직후, 즉 트리에 삽입된 직후에 호출**된다.

  DOM 노드가 있어야 하는 초기화 작업이 이 메서드에서 이루어지면 된다.

  예를 들면, _외부에서 데이터를 불러올 때_ 나, _네트워크 요청_ 을 보낼 경우에 적절한다.

### ✨ 업데이트

props나 state가 변경되면, 업데이트가 발생된다.

* 업데이트시 메소드 호출 순서 1. static getDerivedStateFormProps\(\) 2. shouldComponentUpdate\(\) 3. **render\(\)** 4. getSnapshotBeforeUpdate\(\) 5. **componentDidUpdate\(\)**
* shouldComponentUpdate\(\)

```javascript
shouldComponentUpdate(nextProps, nextState)
```

props나 state가 새로운 값으로 변하여서, 렌더링이 발생하기 직전에 호출된다.

초기 렌더링이나 forceUpdate\(\)가 사용될때는 호출되지 않는다.

기본값은 true이다. 만약, shouldComponentUpdate가 false를 반환하면, render\(\)와 componentDidUpdate\(\)는 호출되지 않는다.

* render\(\) 마운트 시 설명한 render\(\) 메서드와 같다.

  shouldComponentUpdate가 false를 반환하면, render는 호출되지 않는다.

* **componentDidUpdate\(\)**

```javascript
componentDidUpdate(prevProps, prevState, snapshot);
```

이 메서드는 **업데이트가 일어난 직후**에 호출된다. 초기렌더링이나 forceUpdate\(\)가 사용될 때는 호출되지 않는다.

컴포넌트가 업데이트 되었을 때, DOM을 조작할 때 활용할 수 있다.

이전 props와 비교하는 등의 조건문으로 감싸져 있지 않으면, 무한 반복이 발생할 수 있음을 주의해야 한다.

마찬가지로, shouldComponentUpdate가 False를 반환하면, componentDidUpdate는 호출되지 않는다.

### ✨ 언마운트

언마운트는 컴포넌트가 DOM에서 제거될 때를 말한다.

* 언마운트시 메소드 호출
  * componenWillUnmount\(\)
* **componentWillUnmount\(\)**

  컴포넌트가 마운트 해제되어 제거되기 직전에 호출된다.

  * 타이머 제거
  * 네트워크 요청 취소
  * componentDidMount 내에서 생성된 구독 해제 등의 필요한 정리 작업이 이루어야 한다.

    이 메소드 내에서는 setState\(\)를 호출하면 안된다.

## 참고

* [React.Component - React](https://reactjs.org/docs/react-component.html)

