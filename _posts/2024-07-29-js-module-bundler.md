---
layout: post  
title: `JavaScript 모듈 번들러`
author: `윤예진`
categories: `프론트엔드 기술블로그`
banner:
  image: `https://media.licdn.com/dms/image/D4E12AQHxWmqrVxZ27g/article-cover_image-shrink_720_1280/0/1681202907635?e=1727913600&v=beta&t=geXsexnmlAugomDPuukCVVVd051jy_G1IvCOW52zL1o`
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`JavaScript`, `module_bundler`, `webpack`, `vite`]
---

## JavaScript 모듈 번들러가 뭐라고 생각하세요?

🎶JavaScript 모듈 번들러란 말이죵…🎶

Boot-up 과정 전까지 Webpack은 그저 저에게 ‘CRA로 생성되는 무언가’로 인식되고 있었습니다. Boot-up 과정 중 Webpack 은 `modern JavaScript 앱을 위한 정적 모듈 번들러` 라는 것을 알게 되었습니다.

이 때 모듈 번들러는 과연 무엇일까요?

## 모듈

개발하는 애플리케이션의 크기가 커지면 유지 보수를 위해 파일을 여러 개로 분리해야 합니다. 이때 분리된 파일 각각을 '모듈(module)'이라고 부릅니다.

과거에는 이런 모듈 스크립트를 직접 HTML 파일에 삽입하는 방식으로 개발을 진행했습니다. 이런 식의 개발은 모듈 간의 변수 충돌이 빈번했을 뿐만 아니라 모듈 로드 순서도 신경 써야 했습니다.

## 모듈 로더

이런 불편함을 해결하고자, CommonJS나 AMD와 같은 모듈 로더 시스템이 등장하게 되었습니다.

CommonJS는 JavaScript를 서버사이드 애플리케이션에서도 지원하는 모듈 시스템입니다. `require()` 메서드를 통해 모듈을 로드하고, `exports` 객체를 통해 모듈을 내보낼 수 있습니다. CommonJS의 주요 장점은 모듈을 파일 단위로 구분하여 관리할 수 있다는 점입니다. 이를 통해 코드의 재사용성과 유지보수성을 높일 수 있습니다. 그러나 CommonJS는 동기적으로 모듈을 로드하기 때문에 브라우저 환경에서는 적합하지 않다는 단점이 있습니다.

AMD(Asynchronous Module Definition)는 이름에서 알 수 있듯이 비동기적으로 모듈을 로드하고 관리할 수 있는 모듈 시스템입니다. AMD는 주로 브라우저 환경에서 사용되며, 이를 통해 여러 모듈을 비동기적으로 로드할 수 있어 성능을 최적화할 수 있습니다. AMD는 `define`이라는 구문을 사용하여 모듈을 정의하는데, 이 구문은 모듈의 이름, 의존성, 그리고 모듈의 정의를 포함합니다. 이로 인해 복잡한 애플리케이션에서는 `define` 구문이 장황해지고, 전달해야 할 인자가 많아 가독성이 떨어질 수 있습니다.

이후 JavaScript 언어 자체에서 모듈 시스템을 지원하기 위해 ECMAScript 6에서 ES Module이라는 표준 모듈 시스템이 도입되었습니다. 이 방식은 우리가 개발할 때 흔히 사용하는 import/export 구문을 사용합니다. ES Module은 브라우저에서 지원되며, 덕분에 `<script type="module">`을 사용하여 모듈을 불러올 수 있습니다.

## 모듈 번들러

-   **모듈 단위로 개발하여 유지 보수성을 높일 수 있습니다**
    번들러를 사용하지 않을 때는 각각의 JS 파일을 사용하기 위해 파일의 종속성을 고려하여 많은 스크립트 태그를 추가해야 했는데, 번들러는 파일끼리의 종속성을 알아서 확인하여 묶어줍니다.
-   **한 번에 많은 요청을 하지 않아도 됩니다.**
    모듈 번들러는 JS 모듈을 브라우저에서 실행할 수 있는 단일 JS 파일로 번들링 해줍니다. 따라서 한 번의 네트워크 요청으로 우리는 웹 페이지를 로드할 수 있게 됩니다.
-   **Tree Shaking**
    필요 없는 코드를 제거하고 번들 파일의 크기, 번들링 시간을 줄여줍니다.
-   **HMR (Hot Module Replacement)**
    코드가 변경되면 감지하고 브라우저에 최신 코드를 반영하여 자동으로 모듈을 교체합니다. 개발자는 새로고침을 하지 않아도 반영된 것을 빠르게 확인할 수 있으며 변경 사항만 업데이트하기 때문에 개발 속도가 빨라집니다.
