# 성능 최적화

## 🖍 레이아웃과 리페인트

브라우저 로딩 과정 중 스타일 이후의 과정 \(스타일 -&gt; 레이아웃 -&gt; 페인트 -&gt; 합성\)을 **렌더링**이라고 한다. 이 렌더링 과정은 상황에 따라 반복하여 발생할 수 있다.

### 레이아웃

![](https://i.imgur.com/OTL0zmM.png)

* **DOM이 추가/삭제되거나, 기하적인 영향\(넓이, 높이, 위치\)을 주는 CSS 속성값을 변경**되면, 렌더 트리는 재구성된다.
* 레이아웃 이후 과정부터 다시 수행하는 것을 **레이아웃** 또는 **리플로우**라고 한다.
* 레이아웃은 화면에서의 전체 픽셀을 다시 계산해야 하므로 _부하가 크다_. 따라서 불필요한 레이아웃이 발생하지 않도록 주의해야 한다.
* 기하적 영향을 주는 css 속성값 : height, width, left, right, font-size, ...

### 리페인트

![](https://i.imgur.com/qJUbojk.png)

* 기하적인 요소에는 영향을 주지 않는 CSS 속성값을 변경하면 레이아웃은 건너뛰고, 페인트부터 수행하게 된다. 이를 **리페인트**라고 한다.
* 이미 계산된 픽셀값을 이용하여 화면을 그리기 때문에, 레이아웃에 비해 부하가 적다.
* 기하적 영향이 없는 css 속성값 : color, visibility, text-decoration, ...

리플로우와 리페인트에 영향을 주는 css 속성은 [여기](http://goo.gl/lPVJY6)에서 확인할 수 있다.

## 🖍 웹 페이지 로딩 최적화

### ✨ 블록 리소스 최적화

파싱 중 블록 리소스가 발생할 수 있는데, css와 js가 블록 리소스에 해당한다.

#### css 최적화

* **CSS는 항상 HTML 문서 최상단**에 배치한다.

    CSSOM 트리는 CSS를 모두 해석해야 구성할 수 있다. 즉, CSSOM 트리가 구성되지 않으면, 렌더 트리가 만들어지지 않고 렌더링이 차단된다. 렌더링이 차단되지 않도록 HTML 문서 최상단에 배치한다.

```text
<head>
  <link href="style.css" rel="stylesheet" />
</head>
```

* 특정 조건에서만 필요한 CSS가 있을 때, **미디어 쿼리**를 사용한다.

    페이지를 인쇄하거나, 화면이 세로 모드일 경우에만 사용하는 CSS가 있다면, 해당 스타일을 사용하는 경우에만 로드할 수 있도록 `media` 속성을 명시하여 사용한다.

```text
<link href="style.css" rel="stylesheet" />
<link href="print.css" rel="stylesheet" media="print" />
<link href="portrait.css" rel="stylesheet" media="orientation:portrait" />
```

* 외부 스타일시트를 가져올 때 `@import` 사용은 피한다.

    @import를 사용하면, 브라우저는 스타일 시트를 병렬로 다운로드할 수 없어서, 로드 시간이 늘어날 수 있다.

```css
/* foo.css */
@import url("bar.css")
```

* 때에 따라 내부 스타일 시트를 사용한다.

    외부 스타일 시트를 가져올 때 발생하는 요청 횟수를 줄일 수 있다. 단, 내부 스타일 시트는 리소스 캐시를 사용할 수 없으므로, 필요한 경우에만 사용한다.

```text
<head>
  <style type="text/css">
    .wrapper {
      background-color: red;   
    }
  </style>
</head>
```

#### 자바스크립트 최적화

* **자바스크립트는 HTML 문서 최하단 \(`</body>`직전\)에 배치**한다.

    자바스크립트는 DOM트리와 CSSOM트리를 동적으로 변경할 수 있어, HTML 파싱을 차단한다. 또한, script 태그 이전까지 생성된 DOM에만 접근할 수 있다.

    `<script>` 태그를 만나면, 스크립트 실행이 완료될 때까지 DOM 트리 생성이 중단된다. 자바스크립트가 HTML 파싱을 차단하지 않게 하기 위해, HTML 문서 최하단에 배치한다.

  \`\`\`htmlembedded

  ......

```text
- `<script>`태그에 defer나 async 속성을 명시한다.
    defer, async는 DOM 트리와 CSSOM 트리를 변경하지 않겠다는 의미이므로, 브라우저가 파싱을 멈추지 않는다. 따라서, head에 자바스크립트 파일을 두어도 된다. 그러나, 지원하는 브라우저가 한정적이므로 사용에 유의한다.
    ![](https://i.imgur.com/SkzFiPZ.png)
    ![](https://i.imgur.com/GZnmhg4.png)
    - defer
        - defer는 스크립트를 '백그라운드'에서 다운로드한다. 따라서 스크립트를 다운로드 하는 중에도 HTML은 파싱을 할 수 있다. 
        - defer 스크립트 실행은 페이지 구성이 끝날때까지 지연된다. 
        - DOMContentLoaded 이벤트 발생 전에 지연 스크립트가 실행된다. 따라서 둘의 정확한 순서는 예측할 수 없다.
        - defer는 외부 스크립트에만 유효하다.
        - 지연 스크립트는 HTML에 추가된 순으로 실행된다.
    - async
        - async 또한 스크립트를 '백그라운드'에서 다운로드한다. => HTML 파싱을 막지 않는다.
        - 단, async 스크립트가 실행중에는 HTML 파싱을 멈춘다.
        - DOMContentLoaded 이벤트와 비동기 스크립트는 서로를 기다리지 않는다.
        - 다른 스크립트와 비동기 스크립트는 서로를 기다리지 않는다.
        - 페이지에 async 스크립트가 여러 개 있는 경우, 그 실행 순서가 제각각이 됩니다. 실행은 다운로드가 끝난 스크립트 순으로 진행된다.

```htmlembedded
<html>
  <head>
    <script async src="https://google.com/analatics.js" type="text/javascript"></script>
  </head>
  <body>
    <div>...</div>
  </body>
</html>
```

### ✨ 리소스 용량 줄이기

용량이 큰 리소스도 웹 페이지 로딩 시간을 느리게 한다. 불필요한 데이터를 제거하고, 압축하여 사용하는 것이 좋다.

#### 중복 코드 제거하기

자주 사용되는 코드는 utils.js 파일로 정리해 사용한다.

#### 만능 유틸 사용 주의하기

loadsh와 같은 만능 유틸 라이브러리를 사용할 때, 일반적인 방식으로 가져와 사용하면 유틸함수 전체가 포함되어 자바스크립트 파일 용량이 커진다. 필요한 함수만 부분적으로 가져와서 용량이 늘어나지 않도록 한다.

```javascript
import array from 'lodash/array';
import object from 'lodash/fp/object';

array(...);
object(...);
```

#### HTML 마크업 최적화

불필요한 마크업을 사용하여 DOM 트리가 커지는 것을 막고, HTML 파일 용량이 늘어나지 않도록 한다.

* HTML은 **태그의 중첩을 최소화**하여 단순하게 구성한다.
* 공백, 주석 등을 제거하여 사용한다.

#### 간결한 CSS 선택자 사용

* **선택자는 최소화**하여 사용한다.
* 클래스 선택자를 사용하면 중복되는 스타일을 묶어서 처리한다.

#### 압축하여 사용하기

* HTML, JS, CSS 모두 압축하여 사용
* 불필요한 주석과 공백을 제거한 후, 난독화하여 사용한다.
* webpack과 같은 플러그인으로 이를 처리할 수 있다.

## 🖍 웹 페이지 렌더링 최적화

### 레이아웃 최적화

레이아웃은 DOM 요소들이 화면에 배치되는 것을 결정하는 계산 단계이다. 레이아웃은 일일이 계산하고, 요소간 관계를 모두 파악해야 하므로 시간이 오래 걸리는 과정이다. 레이아웃 최적화를 통해 레이아웃에 걸리는 시간을 최대한 단축해야 한다.

#### 1. 자바스크립트 실행 최적화

자바스크립트의 실행 시간이 길 겨웅, 한 프레임 처리가 오래 걸려 렌더링 성능이 떨어진다.

* **강제 동기 레이아웃 피하기**

    원래 레이아웃은 비동기로 일어나지만, 특정 상황에서 동기적으로 레이아웃이 발생한다. 이를 강제 동기 레이아웃이라고 하는데, 강제 동기 레이아웃은 JS의 실행 시간을 늘어나게 하므로 주의해야 한다.

    스타일을 변경한 다음 `offsetHeight`, `offfsetTop`과 같은 계산된 속성을 일을 때, 강제 동기 레이아웃이 수행된다. 이와 같은 코드를 최대한 사용하지 않도록 주의한다.

```javascript
    const tabBtn = document.getElementById('tab_btn');

tabBtn.style.fontSize = '24px';
console.log(testBlock.offsetTop); // offsetTop 호출 직전 브라우저 내부에서는 동기 레이아웃이 발생한다.
tabBtn.style.margin = '10px';
// 레이아웃
```

* **레이아웃 스레싱 피하기**

    한 프레임 내에서 강제 동기 레이아웃이 연속적으로 발생하면, 성능이 더욱 저하된다. for문이 반복될 때마다 레이아웃이 발생하는 것을 **레이아웃 스레싱**이라고 한다. 반복문 밖에서 box 엘리먼트의 너비를 읽어오면 레이아웃 스레싱을 피할 수 있다.

```javascript
function resizeAllParagraphs() {
  const box = document.getElementById('box');
  const paragraphs = document.querySelectorAll('.paragraph');

  for (let i = 0; i < paragraphs.length; i += 1) {
    paragraphs[i].style.width = box.offsetWidth + 'px';
  }
}
// 레이아웃 스래싱을 개선한 코드
function resizeAllParagraphs() {
  const box = document.getElementById('box');
  const paragraphs = document.querySelectorAll('.paragraph');
  const width = box.offsetWidth;

  for (let i = 0; i < paragraphs.length; i += 1) {
    paragraphs[i].style.width = width + 'px';
  }
}
```

* **가능한 하위 노드의 DOM을 조작하고 스타일을 변경**
  * DOM 트리의 상위 노드의 스타일을 변경하면 하위 노드에 모두 영향을 미친다.
  * 변경 범위를 최소화하여 레이아웃의 범위를 줄인다.
* **숨겨진 엘리먼트 수정**
  * 숨겨진 상태에서 엘리먼트를 변경하고 다시 보이도록 하여, 레이아웃 발생을 최대한 줄인다.
  * `visibility: hidden`은 보이지 않아 리페인트는 발생하지 않지만, 공간을 차지하기 때문에 레이아웃은 발생한다.
  * `display: none`으로 숨겨진 엘리먼트를 변경하면, 레이아웃과 리페인트가 발생하지 않아 성능에 유리하다.

#### 2. HTML, CSS 최적화

* **css규칙 수 최적화**
  * 사용하는 규칙이 적을수록 계산이 빠르므로 최소화한다.
  * 복잡한 선택자는 스타일 계산에 많은 시간이 걸리므로 피한다.
* **DOM 깊이 최소화**

    DOM트리가 깊을 수록, 하나의 노드에 자식 노드가 많을수록 DOM 트리는 커진다. 그만큼 DOM을 변경했을 때, 업데이트에 필요한 계산은 많아진다.

  * DOM이 작고 깊이가 얕을수록 계산이빠르다.
  * 불필요한 래퍼 엘리먼트는 제거한다.

#### 3. 애니메이션 최적화

**한 프레임 처리가 16ms**내로 완료되어야 렌더링시 끊기는 현상없이 자연스러운 렌더링을 만들어낼 수 있다. 애니메이션을 구현할 때, 네이티브 자바스크립트 API를 사용하는 것보다, CSS 사용을 권장한다.

* **requestAnimationFrame\(\) 사용** 
  * requestAnimationFrame API를 사용하면 브라우저의 프레임 속도\(보통 60fps\)에 맞추어 애니메이션을 실행할 수 있도록 해준다.
  * 프레임을 시작할 때 호출되므로, 일정한 간격으로 애니메이션을 수행할 수 있는 장점이 있다.
  * 현재 페이지가 보이지 않을 때는 콜백함수가 호출되지 않으므로 불필요한 동작을 하지 않는다.
* **CSS 애니메이션 사용**

    자바스크립트를 사용한 애니메이션은 성능이 나쁠 수 있다. CSS3 애니메이션을 사용하면, 브라우저가 애니메이션을 처리하는데 최적화되어 있어서 부드러운 애니메이션을 구현할 수 있다. 

  * **position:absolute** 처리

      애니메이션이 주변 영역에 영향을 주지 않도록 주의해야 한다. position을 absolute나 fixed로 설정하여 주변 레이아웃에 영향을 주지 않는다.

  * **transform 사용**

      position, width, height와 같이 기하적 변화를 유발하는 속성을 변경하면 레이아웃이 발생한다. **transform을 사용하는 엘리먼트는 레이어로 분리**되어, 영향받는 엘리먼트가 제한되므로 레이아웃과 페인트를 줄일수 있다. 또한 합성만 발생시키기 때문에 애니메이션 사용시 렌더링 속도가 향상될 수 있다. 하드웨어가 지원될 경우, GPU를 사용할 수 있어 성능이 빠르다.

#### 4. DocumentFragment 이용하기

자바스크립트의 DOM 객체는 연산을 수행할 때마다 DOM tree라는 자료구조에 접근해야 하기 때문에 자바스크립트의 성능을 저하시키는 주된 요인 중 하나이다. 따라서, 자바스크립트의 성능을 최적화하기 위해서는 DOM 객체 접근을 최소화하도록 코드를 작성해야 한다.

DocumentFragment는 메인 DOM 트리의 일부가 되지 않는다. 따라서, DocumentFragment를 변경하여도 문서에는 영향이 없으며, 리플로우도 일으키지 않는다.

DocumentFragment는 주로, createDocumentFragment\(\)로 DocumentFragment를 생성하고, 그 안에서 DOM 하위 트리를 조립한 다음, DocumentFragment를 DOM 트리에 추가하는 것이다. 이렇게 하면 DocumentFragment의 노드들이 DOM으로 이동되고 빈 DocumentFragment만 남게 됩니다. 이를 통해, 여러번 리플로우와 렌더링이 일어나지 않고, 단 한 번만 리플로우와 렌더링이 일어나게 된다.

* 최적화 예시

```javascript
function addElements() {
    var target = document.getElementById('list');

    for (var i = 0; i < 100; i++) {
        var div = document.createElement('div');

        div.innerText = 'div';
        target.appendChild(div);
    }
}
```

위의 코드는 DOM 객체에 100번 접근하게 된다.

자바스크립트에서 제공하는 **DocumentFragment** 객체를 이용하면 DOM 객체 접근을 최소화할 수 있다.

```javascript
function addElements() {
    var target = document.getElementById('list');
    var docFrag = document.createDocumentFragment();

    for (var i = 0; i < 100; i++) {
        var div = document.createElement('div');

        div.innerText = 'div';
        docFrag.appendChild(div);
    }
    target.appendChild(docFrag);
}
```

## 출처

* [성능 최적화 - toast](https://ui.toast.com/fe-guide/ko_PERFORMANCE#%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83%EA%B3%BC-%EB%A6%AC%ED%8E%98%EC%9D%B8%ED%8A%B8)
* [DocumentFragment를 이용한 JavaScript 성능 최적화](https://untitledtblog.tistory.com/44)

