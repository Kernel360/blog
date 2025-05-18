---
layout: post  
title: 클라이언트 환경에서 파일을 문자열로 변환할 수 없는 이유
author: 윤예진
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [JavaScript, module_bundler, webpack, vite]
---

[오픈소스 프로젝트](https://shadcn-dialog-snippet.vercel.app/)의 첫 배포가 이번 주에 완료되었습니다! 🎉 이 프로젝트는 `shadcn/ui` 라이브러리의 `dialog` 컴포넌트를 활용하여 개발자들이 자주 마주치는 다양한 `dialog` 상황에 대한 코드 스니펫을 모아둔 모음집입니다.

첫 배포를 위해 우리 팀이 구현한 기능은 크게 세 가지입니다.

 1. 사용자들이 실제 구현된 모습을 직접 볼 수 있는 dialog 미리보기 화면 구현 

2. 원하는 코드를 바로 프로젝트에 적용할 수 있는 코드 복사 기능 

3. 코드의 가독성을 높이기 위한 코드 에디터 스타일 구현 

이 중에서 저는 두 번째와 세 번째 기능의 개발을 맡았습니다. 코드 복사 기능은 웹 브라우저의 표준 API인 `Clipboard API`의 `writeText` 메서드를 활용했고, 코드 에디터와 유사한 시각적 효과를 위해 `shiki` 라이브러리를 적용했습니다. 

이 두 가지 주요 기능을 구현하면서 공통적으로 필요했던 리소스는 상황별로 만들어둔 `dialog` 컴포넌트의 문자열 버전이였습니다. 

원했던 상황은 다음과 같습니다. 

```
1. 단순히 화면에 렌더링 되는 부분만이 아니라, `import` 구문부터 `export` 구문, 컴포넌트 태그들까지 IDE에서 개발자가 보는 형태 그대로 문자열 값으로 바꾸기
2. 바꾼 문자열을 변수에 저장해 `export` 하기 
3. 해당 변수를 컴포넌트에 props로 넘겨줌으로써 중복 코드 줄이기
```

위와 같은 상황을 만들기 위해 파일 자체를 직접 문자열로 변환하고 싶었지만, 이 작업을 수행할 수 없었습니다. 저는 이 작업이 왜 불가능한지 이해가 되지 않았습니다. 프로젝트 내부 파일을 문자열로 바꾸는 일인데 말이죠.

---

### 클라이언트 환경에서 프로젝트 내부 파일을 문자열로 변환하는 작업은 왜 불가능할까?

브라우저와 같은 클라이언트 환경에서는 보안상의 이유로 로컬 파일 시스템에 직접 접근하거나 소스 코드 파일을 읽어 문자열로 변환하는 작업이 제한됩니다. 

이러한 제한의 핵심에는 동일 출처 정책(`Same-Origin Policy`)이 있습니다. 이 정책은 한 웹 페이지에서 실행되는 스크립트가 다른 출처(`Origin`)의 리소스에 접근하는 것을 막습니다. 이는 악의적인 스크립트가 사용자의 중요한 정보를 탈취하거나 조작하는 것을 방지하기 위한 중요한 안전장치입니다. 

프로젝트 내부 파일을 읽는 것 역시 이 동일 출처 정책의 영향을 받습니다. 브라우저에서 로컬 파일은 일반적으로 `file://` 프로토콜을 통해 접근되는데, 이는 웹 리소스와는 다른 방식으로 취급됩니다. `file://` 프로토콜은 동일 출처 정책 적용 시 하나의 출처로 간주되지만, 보안 위험이 크기 때문에 더욱 엄격하게 관리됩니다. 

따라서 개발자가 작성한 파일을 읽으려면 API나 서버를 통한 별도의 승인된 경로가 필요합니다. 이러한 보안 정책들은 사용자의 데이터와 시스템을 보호하는 데 중요한 역할을 합니다. 

보안 정책으로 인해 클라이언트 환경에서 파일 시스템에 직접 접근할 수 없는 이유도 알게 되었습니다. 그런데, 여전히 한가지 의문이 남아있습니다. 

react 컴포넌트를 만들 때 import 구문을 사용해서 다른 컴포넌트 파일을 사용할 수 있습니다. 브라우저에서 파일 시스템에 접근하지 못한다면, **그럼 왜 `react` 컴포넌트 내부의 `import`는 가능할까요?** 

### import 가 동작하는 이유

React 컴포넌트 내부의 `import`가 가능한 이유는 이 작업이 브라우저에서 실행되지 않기 때문입니다. `import`는 **빌드 시점**에서 처리됩니다. 빌드 도구가 개발자의 파일 시스템에 접근해서 `import`된 파일들을 분석하고, 이를 하나의 번들 파일로 통합합니다. 이러한 과정은 **서버나 로컬 개발 환경**에서 이루어지기 때문에 파일 시스템 접근에 제한이 없습니다.

### 프로젝트에 어떻게 적용할 수 있을까?

빌드 시점에 빌드 도구가 파일 시스템에 접근 할 수 있다는 것을 알았으니, 파일을 빌드 시점에 문자열로 수정하면 제가 원하는 상황이 만들어 질 수 있을 것 같습니다. 

우선 파일을 읽어 문자열로 변환하는 함수를 만들어 줍니다.

```tsx
import path from "node:path";
import fs from "fs-extra";

const convertTsxToString = async () => {
  const inputFilePath = path.resolve(
    "문자열로 바꾸고 싶은 파일 경로",
  );
  const outputFilePath = path.resolve("문자열로 바꾼 후 저장될 파일 경로");

  try {
    // 파일을 문자열로 읽기
    const code = await fs.readFile(inputFilePath, "utf-8");

    // 읽은 문자열을 코드로 내보내기 형태로 변환
    const generatedCode = `
      export const tsxString = ${JSON.stringify(code)};
    `;

    // 변환된 코드를 파일에 쓰기
    await fs.outputFile(outputFilePath, generatedCode)
  } catch (error) {
    console.error("에러 발생:", error);
    process.exit(1); // 에러 발생 시 프로세스를 종료
  }
};

// 변환 함수 실행
convertTsxToString();

```

빌드 시점에 `convertTsxToString` 함수가 실행되어야 하므로, package.json에 스크립트를 따로 만들어야 합니다. 

```json
"dev": "vite",
"convert": "NODE_OPTIONS='--loader ts-node/esm' ts-node <convertTsxToString파일 경로>",
"build": "npm run convert-tsx && vite build",
```

런타임 환경은 `Node.js` 환경입니다. `Node.js`는 `TypeScript` 파일을 직접 실행할 수 없으므로 `ts-node` 를 설치해주어야 합니다. 

`ts-node`는 기본적으로 Node.js의 CommonJS 방식으로 TypeScript 파일을 처리합니다. ESM 환경에서 `ts-node`를 사용하려면, `--loader ts-node/esm` 옵션을 추가해야 Node.js가 ESM 방식으로 TypeScript를 처리할 수 있습니다.

Node.js 실행 옵션을 `npm run` 스크립트를 통해 전달하려면 `NODE_OPTIONS` 환경 변수를 사용해야 합니다. `NODE_OPTIONS`는 Node.js의 런타임 옵션을 전달할 수 있는 방법이며, `-loader ts-node/esm`를 Node.js 실행 프로세스에 추가하기 위해 사용됩니다.

`convert-tsx-to-string.ts`에서 사용하는 `import`, `export`, 경로 별칭, 타입 검사 등의 동작이 올바르게 작동하려면 `tsconfig.json` 설정도 필요합니다. 특히, **ESM 환경과의 호환성**을 위해 `tsconfig.json`에서 적절한 설정(`module: "ESNext"`, `target: "ESNext"`)을 해야 합니다.

프로젝트에서 적용한 `tsconfig` 설정은 다음과 같습니다. 

```tsx
{
  "compilerOptions": {
    "module": "ESNext", // ESM 형식의 import/export 사용
    "moduleResolution": "Node", // Node.js 방식으로 모듈 해석
    "target": "ESNext", // 최신 JavaScript 표준으로 컴파일
    "strict": true, // 엄격한 타입 검사 활성화
    "esModuleInterop": true, // CommonJS와 ES 모듈 간 호환 지원
    "jsx": "react-jsx", // React JSX 변환 (React 17+)
    "baseUrl": ".", // 프로젝트 루트를 기준 경로로 설정
    "paths": {
      "@/*": ["src/*"] // @로 별칭 지정
    }
  },
  "include": ["src"], // 컴파일할 파일
  "exclude": ["node_modules", "dist"] // 컴파일 제외 경로
}

```

이렇게 3가지 과정을 통해 

1. `import` 구문부터 `export` 구문, 컴포넌트 태그들까지 IDE에서 개발자가 보는 형태 그대로 문자열 값으로 바꿀 수 있었고
2. 바꾼 문자열을 변수에 저장해 `export` 하여
3. 해당 변수를 컴포넌트에 props로 넘겨줌으로써 중복 코드 줄이는 것

이 가능해 졌습니다.
