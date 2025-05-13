---
layout: post
title: "Redis 직렬화 문제 해결기: serialVersionUID에서 GenericJackson2JsonRedisSerializer로"
author: "이영석"
categories: "백엔드 기술블로그"
banner:
  image: 
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [`Redis`, `직렬화`, `Spring`, `트러블슈팅`, `Java`, `Spring Security`]
---

안녕하세요! 오늘은 HouseHub 프로젝트를 진행하면서 겪었던 Redis 직렬화 문제와 그 해결 과정을 공유하고자 합니다.

## Spring Security와 Redis의 연동

우리 팀은 HouseHub 프로젝트에서 Spring Security와 Redis를 함께 사용하여 사용자 인증을 구현했습니다. 이 조합을 선택한 이유는 다음과 같습니다:

- **서버 확장성**: 여러 서버에서 동일한 세션 정보를 공유할 수 있습니다
- **세션 지속성**: 서버를 재시작해도 세션 정보가 유지됩니다
- **빠른 접근**: 인메모리 저장으로 인해 세션 조회가 매우 빠릅니다

처음에는 다음과 같이 Redis 설정을 했습니다:

```java
@Configuration
@EnableRedisHttpSession
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return template;
    }
}
```

이렇게 설정하고 나서는 사용자가 로그인하면 Spring Security가 인증을 처리하고, 인증이 성공하면 세션 정보를 Redis에 저장하는 방식으로 잘 동작했습니다. 클라이언트는 세션 ID를 쿠키로 받아서 이후 요청에 포함시켰고, 서버는 이 ID를 통해 Redis에서 세션 정보를 조회하여 인증을 처리했습니다.

## 문제의 시작

그런데 어느 날 갑자기 로그인과 회원가입이 동작하지 않는 상황이 발생했습니다. 에러 로그를 확인해보니 다음과 같은 메시지가 있었습니다:

```
org.springframework.data.redis.serializer.SerializationException: Cannot deserialize
```

이 에러는 Redis에서 데이터를 역직렬화하는 과정에서 발생했습니다. 원인을 파악해보니, 우리가 사용하던 JdkSerializationRedisSerializer가 문제였습니다. 이 직렬화 방식은 Java의 기본 직렬화를 사용하는데, 클래스 구조가 변경되면 serialVersionUID가 자동으로 재계산되어 호환성 문제가 발생할 수 있습니다.

문제가 발생한 과정은 다음과 같습니다:

1. Agent 도메인의 enum 클래스들을 리팩토링하면서 기존 entity 패키지에서 enums 패키지로 이동
2. AgentResDto의 import 문이 변경되면서 serialVersionUID가 자동으로 변경
3. 기존에 저장된 Redis 데이터와 새로운 serialVersionUID가 불일치하여 역직렬화 실패

## Redis Serializer의 세계

이 문제를 해결하기 위해 Redis Serializer에 대해 자세히 알아보았습니다. Redis Serializer는 Java 객체를 Redis에 저장할 수 있는 형태로 변환하고, 다시 Java 객체로 복원하는 역할을 합니다. Spring Data Redis는 여러 가지 Serializer를 제공하는데, 각각의 특징이 다릅니다.

### JdkSerializationRedisSerializer

- Java의 기본 직렬화 방식을 사용
- Serializable 인터페이스를 구현한 클래스만 직렬화 가능
- serialVersionUID를 사용하여 클래스 버전 관리
- 장점:
  - 구현이 간단하고 모든 필드가 자동으로 직렬화
- 단점:
  - 클래스 구조 변경 시 역직렬화 실패 가능
  - 데이터 크기가 크고 가독성이 떨어짐
  - 다른 언어와의 호환성 없음

### GenericJackson2JsonRedisSerializer

- Jackson 라이브러리를 사용하여 JSON 형식으로 직렬화
- 클래스 정보를 JSON에 포함시켜 정확한 타입으로 복원
- 장점:
  - 클래스 구조 변경에 유연하게 대응 가능
  - JSON 형식으로 저장되어 가독성이 좋음
  - 다른 언어와의 호환성이 좋음
- 단점:
  - 직렬화/역직렬화 속도가 조금 느림
  - 메모리 사용량이 더 많을 수 있음

## 해결 과정

이러한 분석을 바탕으로, 우리는 직렬화 방식을 JdkSerializationRedisSerializer에서 GenericJackson2JsonRedisSerializer로 변경하기로 결정했습니다. 변경된 Redis 설정은 다음과 같습니다:

```java
@Configuration
@EnableRedisHttpSession
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // Key는 문자열로 직렬화
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());

        // Value는 JSON으로 직렬화 (객체 저장을 위해)
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

이렇게 변경하니 AgentResDto가 다음과 같은 JSON 형태로 저장되었습니다:

```json
{
  "@class": "com.househub.enums.AgentResDto",
  "id": 12345,
  "status": "ACTIVE"
}
```

### 변경 후 얻은 장점

- 클래스 구조 변경에 유연하게 대응 가능
- 필드 추가/삭제 시에도 기존 데이터 역직렬화 가능
- Redis 데이터를 직접 확인 가능하여 디버깅이 쉬움
- 다른 언어에서도 데이터 접근이 용이

## 마무리

이번 경험을 통해 Redis 직렬화 방식의 중요성을 깊이 이해할 수 있었습니다. JdkSerializationRedisSerializer는 간단하고 직관적이지만, 클래스 버전 변경에 취약합니다. 반면 GenericJackson2JsonRedisSerializer는 약간의 성능 손해를 감수하더라도 더 안정적이고 유연한 데이터 저장이 가능합니다.

프로젝트의 특성과 요구사항에 따라 적절한 직렬화 방식을 선택하는 것이 중요하다는 것을 배웠습니다. 특히 클래스 구조가 자주 변경될 수 있는 프로젝트에서는 GenericJackson2JsonRedisSerializer를 사용하는 것이 더 안전한 선택일 수 있습니다.

이번 트러블슈팅을 통해 얻은 경험을 바탕으로, 앞으로도 Redis를 사용할 때는 직렬화 방식에 대해 더 신중하게 고민하고 선택해야겠다는 생각을 하게 되었습니다.
