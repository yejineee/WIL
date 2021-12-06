# Virtual DOM

virtual DOM은 리액트가 사용하게 되면서 유명해졌다. 이 글에서 virtual DOM이 무엇인지, DOM과는 어떻게 다른지, 어떻게 사용하는지 알아볼 것이다.

## 왜 virtual DOM이 필요할까?

DOM은 두 파트로 나뉘어진다.

1. 객체 기반의 HTML document 표현
2. 그 객체를 조작하는 API

만약 아래의 HTML에서 첫 번쨰 아이템의 이름을 "List item one"으로 변경하고, 두 번째 아이템을 추가한다고 해보자.

```text
<!doctype html>
<html lang="en">
 <head></head>
 <body>
    <ul class="list">
        <li class="list__item">List item</li>
    </ul>
  </body>
</html>
```

```javascript
const list = document.querySelector(".list")
const firstItem = list.children[0]
firstItem.textContent = 'List item one';
const secondItem = document.createElement('li');
secondItem.classList.add('list__item');
secondItem.textContent = 'List item two';
list.appendChild(secondItem)
```

DOM이 1998년에 처음 나왔을 때는 페이지의 내용을 생성하고 업데이트하는데 DOM API를 훨씬 적게 의존하였다.

작은 범위에서 간단한 메소드를 사용하는 것은 괜찮지만, 매 초마다 페이지의 여러 엘리먼트를 바꾼다면, 지속적인 query와 DOM을 업데이트하는 것은 매우 비용이 많이 들게 된다.

게다가, 넓은 범위를 업데이트하는 비용이 큰 operation을 사용하는 것이, 특정 부분을 찾아서 update하는 것보다 더 간단하다. 아래의 예시처럼 innerHTML로 전체 list를 교체하는 것이 훨씬 더 간단하다.

```javascript
const list = document.querySelector(".list")
list.innerHTML = `
    <li class='list__item'>list item one</li>
    <li class='list__item'>list item two</li>
`
```

이 예제에서는 두 방법에서의 성능 차이가 크게 두드러지지는 않는다. 그러나, 웹 페이지가 커짐에 따라, 필요한 부분을 찾아내서 업데이트하는게 더 중요해진다.

## VDOM은 그걸 해냅니다...

VDOM은 DOM을 자주 업데이트해야하는 문제를 좀 더 성능이 좋은 방식으로 해결하기 위해 만들어졌다. DOM이나 shadow DOM과는 다르게, VDOM은 공식적인 명세가 존재하지 않는다.

VDOM은 original DOM의 복사본으로 생각하면 된다. 이 복사본이 DOM API 없이 자주 조작되고 업데이트된다. VDOM에 모든 업데이트가 완료되면, original DOM에서 어떠한 변화가 일어나야하는지를 볼 수 있다. 그리고, 그 영역을 특정하여 최적화된 방식으로 변화를 만들어낼 수 있다.

## VDOM은 어떻게 생겼나

VDOM은 일반적인 자바스크립트 객체이다. 아래의 DOM 트리는 JS 객체로 나타내질 수 있다. ![](https://i.imgur.com/eFbIkKJ.png)

```javascript
const vdom = {
    tagName: "html",
    children: [
        { tagName: "head" },
        {
            tagName: "body",
            children: [
                {
                    tagName: "ul",
                    attributes: { "class": "list" },
                    children: [
                        {
                            tagName: "li",
                            attributes: { "class": "list__item" },
                            textContent: "List item"
                        } // end li
                    ]
                } // end ul
            ]
        } // end body
    ]
} // end html
```

이러한 객체를 VDOM이라고 생각할 수 있다. DOM처럼 HTML document를 객체 기반으로 표현한 것이다. 그러나, 이는 순수 JS 객체이므로, 자유롭고 자주 조작할 수 있다. 실제 DOM을 건들이지 않으면서 조작할 수 있게 되는 것이다.

전체 객체를 사용하는 것보다, VDOM의 작은 부분에 작업을 하는 것이 더 일반적이다. 예를 들어, 아래와 같은 list 컴포넌트로 작업을 하게 된다.

```javascript
const list = {
    tagName: "ul",
    attributes: { "class": "list" },
    children: [
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item"
        }
    ]
};
```

## VDOM의 아래에서는

VDOM은 DOM의 성능과 사용성 문제를 어떻게 해결할까?

VDOM은 DOM에서 필요한 변화를 찾아내고, 특정한 부분을 업데이트하는 방식으로 VDOM을 사용한다.

### 1. VDOM의 copy를 만들기

먼저 할일은 VDOM의 복사본을 만드는 것이다. 여기에는 변화를 주어야하는 부분들이 적용되어있다\(list item two가 추가됨!\). 실제 DOM API는 필요없기 때문에, 그냥 새로운 객체를 만들어주면 된다.

```javascript
const copy = {
    tagName: "ul",
    attributes: { "class": "list" },
    children: [
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item one"
        },
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item two"
        }
    ]
};
```

이 copy는 원래의 VDOM\(여기서는 list\)와 비교하여 diff라는 것을 생성한다. diff는 이런식으로 생겼다.

```javascript
const diffs = [
    {
        newNode: { /* new version of list item one */ },
        oldNode: { /* original version of list item one */ },
        index: /* index of element in parent's list of child nodes */
    },
    {
        newNode: { /* list item two */ },
        index: { /* */ }
    }
]
```

diff는 실제 DOM을 어떻게 변경시킬지를 알려준다. 모든 diff가 모여지게 되면, DOM에 필요한 부분만을 업데이트시킨다.

예를들어, 각 diff를 확인하여, 새로운 child를 추가하거나, 기존 것을 변경하는 식으로 이루어진다.

```javascript
const domElement = document.getElementsByClassName("list")[0];

diffs.forEach((diff) => {

    const newElement = document.createElement(diff.newNode.tagName);
    /* Add attributes ... */

    if (diff.oldNode) {
        // If there is an old version, replace it with the new version
        domElement.replaceChild(diff.newNode, diff.index);
    } else {
        // If no old version exists, create a new node
        domElement.appendChild(diff.newNode);
    }
})
```

## 출처

* [Understanding the Virtual DOM](https://bitsofco.de/understanding-the-virtual-dom/)

