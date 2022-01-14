
# Graphics System
- CSS Rendering 1회차
---


- Fixed Number
    - 고정된 숫자로 만들어진 그래픽스 시스템은 환경에 맞출 수 없다는 문제가 있다.
    - 스마트폰의 크기, 가로와 세로모드 등 여러 환경이 있는데, 고정된 숫자로는 범용적인 그래픽스 시스템을 만들 수 없다.
- Abstract  Calculator
    - %, Left, Inline, Float
    - 환경을 인식해서 숫자로 바뀌는 것.
    - 공식을 이용해서 숫자를 계산하게 하는 체계
    - 즉, ‘함수’
- Component
    - 추상화된 그래픽스 시스템을 이어받는 공통적인 시스템
    - 예) HTML의 tag
- Framework
    - 컴포넌트들을 일정한 규칙과 사용방법을 지키는 형태로 구현한 것
    - 예) HTML 체계 전체
- 그래픽스 시스템은 Fixed Number에서 Abstract Calculator로 진화
- 그래픽스 시스템 : 점 찍는 방법

# Rendering System

- 그림을 표현하기 위한 정보를 그림으로 표현하는 것.
- Rendering System의 구성
    1. Geometry Calculate
        - 어떻게 영역이 나눠져있는지 계산
        - 브라우저 - Reflow
    2. Fragment Fill
        - 영역을 색칠하기
        - 브라우저 - Repaint

# CSS Specification

- CSS를 배운다 : CSS 속성과 값이 구체적으로 발현될 때는 어떠한 방식으로 계산되어서 표현되는지 이해하는데 있다.
- CSS Level 1
    - w3c에 css라는 것을 넣도록 제안
- CSS Level 2 + Module
- CSS Level 2.1
    - 어떠한 그래픽스 시스템은 레벨업이 안되는 것이 있고, 어떤 것은 계속 발전하므로 레벨업이 되는 사양들이 있다.
    - include level 3 module
        - syntax3, color3, background 3, fonts 3, text 3... 등등
- Module Level
    - transforms 1
    - compositing 1
    - effects 1
    - grid 1
    - flexbox 1
    - masking 1
    

→ 현재는 CSS는 각 사양마다 레벨이 다름.

[All CSS specifications](https://www.w3.org/Style/CSS/specs.en.html)

# Other Specification

W3C community and business groups

- WICG : Web Platform Incubator Community Group
    - 구글이 이끔.
    - 최신 스펙이 제일 많이 생기는 곳. WICG에서 크롬에 구현 먼저 하고, W3C에 draft 던져놓음.
- RICG : Responsive Issues Community Group