---
layout: post
title: "JWT 기반의 인증 방식 학습"
author: "박성민"
categories: "백엔드 기술블로그"
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["JWT", "Spring Security"]
---
# JWT란?

JWT는 JSON 데이터를 Base64URL로 인코딩한 문자열 형태의 데이터를 이용해 사용자 인증 및 정보 교환을 안전하게 수행하는 토큰 기반 인증 방식입니다. JWT는 전자 서명을 포함하여 변조를 방지하며, 서버에서 세션을 유지하지 않고도 사용자 인증을 할 수 있도록 도와줍니다.

JWT는 주로 아래와 같은 구조로 구성됩니다:

#### 1. Header: 토큰 타입과 해싱 알고리즘을 정의합니다.

#### 2. Payload: 사용자 정보와 토큰의 만료 시간 등의 클레임(Claim) 데이터를 포함합니다.

#### 3. Signature: Header와 Payload를 비밀 키로 서명하여 변조를 방지합니다.

JWT는 일반적으로 헤더.페이로드.서명 형태의 문자열로 표현되며, Base64로 인코딩됩니다.


# JWT 기반 인증 방식

JWT는 인증(Authentication)과 인가(Authorization) 과정에서 사용됩니다.

## 인증 단계
#### 1. 사용자가 아이디와 비밀번호로 로그인 요청을 보냅니다.

#### 2. 서버에서 사용자의 인증 정보를 확인한 후 JWT를 생성합니다.

## 인가 단계
#### 3. 클라이언트는 JWT를 저장하고 이후 요청 시 Authorization 헤더에 포함하여 보냅니다.

#### 4. 서버는 클라이언트의 JWT를 검증한 후 요청을 처리합니다.

이 과정에서 서버는 사용자 세션을 유지할 필요가 없기 때문에 확장성이 뛰어난 인증 방식입니다.

# Spring Security에서 JWT 적용하기

Spring Security에서 JWT를 적용하려면 다음과 같은 단계가 필요합니다.

### 1. JWT 생성

JWT를 생성하기 위해 io.jsonwebtoken.Jwts 라이브러리를 사용합니다.
```bash
public String generateToken(String username) {
    return Jwts.builder()
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 3600000)) // 1시간 후 만료
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
}
```

### 2. JWT 검증

클라이언트가 보낸 JWT를 검증하고 사용자 정보를 추출하는 메서드입니다.
```bash
public Claims extractClaims(String token) {
    return Jwts.parser()
            .setSigningKey(SECRET_KEY)
            .parseClaimsJws(token)
            .getBody();
}

public boolean validateToken(String token) {
    try {
        extractClaims(token);
        return true;
    } catch (Exception e) {
        return false;
    }
}
```
### 3. JWT 필터 적용

JWT 인증을 처리하기 위해 OncePerRequestFilter를 구현한 필터를 생성합니다.
```bash
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String token = resolveToken(request);
        if (token != null && validateToken(token)) {
            Authentication authentication = getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        chain.doFilter(request, response);
    }
}
```
# JWT의 장점과 단점

### 장점

- Stateless: 서버에서 세션을 유지할 필요 없이 확장성이 뛰어남.

- 보안성: 서명된 토큰이므로 변조가 어렵고, 토큰을 통해 사용자 정보를 안전하게 전달할 수 있음.

- 빠른 인증: 데이터베이스 조회 없이 토큰만으로 인증 가능.

### 단점

- 토큰 무효화의 어려움: Access Token이 만료되더라도 즉시 강제 폐기할 방법이 없으며, Refresh Token을 통한 재발급이 가능하므로 보안 취약점이 발생할 수 있음. 특히, Access Token과 Refresh Token의 TTL(Time-To-Live) 설정에 따라 보안성과 사용자 편의성 간의 균형을 고려해야 함.

- 토큰 크기: JWT는 일반 세션 쿠키보다 크므로 요청 시 데이터 크기가 증가할 수 있음.

- 보안 위험: 토큰이 유출되면 해당 사용자의 권한을 탈취할 수 있음. 이를 방지하기 위해 HTTPS를 사용해야 함.

# 마무리

JWT는 세션 기반 인증보다 확장성이 뛰어나고 유지보수가 용이한 인증 방식입니다. Spring Security와 함께 사용하면 보안성을 높이면서도 효율적인 인증 시스템을 구축할 수 있습니다.
