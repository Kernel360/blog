---
layout: post
title: "스프링 시큐리티"
author: "김자성"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ "JAVA" ]
---

## 주제: Spring Security - 인증 아키텍처

### 1. 시큐리티 전체 흐름도
Spring Security의 인증 과정에서 전체적인 흐름을 파악하는 것이 중요합니다.

### 2. 인증 절차 흐름
인증(Authentication)은 특정 자원에 접근하려는 사용자의 신원을 확인하는 과정입니다.

#### Authentication 개념
- 사용자의 신원을 확인하는 방법.
- 일반적으로 사용자 이름과 비밀번호 입력을 통해 수행됨.
- 인증이 성공하면 권한을 부여할 수 있음.
- `Authentication` 객체는 사용자 인증 정보를 저장하는 토큰 개념의 객체이며, `SecurityContext`에 저장되어 전역적으로 참조 가능.

#### Authentication 구조
- `getPrincipal()`: 인증 주체를 의미하며, 인증 요청 시에는 사용자 이름, 인증 후에는 `UserDetails` 타입 객체가 될 수 있음.
- `getCredentials()`: 인증을 증명하는 정보(일반적으로 비밀번호).
- `getAuthorities()`: 인증된 사용자의 부여된 권한.
- `isAuthenticated()`: 인증 상태 반환.

### 3. 인증 관련 주요 컴포넌트

#### AuthenticationFilter
- 인증 요청을 가로채고 인증을 처리하는 역할.

#### AuthenticationManager
- 인증 프로세스를 총괄하는 관리자 역할.

#### AuthenticationProvider
- 실제 인증 로직을 처리하는 컴포넌트.
- `AuthenticationManager`가 여러 개의 `AuthenticationProvider`를 가질 수 있음.
