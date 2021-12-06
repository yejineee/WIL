# 리액트 컴포넌트, 엘리먼트, 인스턴스

리액트 블로그에 있는 "[React Components, Elements, and Instances](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)" 글을 번역하면서 정리한 글이다\(~~아주 많이 틀릴 수 있음 주의~~\). 리액트의 컴포넌트, 엘리먼트, 인스턴스의 차이를 알아보자.

## 인스턴스 관리

리액트를 처음 배운다면, 컴포넌트 클래스와 인스턴스만 사용해봤을 것이다. 예를들어, Button 컴포넌트를 클래스를 만들어서 선언했을 것이다. 앱이 동작하고 있을 때, 이 컴포넌트의 여러 인스턴스가 스크린에 있을 것이며, 각각의 인스턴스는 각자의 프로퍼티와 로컬 상태를 갖고 있다. 이것이 전통적인 object-oriented UI 프로그래밍이다.

이러한 전통적인 UI 모델에서 자식 컴포넌트 인스턴스를 생성하고 없애는 것을 관리하는 것은 전적으로 당신의 몫이다. 만약 Form 컴포넌트가 Button 컴포넌트를 렌더링하고자 할 때, Form 컴포넌트는 인스턴스를 만들고, 새로운 정보를 수동으로 업데이트해야 한다.

```javascript
class Form extends TraditionalObjectOrientedView {
  render() {
    // Read some data passed to the view
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      // Form is not yet submitted. Create the button!
      this.button = new Button({
        children: buttonText,
        color: 'blue'
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // The button is visible. Update its text!
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      // Form was submitted. Destroy the button!
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // Form was submitted. Show the success message!
      this.message = new Message({ text: 'Success!' });
      this.el.appendChild(this.message.el);
    }
  }
}
```

각 컴포넌트 인스턴스는 그 DOM 노드와 자식 컴포넌트의 인스턴스에 대한 참조를 갖고 있어야 한다. 또한, 적절한 때에 그것들을 생성하고, 업데이트하고, 없애야 한다.

코드의 라인 수는 컴포넌트가 갖고 있는 상태의 수가 늘어남에 따라 그리고 부모가 자식 컴포넌트 인스턴스에 직접 접근함에 따라 점차 늘어날 것이다. 미래에 그것들을 분리하는 것을 더욱 어렵게 만든다.

리액트는 어떻게 다를까?

## 엘리먼트가 트리를 표현하다

리액트에서는 **element**가 이 문제를 해결하기 위해 등장했다.

> An element is a plain object describing a component instance or DOM node and its desired properties

엘리먼트는 **컴포넌트 인스턴스나 DOM 노드를 기술하는 순수한 객체**이다. 엘리먼트는 `component type`\(예를 들면, 버튼\)과 그 `properties`\(예를 들면, color\), 그리고 그 내부의 `child element`에 대한 정보만 갖고 있다.

엘리먼트는 실제 인스턴스는 아니다. 엘리먼트는 스크린에서 보고자 하는 것을 리액트에게 알려주는 방식이다. 엘리먼트에 어떠한 메소드도 호출할 수 없다. 엘리먼트는 그저 두 가지 필드에 대한 immutable description object이다. `type:(string | ReactClass)`, `props: Object`

## DOM Elements

엘리먼트의 타입이 string이면, 그것은 태그 이름으로 DOM node를 나타내는 것이고, props는 그 속성에 해당하는 것이다.

```javascript
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

위의 엘리먼트는 다음 HTML을 순수 객체로 표현한 방식이다.

```text
<button class='button button-blue'>
  <b>
    OK!
  </b>
</button>
```

각 엘리먼트가 어떻게 중첩되어 있는지를 주목하라. 컨벤션에 의해, 엘리먼트 트리를 만들고 싶을 때, 하나 이상의 child elements를 그것이 속한 엘리먼트의 children prop으로 명시하게 된다.

_중요한 것은 child와 parent 엘리먼트 모두, 그저 description일 뿐이지 실제 인스턴스가 아니라는 것이다._ 엘리먼트들을 만들 때, 스크린에서의 어떠한 것도 참조하지 않는다. 엘리먼트를 만들고 버리는 것은 크게 문제가 되지 않는다.

리액트 엘리먼트는 순회하기 쉬우며, 파싱할 필요도 없다. 실제 DOM elements보다 훨씬 더 가볍다. 리액트 엘리먼트는 그저 객체이기 때문이다. \(객체니까 바로 접근하면 되므로, 순회하기 쉽고 파싱할 필요도 없다고 하는 것 같다\)

## Component Elements

엘리먼트의 타입이 스트링이 아니라, 리액트 컴포넌트에 해당하는 클래스나 함수일 수도 있다.

```javascript
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

다음은 리액트의 핵심 아이디어이다.

> An element describing a component is also an element, just like an element describing the DOM node. They can be nested and mixed with each other.

