
---
layout: post  
title: `Github Actions`
author: `이종찬`
categories: `기술세미나`
banner:
  image: `![](https://github.com/Kernel360/blog-image/blob/main/1208/1.png)`
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`GithubActions`]
---

목차
- 목표
- CI / CD란?
- GithubActions란?
	- workflow?
	- 사용해야하는 이유?
	- 장점
	- 단점
- 정리

## 목표

CI / CD, Github Actions이 무엇이고, 어떻게 사용할 수 있는지, 그리고 어떤 장단점이 있는지를 알고 사용하는 것을 목표로 글을 작성합니다.

## 1. CI / CD란?

CI/CD는 Continuous Integration과 Continuous Delivery 또는 Continuous Deployment의 약자로, 소프트웨어 개발에서 흔히 사용되는 방법론입니다.

### 1-1. Continuous Integration

![](https://github.com/Kernel360/blog-image/blob/main/1208/2.png)

CI는 지속적인 통합이라는 뜻으로, 개발자들이 코드를 자주 병합하고, 빌드하고, 테스트하는 과정을 말합니다. CI를 통해 코드의 품질을 높이고, 버그를 줄이고, 협업을 쉽게 할 수 있습니다.

### 1-2. Continuous Deployment

CD는 지속적인 배포 또는 지속적인 전달이라는 뜻으로, CI의 결과물을 자동으로 배포하거나, 배포 준비 상태로 만드는 과정을 말합니다. CD를 통해 배포의 속도와 안정성을 높이고, 고객의 피드백을 빠르게 반영할 수 있습니다.

![](https://github.com/Kernel360/blog-image/blob/main/1208/3.png)

### 1-3. 장점

CI / CD 환경이 필요한 이유는 반복 작업의 자동화 및 피드백 루프 단축 등을 통해 소프트웨어 릴리스 프로세스의 속도를 개선하는 것 입니다.

![](https://github.com/Kernel360/blog-image/blob/main/1208/4.png)

짧은주기의 개발단위를 반복하며, 많은 협력과 피드백을 필요로 하는 애자일의 원칙을 실현하는 데 핵심적인 역할을 합니다.
![](https://github.com/Kernel360/blog-image/blob/main/1208/5.png)

## 2. Github Action

Github Actions은 Github에서 제공하는 CI/CD 도구입니다.

Github 저장소에서 발생하는 다양한 이벤트에 따라 원하는 작업을 자동화할 수 있으며 workflow라는 단위로 구성되어있습니다.

![](https://github.com/Kernel360/blog-image/blob/main/1208/1.png)

### 2-1. Workflow

Workflow는 Github Actions에서 자동화할 수 있는 작업의 흐름을 의미합니다. ./github/workflows 폴더에 저장되며 다음과 같은 요소로 구성됩니다.

- name: Workflow의 이름을 지정합니다. (선택 사항)
- on: Workflow를 트리거할 이벤트를 지정합니다. (필수 사항)
- jobs: Workflow에서 실행할 Job들을 지정합니다. (필수 사항)   
- env: Workflow 전체에서 사용할 환경 변수를 지정합니다. (선택 사항)

```yml
# Workflow 이름
name: Java CI with Gradle

# workflow trigger
on:
  push:
    branches: [ "develop" ]

# permission
permissions:
  contents: read

# Job
jobs:
  build:

	# Job 실행 환경
    runs-on: ubuntu-latest
    
	# Job Step
    steps:
	    # Step Action
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        
        # Step Input
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run chmod to make gradlew executable
        # Step 커맨드
        run: chmod +x ./gradlew

      - name: Build with Test
        run: ./gradlew clean build test

```

### 2-2. 장점

- Github Actions를 사용하면 다음과 같은 이유로 CI/CD를 쉽고 편리하게 구현할 수 있습니다.
- Workflow를 YAML 파일로 간단하게 정의하고, 코드와 함께 버전 관리할 수 있습니다.
- Github 마켓플레이스에서 수많은 Action을 찾아서 사용할 수 있으며, 자신이 만든 Action을 공유하거나 재사용할 수 있습니다.
- Workflow를 트리거할 수 있는 이벤트의 범위가 넓습니다.

### 2-3. 단점

- Workflow의 실행 시간과 리소스에 제한이 있습니다.
- Workflow의 실행 환경에 따라서, 도커 컨테이너로 작성한 Action을 사용할 수 없는 경우가 있습니다.
- Workflow의 실행 결과와 로그를 Github 외부에서 확인하거나, 공유하기 어려울 수 있습니다.
- Workflow의 테스트와 디버깅을 위한 툴이 부족할 수 있습니다.

## 정리

- CI / CD는 지속적인 통합과 지속적인 배포를 의미하며, 소프트웨어 출시 및 운영에 필요한 반복적인 작업을 빠르게 안정적으로 가져가기 위해 필요
- Github Action은 CI / CD 환경 구축에 대한 리소스를 최소화 하며 사용할 수 있으며 트리거 할 수 있는 이벤트 범위가 넓기 때문에 사용
- Github Actions의 단점은 workflow에 대한 테스트 및 디버깅을 할 수 있는 수단이 부족하며 외부에서 로그 확인이 힘듬
