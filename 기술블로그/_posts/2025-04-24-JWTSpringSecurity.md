---
layout: post  
title: "Spring Security로 구현한 JWT 기반 인증 시스템"
author: "조수연"
categories: "기술블로그"
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2025/0425/JWTSecurity.jpg?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`Spring Security`, `JWT`, `AccessToken`, `RefreshToken`, `인증`, `백엔드 보안`]
---


## 🔐 인증과 인가란?

**인증(Authentication)**은 사용자의 신원을 확인하는 절차입니다.  
예를 들어, 사용자가 아이디와 비밀번호를 입력하여 본인이 누구인지 증명하는 과정입니다.

**인가(Authorization)**는 인증된 사용자가 **어떤 리소스에 접근할 수 있는지**를 확인하는 절차입니다.  
예를 들어, 일반 사용자는 관리자 페이지에 접근할 수 없도록 제한하는 것이 인가입니다.

이 두 개념은 보안 시스템을 설계할 때 반드시 함께 고려되어야 하며, Spring Security는 이 인증과 인가를 체계적으로 처리할 수 있도록 도와주는 프레임워크입니다.

---

## 🧱 Spring Security란?

Spring Security는 Spring 기반 애플리케이션에서 인증과 인가를 처리하기 위한 보안 프레임워크입니다.  
주요 기능은 다음과 같습니다:

- 인증 및 권한 관리
- 세션 기반 또는 토큰 기반 인증 처리
- 다양한 공격 방지 (CSRF, 세션 고정, 클릭재킹 등)
- 커스터마이징 가능한 보안 필터 체인 제공

기본적으로 **Servlet Filter 기반의 체계적인 보안 구조**를 제공하며, 필터를 커스터마이징하여 다양한 인증 방식을 유연하게 적용할 수 있습니다.

---

## 🔑 왜 JWT 기반 인증인가?

기존의 **세션 기반 인증**은 사용자가 로그인할 때 서버가 세션을 생성하고, 이를 서버 메모리에 저장한 후 클라이언트에 세션 ID를 쿠키로 전달하는 방식입니다.  
이 방식은 서버가 상태를 유지해야 하기 때문에, 트래픽이 증가할수록 세션 관리 비용이 늘어나고 서버 간 세션 공유가 필요해지는 등의 한계가 있습니다.

**JWT(Json Web Token)**는 이러한 문제를 해결하기 위한 **무상태(Stateless)** 인증 방식입니다.  
JWT는 사용자 인증 정보를 자체적으로 포함하는 토큰으로, 서버는 별도의 세션을 저장할 필요 없이 **클라이언트가 전달하는 토큰만으로 인증 여부를 판단할 수 있습니다.**

---

## 🧩 JWT 구성 요소

JWT는 다음과 같은 세 부분으로 구성되어 있습니다:

1. Header – 토큰 타입과 서명 알고리즘
2. Payload – 사용자 정보 및 기타 클레임
3. Signature – 위조 방지를 위한 서명

JWT는 Base64로 인코딩되어 `.`으로 구분된 문자열로 표현됩니다.
> xxxxx.yyyyy.zzzzz

---

## 🛂 AccessToken과 RefreshToken

### ✅ AccessToken

AccessToken은 사용자의 인증 정보를 담고 있으며, 클라이언트는 이 토큰을 **HTTP 요청의 Authorization 헤더에 포함하여 서버에 전달**합니다.

```http
Authorization: Bearer <access_token>
```

AccessToken은 일반적으로 **만료 시간이 짧게 설정**되며, 서버는 이 토큰을 통해 사용자의 인증 여부를 판단합니다.  
보안상 노출될 위험이 있으므로 토큰 저장 및 전송 방식에 주의를 기울여야 합니다.

---

### ✅ RefreshToken

RefreshToken은 AccessToken이 만료되었을 때 **새로운 AccessToken을 발급받기 위한 용도**로 사용됩니다.  
만료 시간이 비교적 길며, 일반적으로 **HttpOnly 쿠키에 저장**하여 클라이언트 측 스크립트에서 접근할 수 없도록 보안성을 강화합니다.

