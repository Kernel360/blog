---
layout: post
title: 생산성을 도와줄 프로그램들
author: 이종찬
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0118/1.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [기술세미나]
---

# 생산성 향상을 위한 도구들

## 생산성이란?

- 좋은 개발 환경 구축 : 효율적인 작업 환경 ex)IDE, 버전관리 도구, 디버깅 도구...
- 자동화 : 시간을 소모하는(많이)작업을 자동화 하여 시간 단축
- 코드 품질 관리 : 코드 리뷰, 정적 분석 도구 활용
- 학습과 성장 : 최신 개발 트렌드 지식 유지 및 기술 향상
- 작업 관리 : 우선 순위 설정과 같은 체계적으로 단계를 형성하여 관리

이중에서 시간을 아낄 수 있는 효율에 초점을 맞추어 글을 작성하도록 하겠습니다.

## 목표

![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/1.png?raw=true)

## 목차
- JPA Buddy
- IntelliJ Live Template
- Github Copilot
- Octotree
- 마치며

## 1. JPA Buddy

![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/2.png?raw=true)

JPA Buddy는 자바 개발자를 위한 플러그인으로, IntelliJ IDEA에서 엔티티 클래스와 매핑 파일을 자동으로 생성하고 관리할 수 있습니다. 
JPA Buddy는 데이터베이스 스키마와 동기화하고, 쿼리를 테스트하고, JPA 관련 코드를 쉽게 작성할 수 있도록 도와줍니다.

### Table -> Entity
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/4.png?raw=true)
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/5.png?raw=true)

### Entity -> DTO
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/6.png?raw=true)
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/7.png?raw=true)

JPA Buddy를 사용하여 DB에 있는 Table을 Entity로, Entity를 DTO로 변환할 수 있는 것을 확인할 수 있습니다.
약간 아쉽지만 column이 많을 수록 불필요한 작업을 안해도 된다는 점은 좋아보입니다. 아쉬운 부분은 뒷부분에서 채워보겠습니다.

---
## 2. IntelliJ Live Template

IntelliJ Live Template은 IntelliJ IDEA에서 반복적인 코드를 작성할 때 시간을 절약할 수 있는 기능입니다.
Live Template은 미리 정의된 코드 조각을 키워드와 함께 저장하고, 필요할 때 키워드를 입력하면 코드 조각이 자동으로 삽입되는 방식으로 작동합니다.
개발을 하다보면 반복하는 작업이 많습니다. 아래와 같은 작업을 템플릿화 해놓으면 실수도 줄이고 시간 절약도 할 수 있습니다.

### LiveTemplate을 이용한 Entity Template
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/9.png?raw=true)
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/10.png?raw=true)

### JpaBuddy X LiveTemplate

그렇다면 두개의 플러그인을 잘 활용하면 어떻게 생산성을 높일 수 있을까요?
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/110.png?raw=true)

### Notice

실질적으로 사용할 수 있는 코드를 쉽고 빠르게 작성할 수 있습니다.
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/12.png?raw=true)

이러한 프로그램들의 도움이 없었다면 
같은 결과를 도출해도 소모되는 시간이 크게 차이가 나게 됩니다.

---
## 3. Github Copilot

GitHub에서 2021년 출시한 서비스로 개발자가 직접 개발한 코드를 AI가 이해하여 다음 작성 코드를 자동 완성해서 먼저 제시해주는 기능입니다.
코드 에디터에 통합되어 개발자가 작성하는 코드를 분석하고, 적절한 코드 조각을 제안하며,다양한 언어와 프레임워크를 지원합니다.

### Github Copilot 적용 모습
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/11.png?raw=true)

앞서 배운 JPA Buddy 플러그인으로 DTO를 만들고 Copilot이 제안한 코드가 나오는 모습입니다. 
첫 사용에서는 코드AI가 좋지 못하지만 비슷한 유형의 코드를 반복하다보면 원하는 코드를 쉽게 작성할 수 있습니다.

## 4. Octotree
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/15.png?raw=true)

Octotree는 GitHub, GitLab, Bitbucket 등의 코드 호스팅 플랫폼에서 프로젝트를 탐색할 때 유용한 플러그인입니다. 
웹 브라우저에서 코드 저장소의 파일 구조를 트리 형태로 보여주고, 원하는 파일을 빠르게 열 수 있으며 
다크 모드, 코드 검색, 풀 리퀘스트 리뷰 등의 기능도 제공합니다.

기존의 Github에서 코드를 열람하려면 다른 코드로 이동할 때마다 새로고침하며 불편하게 열람했지만 
Octotree를 사용하면 새로고침 없이 레포지토리의 코드를 쉽게 파악할 수 있는 것을 확인할 수 있습니다.
![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/16.png?raw=true)

## 마치며

![](https://github.com/Kernel360/blog-image/blob/main/2024/0118/220.png?raw=true)

이 밖에도 많은 생산성에 도움을 주는 도구들이 있습니다. 
어떻게 하면 생산성을 높일 수 있을까? 라는 생각이 개발을 하는데 많은 도움이 된다고 생각하여 글을 작성하였고 많은 도움이 되었으면 좋겠습니다.
