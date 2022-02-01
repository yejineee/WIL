# state 처리 - setState와 state hook

Created: October 28, 2020 2:00 AM

리액트에서 컴포넌트 내의 상태를 관리하는 방법을 클래스 컴포넌트와 useState 훅을 사용한 함수 컴포넌트로 알아본다. 

이 문서에서 쓰인 예시는 [codesandbox](https://codesandbox.io/s/study-react-17-vtpcy?file=/src/App.js)에서 확인할 수 있다.

- version
    - React 17.0.2

# ✔ 클래스 컴포넌트에서의 state 처리

```jsx
import React from "react";

class ClassCount extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  onClick() {
    this.setState((state) => ({
      count: state.count + 1
    }));
    this.setState((state) => ({
      count: state.count + 1
    }));
  }

  render() {
    return (
      <div>
        <h1> class component </h1>
        <p> click {this.state.count}</p>
        <button onClick={() => this.onClick()}>click</button>
      </div>
    );
  }
}

export default ClassCount;
```

### state 선언

```jsx
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```

React.Component를 extend한 클래스의 constructor 내에서 this.state에 상태의 초기값을 정의한다.

클래스 컴포넌트에서 state는 **object**여야 한다.

### state 변경

- state 변경은 `setState` 를 통해 이뤄져야 한다. 직접 state를 변경하면 안된다.
    
    ```jsx
      onClick() {
        this.setState((state) => ({
          count: state.count + 1
        }));
        this.setState((state) => ({
          count: state.count + 1
        }));
      }
    
    ```
    
- state 변경은 `비동기적`일 수 있다.
    
    리액트에서는 성능을 위해 여러 setState() 호출을 batch해놓고, 한 번의 update를 수행한다.
    
    따라서 이전 state값을 받아서 처리해야할 때는 setState에 함수를 넣고, 이전 state를 argument로 받아야 한다.
    
    함수의 첫 번째 인자는 이전 state 값이고, 두 번째 인자는 props이다.
    
    ```jsx
    // Wrong
    this.setState({
      counter: this.state.counter + this.props.increment,
    });
    
    // Correct
    this.setState((state, props) => ({
      counter: state.counter + props.increment
    }));
    ```
    

### 이벤트 핸들러 등록

클릭 이벤트 핸들러 등록은 `onClick` 에 등록한다.

이 때 this binding이 컴포넌트에 묶여있게 하기 위해, `화살표 함수`로 이벤트 핸들러를 등록한다.

```jsx
<button onClick={() => this.onClick()}>click</button>
```

# ✔ State Hook - useState

기존에는 컴포넌트 내에서 React state를 갖고 있으려면, 클래스 컴포넌트가 필요했다. 

Hooks이라는 기능이 도입되면서, 함수 컴포넌트에서도 state를 처리할 수 있게 되었다.

함수 컴포넌트에서 state를 처리하게 해주는 hook이 `useState` 이다. 

### state 선언

state 등록은 `useState`에 state의 초기값을 넣어서 등록한다.

이때 클래스 컴포넌트에서의 state 선언은 object여야했지만, useState훅에서는 객체, 배열, primitive type 등 어떠한 형태로도 가능하다.

만약 여러 개의 state가 필요하다면, 여러 useState를 추가하면 된다.

```jsx
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

### state 변경

useState가 반환하는 배열의 두 번째 요소는 state를 변경할 수 있게 해주는 함수이다. 이는 클래스 컴포넌트에서의 this.setState와 유사하다.

setState에 변경하려는 값을 넣으면 된다.

```jsx
setState(newState);
```

만약, 변경하려는 state가 이전 state값을 기반으로 변경해야 한다면, setState에 함수를 넣어야 한다. 이를 `functional update`라고 한다.

setState가 받는 함수의 첫 번째 인자는 이전 state값이다.

```jsx
function onClickButton() {
    setCount((count) => count + 1);
    setCount((count) => count + 1);
  }
```

### 이벤트 핸들러 등록

클릭시 처리할 이벤트 핸들러를 등록한다. 

```jsx
<button onClick={onClickButton}>click</button>
```

### Lazy initial state

useState의 초기값은 첫 번째 렌더링에서만 사용되고, 이후 렌더링에서는 사용되지 않는 값이다.

만약 초기값을 계산하는 과정이 expensive하다면, useState에 함수를 넣어서, 초기 렌더링에만 실행되도록 할 수 있다.

- Bad
    
    ```jsx
    const getInitialValue = () => {
      console.log("compute initial value"); // 렌더링 될 때마다 실행됨.
      return 0;
    };
    
    function Count() {
      const [count, setCount] = useState(getInitialValue());
    	...
    ```
    
- Good
    
    ```jsx
    const getInitialValue = () => {
      console.log("compute initial value"); // 초기 렌더링에서만 실행됨.
      return 0;
    };
    
    function Count() {
      const [count, setCount] = useState(() => getInitialValue());
    ```
    

### Bailing out of a state update

만약, setState로 변경되는 값이 **이전 state값과 같다면, 리액트는 children 컴포넌트를 리렌더링하거나, effect를 호출하지 않는다**.

이전 state와 같은지는 `[Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#description)` 를 사용하여 비교한다.

```jsx
Object.is([], []);                // false
var foo = { a: 1 };
var bar = { a: 1 };
Object.is(foo, foo);              // true
Object.is(foo, bar);              // false
```

# Reference

[State and Lifecycle - React](https://reactjs.org/docs/state-and-lifecycle.html)

[Using the State Hook - React](https://reactjs.org/docs/hooks-state.html)

[Hooks API Reference - React](https://reactjs.org/docs/hooks-reference.html#usestate)