컴포넌트를 표현하는 엘리먼트 또한 엘리먼트이다. DOM노드를 표현하는 엘리먼트가 엘리먼트인 것과 마찬가지인 것이다. 그것들은 서로 중첩될 수 있으며 섞일 수 있다.

이러한 특성은 컬러 속성을 가진 `Button` 타입의`DangerButton`을 정의할 수 있게 한다. 이 DangerButton은 `Button`이라는 것이 `<button>`, `<div>` 혹은 다른 어떠한 태그로 렌더링될지에 대한 걱정을 전혀하지 않게 된다.

```javascript
const DangerButton = ({ children }) => ({
  type: Button,
  props: {
    color: 'red',
    children: children
  }
});
```

하나의 엘리먼트 트리 안에 DOM과 컴포넌트 엘리먼트를 믹스하거나 매치시킬 수도 있다.

```javascript
const DeleteAccount = () => ({
  type: 'div',
  props: {
    children: [{
      type: 'p',
      props: {
        children: 'Are you sure?'
      }
    }, {
      type: DangerButton,
      props: {
        children: 'Yep'
      }
    }, {
      type: Button,
      props: {
        color: 'blue',
        children: 'Cancel'
      }
   }]
});
```

JSX라면 다음처럼 표현된다.

```jsx
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);
```

이러한 믹스 앤 매치는 컴포넌트끼리 서로 분리되게 하는데 도움이 된다. 이는 각 컴포넌트가 조합을 통해 두 _is-a_와 _has-a_관계가 배타적으로 표현될 수 있기 때문이다.

* Button is a DOM `<button>` with specific properties. \(is-a\)
* DangerButton is a Button with specific properties. \(is-a\)
* DeleteAccount contains a Button and a DangerButton inside a `<div>`. \(has-a\)

## Component Encapsulate Element Trees

리액트가 함수나 클래스 타입의 엘리먼트를 보게 된다면, 해당하는 props에 관하여 그 컴포넌트가 어떠한 엘리먼트를 렌더링할지 물어본다.

다음과 같은 엘리먼트를 본다면,

```javascript
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

리액트는 `Button` 컴포넌트에게 무엇을 렌더링할 것인지를 물어본다. 그 Button 컴포넌트는 다음의 엘리먼트를 반환한다.

```javascript
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

리액트는 이 과정을 페이지의 모든 컴포넌트에게 그 아래에 있는 DOM 태그 엘리먼트를 알 때까지 반복한다.

위에 있던 `Form` 예시는 리액트로 다음처럼 표현될 수 있다.

```javascript
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // Form submitted! Return a message element.
    return {
      type: Message,
      props: {
        text: 'Success!'
      }
    };
  }

  // Form is still visible! Return a button element.
  return {
    type: Button,
    props: {
      children: buttonText,
      color: 'blue'
    }
  };
};
```

리액트 컴포넌트에게, props는 Input이며, element tree가 output이 된다.

반환된 element tree는 DOM node를 설명하는 엘리먼트와 다른 컴포넌트를 설명하는 엘리먼트가 포함될 수 있다. 이는 그 내부의 DOM 구조에 기대지 않고, 독립적인 UI를 구성할 수 있게 해준다.

리액트가 instance를 생성하고, 업데이트하고, 없앤다. 우리는 그 인스턴스를 컴포넌트로부터 반환한 엘리먼트들로 표현하며, 리액트는 그 인스턴스를 관리한다.

## 컴포넌트는 클래스이거나 함수일 수 있다

위 코드에서 `Form`, `Message`, `Button`은 리액트 컴포넌트이다. 이것들은 함수로 쓰여질 수 있고, `React.Component`로 부터 내려온 클래스로 쓰여질 수도 있다. 컴포넌트를 정의하는 세 가지 방식들은 거의 동일하다.

```javascript
// 1) As a function of props
const Button = ({ children, color }) => ({
  type: 'button',
  props: {
    className: 'button button-' + color,
    children: {
      type: 'b',
      props: {
        children: children
      }
    }
  }
});

// 2) Using the React.createClass() factory
const Button = React.createClass({
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
});

// 3) As an ES6 class descending from React.Component
class Button extends React.Component {
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
}
```

컴포넌트가 클래스로 정의되면, 함수 컴포넌트보다 조금 더 파워풀하다. **클래스형 컴포넌트**는 로컬 상태를 저장할 수 있으며, 그에 해당하는 DOM node가 생성되거나 삭제되었을 때, 커스텀 로직을 수행할 수 있다.

**함수형 컴포넌트**는 덜 파워풀하지만, 더 간단하며, `render()` 메서드로 클래스형 컴포넌트처럼 행동할 수 있다.

클래스형에만 있는 특징이 필요하지 않는다면, 함수형 컴포넌트를 사용할 것을 권장한다.

그러나, 함수나 클래스나 그 바탕에 있어서는 리액트에게 있어서 모두 컴포넌트이다. 그 컴포넌트들은 props를 인풋으로 받으며, 엘리먼트를 아웃풋으로 반환한다.

