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

오늘은 HouseHub 프로젝트를 진행하면서 마주친 Redis 직렬화 문제와 그 해결 과정을 공유하고자 합니다.

## Spring Security와 Redis의 연동

HouseHub 프로젝트에서는 Spring Security와 Redis를 함께 사용하여 사용자 인증을 구현했습니다. 이 조합을 선택한 이유와 구현 방법을 설명드리겠습니다.

### 1. Redis + Spring Security를 선택한 이유

Spring Security는 기본적으로 HttpSession을 사용하여 인증 정보를 관리합니다. Redis를 세션 저장소로 사용하면 다음과 같은 장점이 있습니다:

- 서버 확장성: 여러 서버에서 동일한 세션 정보를 공유할 수 있습니다
- 세션 지속성: 서버를 재시작해도 세션 정보가 유지됩니다
- 빠른 접근: 인메모리 저장으로 인해 세션 조회가 매우 빠릅니다

### 2. 인증 정보 저장

다음은 저희 팀이 문제 발생 이전에 프로젝트에서 사용한 Redis 설정 코드입니다:

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

### 3. 세션 저장 과정

실제로 세션이 어떻게 저장되는지 단계별로 살펴보겠습니다:

1. 사용자가 로그인을 시도합니다
2. Spring Security가 인증을 처리합니다
3. 인증이 성공하면 세션 정보를 Redis에 저장합니다
   - 세션 ID를 키로 사용합니다
   - 인증된 사용자 정보를 값으로 저장합니다
4. 클라이언트에게 세션 ID를 전달합니다 (주로 쿠키로)

### 4. 세션 조회 과정

이제 저장된 세션을 어떻게 조회하는지 알아보겠습니다:

1. 클라이언트가 요청을 보낼 때 세션 ID를 함께 전달합니다
2. Spring Security가 Redis에서 세션 정보를 조회합니다
3. 세션 정보가 유효하면 인증된 요청으로 처리합니다
4. 세션 정보가 없거나 만료된 경우 인증 실패로 처리합니다

## 문제 상황

이렇게 잘 동작하던 시스템에 문제가 발생했습니다. 어느 날 갑자기 로그인과 회원가입이 동작하지 않는 상황이 발생했고, 에러 로그를 확인해보니 다음과 같은 메시지가 있었습니다:

```
org.springframework.data.redis.serializer.SerializationException: Cannot deserialize
```

## 원인 분석

### 1. 기존 직렬화 방식의 문제점

문제의 원인을 파악해보니, 기존에 사용하던 직렬화 방식에 문제가 있었습니다:

- JdkSerializationRedisSerializer를 사용하여 Java 직렬화 방식으로 데이터를 저장하고 있었습니다
- AgentResDto의 serialVersionUID가 클래스 구조 변경 시 자동으로 재계산되어 호환성 문제가 발생했습니다
- Redis에 바이너리 형식으로 저장되어 있어 클래스 버전이 불일치하면 역직렬화가 불가능했습니다

### 2. 문제 발생 과정

문제가 발생한 과정을 자세히 살펴보겠습니다:

1. 리팩토링 과정에서 Agent 도메인의 enum 클래스들을 기존 entity 패키지에서 enums 패키지로 이동했습니다
2. 이로 인해 AgentResDto의 import 문이 변경되면서 serialVersionUID가 자동으로 변경되었습니다
3. 기존에 저장된 Redis 데이터와 새로운 serialVersionUID가 불일치하여 역직렬화에 실패했습니다

## Redis Serializer이 뭔데?

Redis Serializer는 Java 객체를 Redis에 저장할 수 있는 형태로 변환하고, 다시 Java 객체로 복원하는 역할을 합니다. Spring Data Redis는 여러 가지 Serializer를 제공하는데, 각각의 특징을 살펴보겠습니다.

### 1. JdkSerializationRedisSerializer 이란?

- Java의 기본 직렬화 방식을 사용합니다
- `Serializable` 인터페이스를 구현한 클래스만 직렬화 가능합니다
- `serialVersionUID`를 사용하여 클래스 버전을 관리합니다
- 바이너리 형식으로 데이터를 저장합니다
- 장점:
  - Java의 기본 직렬화를 사용하므로 구현이 간단합니다
  - 모든 필드가 자동으로 직렬화됩니다
