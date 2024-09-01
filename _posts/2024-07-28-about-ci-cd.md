---
layout: post
title: CI/CD의 개념과 필요성
author: 윤해진
categories: 기술세미나
banner:
  image: https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0223/spring-batch-tutorial.jpeg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ci/cd, 기술세미나]
---

### 1. 이런 글을 쓰게 된 배경

최근 진행한 헤커톤에서 팀원 교체를 경험하게 되었습니다.

서로 다른 코드 스타일로 인해 전체 소스코드가 개발이 진행되며 조금씩 어지러워진다는 느낌이 들기도 하였습니다. 이는 프로젝트의 규모가 커지며 더 심각하게 느껴질 수 있는 부분일 것입니다. 따라서 저는 위와같은 어려움을 겪으며 CI(지속적 통합)와 CD(지속적 배포)의 필요성을 느끼게 되었습니다.

팀원이 변경될 때마다 발생하는 문제를 최소화하고, 코드 품질을 유지하며, 개발 속도를 높일 수 있는 방법을 찾기 위해 CI/CD와 관련하여 글을 작성하게 되었습니다.

이 글에서는 CI/CD가 왜 중요한지, 어떻게 구현할 수 있는지, 그리고 앞으로 프로젝트에 어떻게 적용가능한지를 공유하고자 합니다.

### 2. CI/CD가 왜 필요한가요?

CI/CD는 소프트웨어 개발에서 중요한 역할을 하는 두 가지 개념입니다.

**2.1 CI(지속적 통합)**

지속적 통합(CI)은 개발자가 작성한 코드를 자주 그리고 자동으로 통합하는 과정을 의미합니다.

- **빠른 버그 발견 및 수정**: 코드 변경이 발생할 때마다 자동으로 테스트를 실행하여 버그를 조기에 발견하고 수정할 수 있습니다.
- **코드 일관성 유지**: 여러 개발자가 동시에 작업하더라도 코드베이스의 일관성을 유지할 수 있습니다.
- **통합의 부담 감소**: 작은 단위로 자주 통합함으로써 대규모 통합의 부담을 줄일 수 있습니다.

**2.2 CD(지속적 배포)**
지속적 배포(CD)는 코드를 자동으로 빌드, 테스트, 배포하는 과정을 포함합니다.

- **빠른 기능 제공**: 새로운 기능을 사용자에게 빠르게 제공할 수 있습니다.
- **높은 배포 신뢰성**: 자동화된 배포 프로세스를 통해 배포 과정에서 발생할 수 있는 실수를 줄일 수 있습니다.
- **배포 프로세스 간소화**: 반복적이고 수동적인 배포 작업을 자동화하여 개발자의 부담을 줄입니다.

이러한 이점으로 지속적인 통합과 배포를 하여 소프트웨어 개발 라이프 사이클을 간소화, 가속화가 가능합니다.

### 3. 어떻게 CI/CD를 수행할 수 있나요?

CI/CD를 구현하기 위해서는 다음과 같은 도구와 원칙을 따를 수 있습니다.

**3.1 소스 코드 관리**

- 소스코드를 적절하게 관리함으로 코드의 변경사항을 효율적으로 관리할 수 있습니다.

**3.2 자동화된 빌드**

- 빌드툴을 사용하여 코드 자동 빌드가 가능합니다.

- 프로젝트에 맞는 빌드 스크립트를 작성하여 빌드 과정을 자동화할 수 있습니다.

- 추가적으로 빌드 실패시 다양한 피드백을 받을 수 있습니다.

**3.3 자동화된 테스트**

- 다양한 테스트를 통하여 각 모듈, 혹은 더 상위의 개념을 테스트 할 수 있습니다
  이런 테스트는 자동으로 실행되며, 테스트 결과를 즉시 확인 가능합니다.

**3.4 코드 스타일 검사**

- 사람들 마다 다른 코드 스타일을 일정한 규칙에 맞춰 검사 할 수 있습니다.
  검사 결과를 즉각적로 확인하여 코드의 일관성을 유지할 수 있습니다.

**3.5 자동화된 배포**

- 다양한 배포툴을 사용하여 자동화 된 배포가 가능합니다.

**3.6 모니터링과 로깅**

- 다양한 모니터링 도구를 사용하여 애플리케이션 상태를 실시간으로 모니터링 가능합니다.
  이로 프로젝트에 문제가 발생하였을 때, 신속한 대응이 가능합니다.

### 4. 우리 프로젝트에 어떻게 녹일 수 있나요?

앞으로 프로젝트를 진행할 때 아래와 같은 방법으로 CI/CD를 적용할 수 있습니다.

**4.1 프로젝트 초기 설정**

- **GitHub 사용**: 소스 코드를 GitHub에 저장하고, 팀원들과의 협업을 위해 브랜치 전략을 도입합니다.
- **GitHub Actions 설정**: GitHub Actions를 사용하여 코드 변경이 있을 때마다 자동으로 빌드 및 테스트가 실행되게 합니다.

**4.2 자동화된 테스트**

- **테스트 스크립트 작성**: 단위 테스트와 통합 테스트를 자동으로 실행하는 스크립트를 작성합니다.
- **CI 파이프라인 구성**: GitHub Actions를 통해 각 코드 변경 시 자동으로 테스트가 실행되도록 파이프라인을 구성합니다.

