---
layout: post  
title: "렌더링 최적화 자체 제공 시스템, React Virtual DOM"
author: "양하연"
categories: "기술블로그"
banner:
  image: 썸네일로 넣고 싶은 이미지 링크
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`React`, `Virtual DOM`, `렌더링`, `최적화`]
---

# 렌더링 최적화를 자체적으로 제공해주는 React 내부 시스템, React Virtual DOM

## 0. 들어가며 - React와 렌더링 최적화

프론트엔드 개발을 하면서 최적화, 그 중에서도 렌더링 최적화는 빠질 수 없는 부분이죠. 우리는 개발을 하면서 개발자도구의 lighthouse, profiler 등을 이용해서 렌더링 변경사항을 확인하고 useMemo, useCallback 최적화에 용이한 리액트 훅을 신경써서 사용한다든가, api 응답을 caching하는 tanstack query를 이용한다든가, 하는 여러가지 방법을 이용해서 리액트 최적화를 실현하 기 위해 노력합니다. 이렇게 개발자가 직접 프로그래밍을 하면서 렌더링 최적화를 신경써야 하는 부분이 대다수이지만 리액트가 내부적으로 렌더링 최적화를 고려하여 만들어진 시스템적인 부분을 인지하고 리액트를 사용하는 것 또한 중요한 부분입니다. 따라서 리액트의 자체적인 렌더링 최적화 시스템인 React Virtual DOM에 대해서 알아보고자 합니다.

## 1. React Virtual DOM 인지 필요성

본 글에서는 리액트가 자체적으로 렌더링에 신경써주는 시스템인 React Virtual DOM에 대해서 알아보고자합니다. 리액트 Virtual DOM은 말 그대로 가상으로 존재하는 DOM입니다. 그런데 과연 DOM이 무엇이길래 리액트에서 Virtual DOM을 만들어서 자체적으로 제공하는 것일까요? DOM이 무엇인지 알아보기 위해서는 브라우저의 전반적인 렌더링 과정에 대해서도 같이 알아볼 필요가 있습니다.

## 2. 브라우저의 전반적인 동작 원리

DOM은 Document Object Model의 준말입니다. 여기서 Document란 무엇이냐고 하면 브라우저에 보여지는 html,css,js코드로 만들어지는 화면을 만드는 코드, 즉 우리가 작성한 프론트엔드 코드라고 이해하면 됩니다. 아래 사진처럼 브라우저는 이 문서를 객체 형식으로 변환하여 브라우저에 보여주는 역할을 하는데, 이 과정은 Parse, Layout, Paint, Composite이라는 크게 4가지의 단계를 가집니다.

![alt text ](image-1.png)
사진 출처 : https://medium.com/@jihyerish/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EB%A0%8C%EB%8D%94%EB%A7%81-%EB%8F%99%EC%9E%91-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-4245c2f0a606

Parsing단계에서는 HTML, CSS 코드를 파싱해서 각 HTML태그가 무엇을 의미하는지 css 태그가 무엇을 의미하는지 태그 단위로 분해하는 행위가 이루어집니다. HTML은 HTML DOM을 형성하고, CSS는 CSSOM을 형성하여 Render을 준비하는 Render Tree를 형성하게 됩니다. 이렇게 만들어진 Render Tree는 브라우저 화면에 어디에 어떤 태그가 어떤 사이즈로 어떤 스타일로 그려질 것인지 그 위치를 정하는 Layout Tree를 형성합니다. Render Tree에서 Layout Tree로 변환하는 이 과정을 reflow, 다른 말로 Render Phase라고 하는데 파싱된 태그를 수직 구조를 가지는 Tree를 만들기 때문에 상당한 시간을 소비하는 과정이라고 할 수 있습니다.
이렇게 레이아웃을 잡는 과정을 진행하고 마지막으로는 페인트 과정을 진행합니다. Layout Tree로 잡은 그 구조를 화면에 페인트해서 그려주는 작업이 진행됩니다. 이 과정을 Repaint, 다른 말로 Commit Phase라고 합니다.

