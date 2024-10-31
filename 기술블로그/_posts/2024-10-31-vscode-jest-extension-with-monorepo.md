---
layout: post
title: "monorepo에서 vscode jest extension 환경 설정"
author: "김승태"
categories: "프론트엔드 기술블로그"
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["jest", "monorepo", "vscode"]
---

## Jest란?

Jest는 메타(구 페이스북)에서 개발한 자바스크립트 테스트 라이브러리입니다. 간편하게 테스트를 할 수 있어 자바스크립트 개발자들에게 많이 사용되고 있습니다.

## Jest vscode extension

Jest는 cli를 이용해서 jest 실행 명령어를 통해 테스트를 실행할 수 있습니다.

`jest`, `jest <특정 파일>`, `jest --watch` 등의 커맨드에 명령어를 입력해도 되지만, vscode에서는 더 간단하게 jest를 실행할 수 있는 extension이 존재합니다.

![alt text](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/1.png)

jest extension은 jest 명령을 찾을 수 있으면 기본적으로 실행 시 watch mode에서 모든 테스트를 자동으로 실행하고 모니터링하며,

테스트, 상태, 오류, 커버리지(활성화된 경우)를 TestExplorer 및 편집기에서 다음과 같이 확인할 수 있습니다.
![alt text](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/2.png)

또한 아래와 같이 어떤 테스트가 실패했는지 가시적으로 살펴보기도 쉽습니다.

| **extension 적용 전**                                                                  | **extension 적용 후**                                                                 |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| ![before](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/3.png) | ![after](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/4.png) |

저는 해당 extension의 도움을 많이 받았었습니다.

## monorepo에서 jest extension 관련해 마주한 문제와 해결과정

저희는 이번 [파이널 프로젝트](https://github.com/Kernel360/F2-TECHPICK)를 진행하면서 웹 서비스와 익스텐션을 개발해야했고, 통일된 버전 관리와, 테마를 가져가기 위해 모노레포를 적용했습니다.
그리고 각 레포마다 jest설정을 적용했었는데요.

```
프로젝트 루트
├── 백엔드
└── 프론트엔드 모노레포(.vscode 폴더가 존재함)
    ├── 웹서비스(jest.config 존재)
    └── 익스텐션
    └── 공통 테마
```

하지만 이번 프로젝트에서는 jest extension에서 문제가 발생했습니다.

```
techpick/src/stores/test.spec.ts
  ● Test suite failed to run

    Jest encountered an unexpected token

    Jest failed to parse a file. This happens e.g. when your code or its dependencies use non-standard JavaScript syntax, or when Jest is not configured to support such syntax.

    Out of the box Jest supports Babel, which will be used to transform your files into valid JS based on your Babel configuration.

    By default "node_modules" folder is ignored by transformers.

    Here's what you can do:
     • If you are trying to use ECMAScript Modules, see https://jestjs.io/docs/ecmascript-modules for how to enable it.
     • If you are trying to use TypeScript, see https://jestjs.io/docs/getting-started#using-typescript
     • To have some of your "node_modules" files transformed, you can specify a custom "transformIgnorePatterns" in your config.
     • If you need a custom transformation specify a "transform" option in your config.
     • If you simply want to mock your non-JS modules (e.g. binary assets) you can stub them out with the "moduleNameMapper" config option.

    You'll find more details and examples of these config options in the docs:
    https://jestjs.io/docs/configuration
    For information about custom transformations, see:
    https://jestjs.io/docs/code-transformation

    Details:

    /Users/gimseungtae/Desktop/GIT/F2-TECHPICK/frontend/techpick/src/stores/test.spec.ts:1
    ({"Object.<anonymous>":function(module,exports,require,__dirname,__filename,jest){import { describe, it, expect } from '@jest/globals';
                                                                                      ^^^^^^

    SyntaxError: Cannot use import statement outside a module

      at Runtime.createScriptFromCode (../../../../.yarn/berry/cache/jest-runtime-npm-29.7.0-120fa64128-10c0.zip/node_modules/jest-runtime/build/index.js:1505:14)
```

처음에는 jest의 설정을 잘못한 탓에 발생한 문제인 줄 알았으나 cli를 이용해 테스트를 하니 올바른 결과를 수행했었습니다.

| **cli 실행(jest --watch)**                                                             | **vscode jest extension**                                                             |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| ![before](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/5.png) | ![after](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1031/6.png) |

처음 예상했던 jest의 설정이 문제가 아니었습니다. 따라서 extension에서 jest를 다른 곳에서 실행한다는 느낌을 받았습니다.

.vscode 폴더가 존재하는 루트 폴더를 기준으로 extension이 실행되는 것일까요?

그를 검증해보기 위해 프론트엔드 모노레포에 jest 관련 설정을 하고 간단한 테스트를 적용해보니, vscode extension이 동작하는 것을 확인할 수 있었습니다.

그러면 각 레포를 기준으로 extension을 실행할 수 있게 하면 동작할 것 같았습니다.

그를 위해 [vscode-jest](https://github.com/jest-community/vscode-jest) 공식 깃허브에 들어가 문서를 찾아보았습니다.

> you can use the extension with monorepo projects, see monorepo project support for details.

안내에 따라 설명을 읽어보니 3가지의 방법이 있었습니다.

1. Single-root workspace:

   모노레포 패키지의 모든 테스트를 중앙 집중식 위치(예: 프로젝트 루트)에서 실행할 수 있는 경우, 적절한 jest.jestCommandLine 및 jest.rootPath 설정을 사용하여 싱글 루트 워크스페이스가 작동합니다.

2. Multi-root workspace:

   각 모노레포 패키지가 자체 로컬 Jest 루트 및 구성 파일을 가지고 있는 경우, 멀티 루트 워크스페이스가 필요합니다. 사용자는 jest.disabledWorkspaceFolders를 사용하여 Jest 실행에서 패키지를 제외할 수 있습니다.

3. Virtual folders:

   v6부터 사용자는 루트 폴더에서 모노레포 패키지를 구성하기 위해 가상 폴더를 사용할 수 있습니다. 가상 폴더에 대한 자세한 내용은 [관련 문서](https://github.com/jest-community/vscode-jest?tab=readme-ov-file#virtual-folders)를 참조하십시오.

저는 3번을 적용했습니다. 이유는 아래와 같습니다.

- 저희 프로젝트의 경우 각 패키지마다 jest가 있을 것이므로 1번은 적용할 수 없습니다.
- 2번 멀티 루트 스페이스의 경우 왼쪽의 폴더 구조의 UI가 달라졌기에 낯설어 적용하지 않았습니다.

방법은 간단합니다.
.vscode 폴더에 존재하는 `settings.json`에 virtual folder 설정을 적용주면 됩니다. 공식문서에선 아래와 같은 예시를 보여주고 있습니다.

```
{
  "jest.virtualFolders": [
    {"name": "package1", "rootPath": "packages/package1"},
    {"name": "package2", "rootPath": "packages/package2"}
  ]
}
```

이를 저희 프로젝트에 적용했습니다.

```
"jest.virtualFolders": [{ "name": "techpick", "rootPath": "./techpick" }]
```

해당 설정을 하고 vscode를 다시 켜주니, 정상적으로 동작함을 확인할 수 있었습니다.