**4.3 코드 스타일 검사 도입**

- **ESLint 설정**: JavaScript 코드의 일관성을 유지하기 위해 ESLint를 설정하고 자동화된 검사 과정을 추가합니다.
- **Stylelint 설정**: CSS 코드의 일관성을 유지하기 위해 Stylelint를 설정하고 자동화된 검사 과정을 추가합니다.
- **GitHub Actions에서의 Lint 체크**: 코드 변경 시마다 자동으로 ESLint와 Stylelint를 실행하여 코드 스타일을 검사하고, 문제가 있는 경우 이를 바로 피드백가능합니다.

**4.4 자동화된 배포**

- **Vercel 사용**: Vercel을 통해 애플리케이션을 자동으로 배포했습니다. GitHub와 연동하여 코드 변경이 발생할 때마다 자동으로 빌드 및 배포가 이루어지도록 설정합니다.

위와 같은 방향으로 CI/CD를 프로젝트에 녹일 수 있을 것입니다.

다양한 테스트 방법과 기술들이 있지만, 프로젝트의 성격과 성향에 따라 적절한 방식으로 CI/CD를 구축하는 것도 매우 중요한 부분일 것입니다.

### 5. 우리가 사용한 방법( github Action )

1차적으로는 airbnb의 lint 룰을 기본으로 하였습니다.

그러나 해당 룰이 너무 과하거나, 어떤 의존성으로 사용이 불가능 할 경우에는 팀원들과의 논의를 통해 lint rule에서 off를 하는 방향을 선택하였습니다. 추가적으로 팀 내부에서 컨벤션을 통해 맞춰진 코드 스타일 등은 추가적으로 lint룰을 설정하여 코드 스타일을 맞추었습니다.

이렇게 코드의 스타일을 지정하였으나, 깜빡하고 챙기지 못하는 경우도 있습니다. 따라서 위에서 이야기를 나누었던, Github Action을 활용하여, 자동적으로 체크해주도록 하였습니다.(추가적으로 tsc 검사도 하였습니다.)

해당 액션은 tsc와 lint를 체크하는 액션으로 main, develop, hotfix로 push, pr을 날릴 때, Action이 실행됩니다.

[action 검사 코드]

```tsx

name: check tsc and lint
on:
  push:
    branches:
      - main
      - develop
      - hotfix
  pull_request:
    branches:
      - main
      - develop
      - hotfix

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install

      - name: Check tsc
        run: npx tsc

      - name: Check eslint
        run: npm run lint

```

pr을 날리거나, 설정한 브랜치로 push를 할때 설정한 action을 통해 자동 검사가 진행되니 코드에 대한 일관성을 지킨채로 프로젝트를 진행 할 수 있었습니다.

<br/>

[실제 사용 시]

<table>
  <tr>
    <td style="vertical-align: top;">
      <img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/1-1.png" alt="1" width="300"/>
    </td>
    <td style="vertical-align: top;">
      <img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/1-2.png" alt="2" width="300"/>
    </td>
  </tr>
</table>

dev혹은 main으로 merge하는 pr을 올렸을 때, github Action이 돌아 결과를 타이틀 옆에 표시해줍니다.
해당 pr을 들어가 확인해보면 아래와 같이 에러가 난 것을 확인 가능했습니다.

![3](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/2.png)

현재 tsc 체크 부분에서 에러가 나, 멈춘 것으로 보이게 됩니다.

위 처럼 작성하신 코드에서 tsc, lint를 체크하고 오류가 있다면 표현해줍니다.

만약 에러가 있을때는 다시 코드를 수정하시고 커밋을 작성하시고 push를 하시면 반영이 다시 되어 action이 동작합니다.

![4](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/3.png)

새로운 커밋을 push 하면 새롭게 action이 도는 모습을 확인 할 수 있었습니다.

올바르게 수정이 된다면 아래처럼 초록색으로 체크 표시와 pr에 표시가 됩니다.

<table>
  <tr>
    <td style="vertical-align: top;">
      <img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/4-1.png" alt="5" width="300"/>
    </td>
    <td style="vertical-align: top;">
      <img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0901/4-2.png" alt="6" width="300"/>
    </td>
  </tr>
</table>

이렇게 스스로가 깜빡하고 lint, tsc를 확인하지 못했을 경우 github Action에서 자동으로 검사를 돌아주는 코드가 하나의 방식으로 통일 되었다는 것을 느낄 수 있었습니다.

<br/>

### 6. 요약

CI/CD는 소프트웨어 개발에서 코드 품질을 유지하고, 개발 속도를 높이며, 팀 협업을 강화하는 데 필수적인 요소입니다. CI/CD를 통해 프로젝트의 일관성을 유지하고, 새로운 팀원이 쉽게 적응할 수 있도록 도울 수 있습니다.

팀이 바뀌는 경험하며 CI/CD의 중요성을 다시 한 번 깨달았으며, 앞으로의 진행하는 프로젝트에 이를 적용하여 더욱 효율적이고 안정적인 개발을 이루고자 합니다.