-   **Code Splitting**
    JS를 청크로 분할하고, 청크가 필요한 경로에만 제공하여 성능을 향상시킵니다.
    모듈 번들러가 의존성 있는 모듈끼리 번들링해서 클라이언트에게 보내 HTTP request 통신 횟수를 최소화 하여 유저의 로딩 시간을 줄였습니다. 그런데 이렇게 되면 하나의 번들 파일의 크기가 너무 커지지 않을까요? 큰 번들을 브라우저에서 파싱하고 컴파일 해야 하니 로딩 시간이 더 길어지지 않을까요? 맞습니다!
    그래서 이러한 문제를 인지하여 코드 스플리팅이라는 개념이 생겼습니다. 하나의 큰 번들을 여러 개의 번들로 쪼개고 필요한 경로에만 제공하여 최적화를 시킵니다.
-   **Transformations**
    트랜스파일러를 사용하여 ES6 버전 이상의 스크립트를 사용 가능하게 해줍니다.

## 모듈 번들러 프로세스

번들링 과정은 일반적으로 다음과 같이 이루어집니다:

1. **엔트리 포인트(Entry Point) 정의**:
    - 애플리케이션의 시작 파일(예: `index.js`)을 엔트리 포인트로 설정합니다. 번들러는 이 파일을 시작점으로 삼아 모든 의존성을 탐색합니다.
2. **의존성 그래프(Dependency Graph) 생성**:
    - 엔트리 포인트에서부터 시작하여 모든 의존성 모듈을 재귀적으로 탐색하여 의존성 그래프를 만듭니다. 이 그래프는 모듈 간의 관계를 나타냅니다.
3. **모듈 변환(Transformation)**:
    - 각 모듈을 ES5로 트랜스파일링하거나, TypeScript와 같은 다른 언어로 작성된 코드를 JavaScript로 변환합니다. 이 단계에서는 Babel과 같은 도구가 사용됩니다.
4. **번들링(Bundling)**:
    - 모든 모듈을 하나의 번들 파일로 결합합니다. 여기서 Tree Shaking과 같은 최적화 기술이 적용되어 사용되지 않는 코드가 제거됩니다.
5. **출력(Output)**:
    - 최종 번들 파일을 생성하여 지정된 디렉터리에 저장합니다.

## Webpack

Webpack은 가장 널리 사용되는 JavaScript 모듈 번들러 중 하나입니다. Webpack의 주요 특징은 다음과 같습니다:

-   **플러그인 시스템**:
    -   Webpack은 다양한 플러그인을 통해 기능을 확장할 수 있습니다. 이를 통해 CSS, 이미지, 폰트 등 다양한 파일 타입을 처리할 수 있습니다.
-   **로더(Loaders)**:
    -   로더를 사용하여 JavaScript 외의 파일을 모듈화할 수 있습니다. 예를 들어, `style-loader`와 `css-loader`를 사용하여 CSS 파일을 로드할 수 있습니다.
-   **정적 파일 처리**:
    -   Webpack은 이미지, 폰트 등 정적 파일을 효율적으로 처리하고 번들에 포함시킬 수 있습니다.
-   **HMR (Hot Module Replacement)**:
    -   코드 변경 시 브라우저를 새로 고침하지 않고도 변경 사항을 즉시 반영할 수 있는 기능을 제공합니다. 이를 통해 개발 생산성이 크게 향상됩니다.

## Vite

Vite는 차세대 프론트엔드 개발 도구로, 빠른 빌드 속도와 간편한 설정이 특징입니다. Vite의 주요 특징은 다음과 같습니다:

-   **빠른 개발 서버**:
    -   Vite는 ES Module을 기반으로 한 빠른 개발 서버를 제공합니다. 이를 통해 즉각적인 HMR이 가능하며, 초기 로딩 속도가 매우 빠릅니다.
-   **최소한의 설정**:
    -   Vite는 기본 설정이 매우 간단하며, 별도의 설정 없이도 빠르게 프로젝트를 시작할 수 있습니다. 필요한 경우 사용자 정의 설정도 쉽게 추가할 수 있습니다.
-   **플러그인 에코시스템**:
    -   Vite는 다양한 플러그인을 지원하며, 이를 통해 프로젝트의 요구에 맞게 기능을 확장할 수 있습니다.
-   **최적화된 빌드**:
    -   Vite는 Rollup을 기반으로 한 최적화된 빌드 프로세스를 제공합니다. 이를 통해 작은 번들 크기와 빠른 로딩 속도를 유지할 수 있습니다.

---

### 참고 자료

-   [Why do we need a module bundler?](https://medium.com/@rajatgms/why-do-we-need-a-module-bundler-c5ff221523f5)
-   [JavaScript Modules](https://ko.javascript.info/modules-intro)
-   Webpack Official Documentation
-   [Vite Official Documentation](https://vitejs.dev/)
-   [Bundler: JavaScript 번들러, 그리고 Webpack, Parcel, Rollup, Vite](https://velog.io/@wynter_j/Bundler-JavaScript-%EB%B2%88%EB%93%A4%EB%9F%AC-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Webpack-Parcel-Rollup-Vite)