- 단점:
  - 클래스 구조가 변경되면 역직렬화가 실패할 수 있습니다
  - 데이터 크기가 크고 가독성이 떨어집니다
  - 다른 언어와의 호환성이 없습니다

### 2. GenericJackson2JsonRedisSerializer

- Jackson 라이브러리를 사용하여 JSON 형식으로 직렬화합니다
- 클래스 정보를 JSON에 포함시켜 역직렬화 시 정확한 타입으로 복원합니다
- 장점:
  - 클래스 구조 변경에 유연하게 대응 가능합니다
  - JSON 형식으로 저장되어 가독성이 좋습니다
  - 다른 언어와의 호환성이 좋습니다
- 단점:
  - 직렬화/역직렬화 속도가 JdkSerializationRedisSerializer보다 느립니다
  - 메모리 사용량이 더 많을 수 있습니다

### 3. 다른 Serializer 옵션들

- **StringRedisSerializer**: 문자열만 직렬화할 때 사용
- **Jackson2JsonRedisSerializer**: 특정 클래스에 대한 JSON 직렬화
- **OxmSerializer**: XML 형식으로 직렬화
- **ByteArrayRedisSerializer**: 바이트 배열로 직렬화

## 해결 방안

### 1. 직렬화 방식 변경

저희 팀은 이 문제를 해결하기 위해 직렬화 방식을 변경했습니다:

- JdkSerializationRedisSerializer에서 GenericJackson2JsonRedisSerializer로 변경했습니다
- JSON 기반 직렬화를 통해 클래스 버전 의존성을 제거했습니다
- 객체를 텍스트 기반 JSON으로 변환하여 저장하도록 했습니다

**변경 후 Redis 설정**

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

**변경 전 직렬화 과정**

```java
// JdkSerialization 사용 시
AgentResDto dto = new AgentResDto(...);
byte[] serialized = serializer.serialize(dto);
// 결과: [ac, ed, 00, 05, 77, 22, ...] (불분명한 바이너리)
```

**변경 후 직렬화 결과**

```json
{
  "@class": "com.househub.enums.AgentResDto",
  "id": 12345,
  "status": "ACTIVE"
}
// 직관적인 데이터 구조 확인 가능
```

### 2. 장점

이 방식으로 변경하면서 얻은 장점들은 다음과 같습니다:

- 클래스 구조 변경에 유연하게 대응할 수 있게 되었습니다
- 필드를 추가하거나 삭제해도 기존 데이터를 역직렬화할 수 있습니다
- Redis 데이터를 직접 확인할 수 있어 디버깅이 쉬워졌습니다
- 다른 언어에서도 데이터에 접근하기 쉬워졌습니다

### 3. 메타데이터 처리 방식

Jackson의 타입 정보 저장 기능을 활용했습니다:

- @Class 필드를 통해 실제 클래스 정보를 JSON에 포함시켰습니다
- 역직렬화 시 정확한 타입으로 복원할 수 있게 되었습니다
- 패키지 경로가 변경되어도 유연하게 대응할 수 있습니다

문제 해결을 위해 직렬화 방식을 변경해야 하는 근본적 이유:

| 문제 요소                | JdkSerialization 대응 | Jackson 대응     |
| ------------------------ | --------------------- | ---------------- |
| 클래스 버전 변경         | 실패                  | 성공             |
| 패키지 구조 변경         | 실패                  | 성공             |
| 크로스플랫폼 데이터 공유 | 불가능                | 가능             |
| 데이터 가시성            | HEX 출력              | 가독성 있는 JSON |

## 결론

Redis 직렬화 방식을 JSON 기반으로 변경함으로써:

1. 클래스 버전에 의존하지 않는 안정적인 데이터 저장이 가능해졌습니다
2. 필드 변경에 더 유연하게 대응할 수 있게 되었습니다
3. Redis 데이터의 가독성 및 호환성이 향상되었습니다