RefreshToken을 사용하면 사용자는 매번 다시 로그인하지 않아도 되고, 서버는 유효한 RefreshToken을 바탕으로 새로운 AccessToken을 발급해 **지속적인 인증 상태를 유지할 수 있습니다.**

---

## 🔐 Spring Security와 JWT 통합 구조

Spring Security에서 JWT 인증을 구현할 때는 다음과 같은 구성 요소가 사용됩니다.
![NON-REPEATABLE READ](https://github.com/Kernel360/blog-image/blob/main/2025/0425/jwtFilter.png?raw=true)

### 1. SecurityFilterChain

- 인증/인가 처리 흐름을 정의하는 보안 설정 클래스입니다.
- CSRF, 세션, 경로 권한 설정, 예외 처리 핸들러, 사용자 정의 필터 등록 등을 수행합니다.

### 2. JwtFilter

- 요청마다 Authorization 헤더에 포함된 토큰을 추출하고, 유효한 경우 인증 객체를 SecurityContext에 등록합니다.

### 3. TokenProvider

- JWT 생성, 유효성 검사, 사용자 정보 추출 등의 기능을 담당합니다.
- 일반적으로 io.jsonwebtoken 라이브러리를 사용하여 토큰을 처리합니다.

### 4. AuthenticationEntryPoint & AccessDeniedHandler

- 인증되지 않은 사용자의 요청 → 401 Unauthorized 응답
- 인가되지 않은 사용자의 요청 → 403 Forbidden 응답

---

## 🔎 토큰 유효성 검사는 어떻게 이뤄지는가

JWT 인증에서 중요한 과정 중 하나는 **클라이언트가 보낸 토큰이 유효한지 검사하는 단계**입니다.  
이 검사는 서버의 `TokenProvider` 클래스 내에서 수행되며, 일반적으로 다음과 같은 항목을 확인합니다:

- 토큰의 서명이 위조되지 않았는가
- 토큰이 만료되지 않았는가
- 토큰에 필요한 클레임이 모두 포함되어 있는가

토큰이 유효하지 않다면 서버는 인증 실패로 간주하고, 요청은 컨트롤러까지 도달하지 않습니다.  
이러한 처리는 `JwtFilter` 내부에서 이루어지며, 유효한 토큰일 경우에만 인증 객체를 SecurityContext에 등록합니다.
---
## 🔒 SecurityContext와 인증 객체 주입

Spring Security는 인증이 완료된 사용자의 정보를 `SecurityContext`에 저장합니다.  
이 컨텍스트는 현재 스레드(Local Thread)에 바인딩되어 있어, 인증 이후의 모든 흐름에서 사용자의 인증 정보를 확인할 수 있습니다.

`JwtFilter`에서 인증이 성공하면 다음과 같이 인증 객체를 등록합니다:

```java
SecurityContextHolder.getContext().setAuthentication(authentication);
```
이후 컨트롤러에서는 `@AuthenticationPrincipal` 또는 `Principal`을 통해 로그인한 사용자 정보를 꺼내 사용할 수 있게 됩니다.

---
## 📌 마무리

JWT 기반 인증은 세션을 관리하지 않아도 되기 때문에 서버의 부담이 줄고, 다양한 클라이언트에서 유연하게 사용할 수 있는 장점이 있습니다.  
Spring Security는 처음에는 복잡하게 느껴질 수 있지만, 각 구성 요소의 역할을 명확히 이해하고 분리하면 유지보수와 확장성 측면에서도 매우 유리한 구조를 만들 수 있습니다.

이번 글에서는 인증과 인가의 기본 개념부터 시작해, JWT의 구조와 동작 방식, 그리고 이를 Spring Security와 통합하여 사용하는 방법까지 전반적인 흐름을 정리해 보았습니다.  
실무에서 이와 같은 인증 구조를 직접 설계하고 구현해보는 과정은 백엔드 보안에 대한 이해도를 높이는 데 큰 도움이 되며, 더 나아가 사용자 경험과 시스템 안정성을 동시에 고려한 인증 시스템을 설계하는 데 기반이 됩니다.