사진에서 강조해서 표시했다시피, Tree를 형성하고 그것을 적절한 브라우저에 그리는 작업이 이루어지는 Render Phase와 Commit Phase는 상당한 시간 리소스를 소비하게 됩니다. 예를 들어, 우리 개발자들은 단순하게 padding을 0pt에서 10pt로 변경하는 아주 간단한 코드만으로, 브라우저는 이 작은 변경사항을 반영한 DOM Tree를 처음부터 다시 형성하고 그걸 다시 그려주는 과정으로 동작합니다. 즉, 아주 작은 속성을 변경하는 것이라도, 이것을 렌더링해서 화면에 보여주는 과정이 오래 걸리게 되는 것입니다.

그래서 우리는 DOM을 직접 수정하는 과정을 최대한 적게 진행해야만 렌더링 최적화를 이룩할 수 있습니다. 그렇다면 DOM을 최대한 변경하지 않기 위해 HTML, CSS 변경하는 코드 자체를 변경하지 말아야할까요? 속성을 변경하지 않는 것보다 변경이 많이 이루어지더라도 그 변경사항에 대한 DOM Tree를 만드는 과정 자체를 줄일 수 있으면 되겠죠. 이것을 바로 React DOM 이 자체적으로 해주는 것입니다.

## 3. 리액트의 Virtual DOM에서 제공하는 최적화

![alt text](image-2.png)
사진 출처 : https://www.youtube.com/watch?v=N7qlk_GQRJU

React의 Virtual DOM은 개발자가 굳이 신경 쓰지 않아도 자동으로 DOM 변경과정을 최적화해주는 시스템입니다. 리액트는 내부적으로 모든 업데이트를 모아서 최소한의 횟수로 DOM을 수정할 수 있도록 하며, 이를 통해 브라우저 상에서의 DOM 작업을 추상화합니다. 리액트는 두 단계의 렌더링 프로세스를 사용합니다. 첫 번째는 ‘렌더 페이즈’로, 컴포넌트를 호출해 필요한 업데이트를 계산하는 단계이고, 두 번째는 ‘커밋 페이즈’로, 렌더 페이즈에서 계산된 변경 사항을 실제 DOM에 반영하는 단계입니다.

렌더 페이즈에서는 리액트 컴포넌트를 호출하여 ‘리액트 엘리먼트’라는 객체를 반환받습니다. 이 객체는 UI에 대한 모든 정보를 담고 있으며, 리액트는 이를 모아 Virtual DOM이라는 가상의 트리를 구성합니다. 이 Virtual DOM은 자바스크립트 객체 형태로 존재하며, 실제 DOM이 아닌 값이므로 수정이 간편합니다.

커밋 페이즈에서는 Virtual DOM을 실제 DOM에 반영하게 되는데, 이때 리액트는 이전 Virtual DOM과 새로 생성된 Virtual DOM을 비교해 변경된 부분만을 찾아내어 반영합니다. 이를 통해 최소한의 DOM 업데이트만 진행하므로, 대부분의 상황에서 충분히 빠른 속도로 화면을 업데이트할 수 있습니다.

### 그러나 항상 최고는 아니다

그러나 리액트의 이러한 방식이 항상 최고의 성능을 보장하는 것은 아닙니다. Virtual DOM을 생성하고 비교하는 과정도 연산 비용이 발생하기 때문에, 더 빠른 기술들이 등장하기도 했습니다. 모든 상황에서 최고로 빠르게 동작하지는 않지만, 평균적으로 Virtual DOM Tree를 생성하고 변경사항을 반영하는 시간이 작기 때문에, 현재 상황으로서는 가장 최적의 방법 중 하나로 여겨지고, 그만큼 React가 프론트엔드 분야에서 많이 쓰이는 기술이라고 생각합니다. 프론트엔드 개발자로서 우리는 이러한 Virtual DOM이 어떻게 렌더링 최적화를 실현하고 있는 지 그 원리를 알고, 그 원리 기반으로 효율적인 코드 작성을 하도록 노력해나갈 수 있겠습니다.

## 츨처

https://medium.com/@jihyerish/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EB%A0%8C%EB%8D%94%EB%A7%81-%EB%8F%99%EC%9E%91-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-4245c2f0a606

https://www.youtube.com/watch?v=N7qlk_GQRJU