## Top-down Reconciliation \(하향식 조화\)

다음을 호출했다고 하자.

```javascript
ReactDOM.render({
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}, document.getElementById('root'));
```

리액트는 Form 컴포넌트에게 주어진 props로 어떠한 엘리먼트 트리를 반환할 것인지를 물어본다. 리액트는 더 간단한 primitive 측면에서 컴포넌트 트리에 대한 이해를 점차 refining하게 된다.

```javascript
// React: You told me this...
{
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}

// React: ...And Form told me this...
{
  type: Button,
  props: {
    children: 'OK!',
    color: 'blue'
  }
}

// React: ...and Button told me this! I guess I'm done.
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

이것은 리액트가 `reconciliation`을 호출하는 과정의 일부이다. Reconciliation은 `ReactDOM.render()`나 `setState()`를 호출할 때, 시작된다.

`reconciliation`이 끝날 때, 리액트는 DOM tree의 결과를 알게 되며, `react-dom`이나 `react-native`같은 renderer가 돔 노드를 업데이트하기 위해 필요한 작은 변화의 집합을 적용한다.

이러한 **점진적인 정제 작업은 리액트앱이 최적화하기 쉽다는 것의 이유**가 된다.

컴포넌트 트리의 어떠한 부분이 너무 커져서 리액트가 효과적으로 방문할 수 없을 때, "refining"하는 작업을 스킵하고, 트리의 특정 파트에 다른 점이 있는지를 찾으며 그 props에 변화가 없는지를 파악하게 한다.

props가 immutable하다면, 그 props가 변화했는지 파악하는 것을 계산하는 것은 매우 빠르다. 따라서 **리액트와 immutability 작업은 함께 할 때 매우 좋고, 적은 노력으로도 큰 최적화가 이루어진다.**

이 블로그는 컴포넌트와 엘리먼트에 대해서만 많이 말하고, 인스턴스에 대해서는 그다지 많이 말하지 않은 것을 알 수 있었을 것이다. 인스턴스는 사실 대부분의 객체 지향적인 UI 프레임워크보다 리액트에서 훨씬 중요하지 않기 때문이다.

클래스로 선언된 컴포넌트만이 인스턴스를 갖고 있으며, 절대로 인스턴스를 직접 만들 수 없다. 이는 리액트가 사용자를 위해 그렇게 한 것이다. 부모 컴포넌트 인스턴스가 자식 컴포넌트 인스턴스에 접근하는 메커니즘은 존재하지만, 그것은 오직 피할 수 없는 행동\(예를 들면, field에 포커스를 세팅하는 것\)에서만 사용되며, 일반적으로는 반드시 피해야 한다.

리액트는 모든 클래스 컴포넌트의 인스턴스를 만드는 것을 관리하므로, 메소드와 로컬 State를 갖는 객체 지향적인 방식으로 컴포넌트를 작성할 수 있다. 그러나 인스턴스는 리액트 프로그래밍 모델에서 그다지 중요하지 않으며, 인스턴스는 리액트 그 자체에 의해서 관리된다.

## 요약

* **Element**
  * 엘리먼트는 DOM node나 다른 컴포넌트에 대해 스크린에 보여졌으면 하는 것을 나타내는 순수 객체이다.
  * 엘리먼트는 다른 엘리먼트를 props로 갖고 있을 수 있다.
  * React element를 만드는 것은 매우 가벼운 작업이다\(cheap\).
  * 엘리먼트가 생성되면, 절대로 변화되지 않는다.
  * 엘리먼트를 생성하기 위해서는 `React.createElement()`나 `JSX`나 `element factory helper`를 사용해야 한다. 실제 코드에서 엘리먼트를 plain object로 쓰면 안된다. 그저 실제로는 엘리먼트가 plain object라는 것만 알아두자.
* **Component**
  * 컴포넌트는 `render()`메서드가 있는 클래스나 함수로 정의될 수 있다.
  * 두 방식 모두 props를 인풋으로 받으며, element tree를 output으로 반환한다.
  * 어떠한 컴포넌트가 props를 인풋으로 받을 때는, 특정 부모 컴포넌트가 타입과 그 props가 있는 element를 반환했기 때문이다. 이를 통해 props는 부모에서 자식으로 전달된다는 것을 알 수 있다.
* **Instance**
  * 인스턴스는 클래스형 컴포넌트에서 `this`로 참조하는 것이다. 이는 로컬 상태와 라이프사이클 이벤트에 대응하는 것에 유용하다.
  * 함수형 컴포넌트는 인스턴스를 전혀 갖고 있지 않다. 클래스 컴포넌트만이 인스턴스를 갖고 있다. 그러나, 리액트가 인스턴스를 만드는 것을 관리하므로, 직접 컴포넌트 인스턴스를 만들지 않아도 된다.

### 출처

* [React Components, Elements, and Instances](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)

