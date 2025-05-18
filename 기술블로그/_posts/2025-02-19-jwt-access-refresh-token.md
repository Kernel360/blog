---
layout: post
title: "JWT 기반 인증: 액세스 토큰과 리프레시 토큰"
author: "박성민"
categories: "백엔드 기술블로그"
banner:
  image: assets/images/post/2023-11-10.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["JWT", "Spring Security"]
---

# 액세스 토큰과 리프레시 토큰

JWT 기반 인증 방식에서는 일반적으로 **액세스 토큰(Access Token)**과 **리프레시 토큰(Refresh Token)**을 함께 사용하여 보안성을 높입니다.

## 1. 액세스 토큰 (Access Token)

액세스 토큰은 사용자가 인증된 후 API 요청을 수행할 때 필요한 토큰으로, 보통 짧은 유효 기간을 가집니다.

- **사용 목적**: 인증된 사용자임을 증명하고, 요청을 처리할 권한을 부여하기 위해 사용됩니다.
- **유효 기간**: 보안성을 위해 짧게 설정 (예: 30분~1시간)
- **특징**:
  - 클라이언트는 요청마다 액세스 토큰을 `Authorization` 헤더에 포함하여 전송합니다.
  - 서버는 토큰을 검증한 후 요청을 처리합니다.

```java
public String generateAccessToken(String username) {
    return Jwts.builder()
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 1800000)) // 30분 후 만료
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
}
```

## 2. 리프레시 토큰(Refresh Token)

리프레시 토큰은 액세스 토큰이 만료되었을 때 새로운 액세스 토큰을 발급받기 위해 사용됩니다. 액세스 토큰보다 긴 유효 기간을 가지며, 보통 안전한 저장소(예: 데이터베이스)에 저장됩니다.

- **사용 목적**: 액세스 토큰이 만료된 경우, 새로운 액세스 토큰을 발급하는 데 사용됩니다.
- **유효 기간**: 비교적 길게 설정 (예: 7일~30일)
- **특징**: 
  - 서버에서 저장하고 관리해야 함.
  - 액세스 토큰보다 보안성이 높아야 하므로 클라이언트에서 쉽게 노출되지 않도록 주의해야 함.

```java
public String generateRefreshToken(String username) {
    return Jwts.builder()
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 604800000)) // 7일 후 만료
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
}
```

## 엑세스 토큰과 리프레시 토큰의 흐름

1. 사용자가 로그인하면 서버는 액세스 토큰과 리프레시 토큰을 생성하여 반환합니다.
2. 클라이언트는 액세스 토큰을 요청 헤더에 포함하여 API 요청을 보냅니다.
3. 액세스 토큰이 만료되면, 클라이언트는 리프레시 토큰을 서버에 보내어 새로운 액세스 토큰을 요청합니다.
4. 서버는 리프레시 토큰을 검증한 후 새로운 액세스 토큰을 발급하여 반환합니다.
5. 리프레시 토큰도 만료된 경우, 사용자는 다시 로그인해야 합니다.

## 리프레시 토큰을 이용한 재발급 API 구현
```JAVA
@RestController
@RequestMapping("/auth")
public class AuthController {
    @PostMapping("/refresh")
    public ResponseEntity<Map<String, String>> refreshToken(@RequestBody TokenRequest tokenRequest) {
        String refreshToken = tokenRequest.getRefreshToken();
        if (validateRefreshToken(refreshToken)) {
            String newAccessToken = generateAccessToken(extractUsername(refreshToken));
            Map<String, String> response = new HashMap<>();
            response.put("accessToken", newAccessToken);
            return ResponseEntity.ok(response);
        }
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}
```

## 5. 보안고려사항 

- 리프레시 토큰 저장 위치: HTTP-Only 쿠키 또는 보안 저장소에 보관하여 노출을 방지해야 합니다.
- 토큰 재사용 공격 방지: 리프레시 토큰을 재사용하면 즉시 폐기하는 로직을 추가해야 합니다.
- 강제 로그아웃 처리: 로그아웃 시 리프레시 토큰을 폐기하여 더 이상 사용할 수 없도록 해야 합니다.

## 6. 마무리

액세스 토큰과 리프레시 토큰을 함께 사용하면 보안성과 사용자 경험을 모두 향상시킬 수 있습니다.
Spring Security와 함께 적용하면 더욱 안전한 인증 시스템을 구축할 수 있습니다.

