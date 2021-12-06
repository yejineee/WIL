# 웹 렌더링 - SSR, CSR, 정적 렌더링

## 📘 용어

📍 **렌더링**

* **SSR** : 서버사이드 렌더링 - 서버의 HTML로 렌더링한다.
* **CSR** : 클라이언트 사이드 렌더링 - 브라우저에서 애플리케이션을 렌더링한다. 일반적으로 DOM을 사용한다.
* **Refydration** : 클라이언트가 버서에서 렌더링한 HTML의 DOM 트리와 데이터를 재사용하도록 자바스크립트 뷰를 부팅한다.
* **Prerendering** : 빌드 타임에 클라이언트측 애플리케이션을 실행하여 초기 상태를 정적 HTML로 캡쳐한다.

📍 **성능**

* **TTFB** : Time to First Byte - 링크 클릭후 처음으로 들어오는 컨텐츠 비트 사이의 시간을 나타낸다.
* **FP** : First Paint - 픽셀이 처음으로 사용자에게 표시되는 시점
* **FCP** : First Contentful Paint - 요청 컨텐츠가 표시되는 시점
* **TTI** : Time to Interactive - 페이지가 상호작용하게 될 때까지의 시간 

## 📘 SSR - 서버사이드 렌더링이란?

![](https://i.imgur.com/59waJ2R.png)

![](https://i.imgur.com/NySwW2P.png)

### 📍 SSR이란?

**SSR은 서버에서 full HTML을 만들어내는 방식이다**. SSR을 사용하면 **모든 데이터가 매핑된 서비스 페이지를 클라이언트에게 바로 보여줄 수 있다**. 서버사이드 렌더링으로 인해, data fetching과 템플릿 작성을 위한 추가적인 round-trip이 발생하지 않는다.

### 👍 SSR의 장점

* **Time-to-Content가 빠르다.**

  일반적으로 빠른 FP\(First Paint\)와 FCP\(First Contentful Paint\)를 생성한다.

* **빠른 TTI\(Time to Interactive\)** 가 가능하다.

  서버에서 페이지 로직 및 렌더링을 실행하여, 많은 자바스크립트를 클라이언트에 보내지 않아도 된다. 따라서 빠른 TTI가 가능하다.

* 더 **나은 SEO**가 가능하다.

  검색 엔진 크롤러가 바로 렌더링이 완료된 페이지에 접근할 수 있다.

* 브라우저에 텍스트와 링크를 보내는 것에 의미가 있다.

  이를 통해, 스트리밍 문서 구문 분석과 같은 브라우저 최적화가 가능하다.

### 👎 SSR의 단점

* SSR은 서버에서 페이지를 생성하는데 시간이 걸린다는 단점 때문에 **TTFB\(Time to First Byte\)는 느려**질 수 있다.
* 매번 새로운 페이지를 요청할 때마다 **새로고침**이 발생한다.
* **서버에 많은 부하**를 줄 수 있다.

  전체 앱을 렌더링한다는 것이 단순히 정적 파일을 서빙하는 것보다 더 많은 CPU를 요구하게 된다.

  간단한 데이터 수정도 서버를 거쳐야하며, 매번 서버에 요청을 하는 것은 서버 과부하의 원인이 된다.

* **SSR을 위한 빌드와 배포 세팅이 필요**하다.

  정적 SPA는 어떠한 정적 파일 서버에서라도 배포될 수 있지만, SSR은 Node.js 서버가 동작할 수 있는 환경을 구축해야 한다.

### 🤔 SSR 선택 시 고려할 점

**컨텐츠가 처음으로 페인트되는 시간 \(time-to-content\)가 얼마만큼 중요한지**를 따져봐야 한다.

만약 초기 로드 시간이 그다지 중요한 사항이 아니면, SSR은 과도할 수 있다. time-to-content가 절대적으로 중요한 경우에 SSR을 사용하여 좋은 초기 로드 성능을 가져갈 수 있다.

### 📍 SSR 도구

* 리액트 : renderToString\(\)이나 Next.js
* 뷰 : [SSR 가이드](https://ssr.vuejs.org/)나 Nuxt.js
* 앵귤러 : 앵귤러 유니버설

## 📘 Static Rendering - 정적 렌더링이란?

![](https://i.imgur.com/oOrzljW.png)

### 📍 정적 렌더링이란?

정적 렌더링은 **빌드 타임에 발생**한다. 일반적으로 **미리 각 URL에 대해 별도의 HTML 파일을 생성**하는 것을 말한다.

### 👍 정적 렌더링의 장점

* **빠른 First Paint, First Contentful Paint 및 Time To Interactive를 제공**한다.
* **SSR보다 빠른 Time To First Byte가 가능**하다.

  SSR은 각 URL에 대해 필요할 때 HTML을 생성한다. 그러나 정적 렌더링은 미리 각 URL에 대한 HTML 파일을 생성하여 제공하므로, 빠른 Time To First Byte가 가능하다.

* HTML 응답을 미리 생성하면, 정적 렌더링을 여러 CDN에 배포하여 **edge-caching을 활용**할 수 있다.

### 👎 정적 렌더링의 단점

* **가능한 모든 URL에 대해 개별 HTML을 생성**해야 한다.

  이러한 URL을 미리 예측할 수 없거나, 독창적인 페이지가 많아서 사이트의 URL을 예측할 수 없다면, 실행이 어렵거나 불가능할 수 있다.

### 📍 정적 렌더링 도구

* Gatsby : 개발자가 프로그램을 빌드 단계로 생성하는 대신, 동적으로 렌더링하는 것처럼 느끼도록 설계되어있다. \(CSR with Prerendering\)
* Jekyl, Metalsmith : 정적인 성질을 이용하여 좀 더 템플릿 중심의 접근 방식을 제공한다.

### 🤔 정적 렌더링과 사전 렌더링\(Prerendering\)의 차이

정적 렌더링과 사전 렌더링의 차이를 이해해야 한다. `정적 렌더링`은 많은 클라이언트측의 JS를 실행하지 않더라도 인터렉티브하다. 그러나, `사전 렌더링`은 페이지가 인터렉티브하기 위해서는 부팅 과정이 필요하다.

JS를 비활성화하고 웹 페이지를 불러온다면, 정적 페이지는 JS가 활성화되지 않아도 계속 존재한다. 그러나, 사전 렌더링은 페이지 대부분이 동작하지 않는다.

## 📘 CSR - 클라이언트 사이드 렌더링이란?

![](https://i.imgur.com/n1877qB.png)

![](https://i.imgur.com/qG66EqO.png)

### 📍 CSR이란?

CSR은 **자바스크립트를 사용하여 브라우저에서 직접 렌더링**하는 것을 의미한다. 모든 로직, data fetching, templating, 라우팅은 클라이언트에서 처리된다.

최초 페이지 로딩 이후, JS를 사용하여 동적으로 데이터를 변경하며 화면을 바꾼다. 서버는 JSON만 보내주는 역할을 하고, 클라이언트는 그 데이터를 갖고 HTML을 렌더링한다.

### 👎 CSR의 단점

* 앱이 커짐에 따라, JS의 양이 증가한다.

  프로세싱 파워를 놓고 경쟁하는 JS 라이브러리 폴리필 서드파티 코드를 추가하면 페이지의 내용을 렌더링하기 전에 처리해야 한다. `코드 스플리팅`과 JS의 `lazy loading` 도입을 고려해보는 것이 좋다.

* 초기 렌더링 속도가 느리다.

  자바스크립트를 다운받아와서, 파싱하고, data를 가져온 후에 컨텐츠를 보여줄 수 있다.

* SEO에 불리하다.

  웹 크롤러, 봇들이 JS를 실행시키지 못하고 HTML에서만 컨텐츠를 수집하므로, CSR 방식으로 개발된 페이지를 빈 페이지로 인식하게 된다. 따라서 CSR에서의 SEO가 불리하다.

### 📍 CSR 성능 개선

* `HTTP/2 server push` 나 `<link rel=preload>`로 중요한 스크립트와 데이터를 더 빠르게 전달할 수 있다. 그러면 파서가 더 빠르게 작동할 것이다.

## 📘 정리

![](https://i.imgur.com/U0u3l4C.png)

## 🔗 출처

* [웹 렌더링 - google developers](https://developers.google.com/web/updates/2019/02/rendering-on-the-web?hl=ko)
* [어서 와, SSR은 처음이지? - 도입 편 - naver D2](https://d2.naver.com/helloworld/7804182)
* [SPA, CSR과 SSR, SEO](https://velog.io/@ksh4820/SPA-CSR%EA%B3%BC-SSR-SEO)
* [서버 사이드 렌더링 그리고 클라이언트 사이드 렌더링 - jbee](https://asfirstalways.tistory.com/244)
* [검색 엔진 최적화 - 위키백과](https://ko.wikipedia.org/wiki/%EA%B2%80%EC%83%89_%EC%97%94%EC%A7%84_%EC%B5%9C%EC%A0%81%ED%99%94)
* [\[VUE\] SPA 웹 프론트앤드 개발을 위한 정리 - JY Lee](https://ko-dev-jp.com/2020/08/19/spa-web-vue-kr/)
* [Vue SSR Guide](https://ssr.vuejs.org/#ssr-vs-prerendering)

