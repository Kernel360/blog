---
layout: post
title: Refresh Token을 Redis에 저장하는 이유
author: 오승민
banner:
  image: 
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [Token, 기술세미나]
---


## Refresh Token을 Redis에 저장하는 이유

### 1. 들어가며
현대 웹 애플리케이션에서는 사용자 인증과 인가가 매우 중요합니다. 특히, 무상태(stateless) 방식의 인증 시스템이 선호됨에 따라 JWT(Json Web Token)가 널리 사용되고 있습니다. 그러나 JWT의 특성상 토큰이 만료되기 전까지는 강제로 무효화할 수 없다는 단점이 존재합니다. 이를 보완하기 위해 많은 시스템이 Refresh Token을 사용하여 사용자 세션을 효율적으로 관리하고 있습니다.

일반적으로 Refresh Token은 지속성이 보장되는 데이터 저장소, 즉 데이터베이스에 저장하는 방식이 흔히 사용됩니다. 하지만 데이터베이스는 동시성 제어와 높은 읽기/쓰기 트래픽 처리에 한계가 있어, 성능과 확장성 면에서 문제가 발생할 수 있습니다.

이러한 문제를 해결하기 위해 Redis와 같은 인메모리 데이터 저장소를 Refresh Token 관리에 활용하는 방법 알아 보겠습니다. Redis가 Refresh Token 저장에 적합한 이유와 함께, Spring Boot를 활용하여 Redis에 Refresh Token을 저장하고 관리하는 방법을 기술적으로 설명하겠습니다.

### 2. 토큰이 왜 필요한가요? JWT는 무엇인가요?

**2.1 토큰의 필요성**

웹 애플리케이션에서는 사용자가 인증을 완료한 이후에도 해당 사용자가 계속해서 자신의 권한을 유지할 수 있어야 합니다. 이를 위해 전통적으로 세션 기반 인증 방식이 사용되었습니다. 하지만 세션 방식은 다음과 같은 한계를 가지고 있습니다.

- 서버에 사용자별 세션 데이터를 저장해야 하므로 서버의 메모리 부담이 커짐
- 확장성이 떨어짐 (서버가 분산 환경일 경우 세션 데이터를 공유하는 데 어려움 발생)

이를 해결하기 위해 무상태(stateless) 방식의 토큰 기반 인증이 등장했습니다. 토큰을 활용하면 클라이언트가 인증 정보를 포함한 토큰을 가지고 다니면서 각 요청에 토큰을 첨부하여 서버로 전달합니다. 서버는 별도의 세션 정보를 저장할 필요 없이 토큰만으로 사용자를 인증하고 인가할 수 있습니다.

**2.2 JWT란?**

JWT(Json Web Token)는 토큰 기반 인증에서 가장 널리 사용되는 형식입니다. JWT는 **Base64 URL-safe 방식**으로 인코딩된 문자열 형태의 토큰이며, 다음과 같은 구조로 구성됩니다.

```text
  헤더(Header).페이로드(Payload).서명(Signature)
```

**[ JWT의 장점과 특징 ]**
- **무상태성** : 서버가 사용자 세션을 관리할 필요가 없기 때문에 확장성과 성능이 향상됩니다.
- **편리한 인증** : 클라이언트가 각 요청에 토큰을 첨부함으로써 인증 상태를 유지할 수 있습니다.
- **변조 방지** : 서명을 통해 토큰이 변조되지 않았음을 서버가 검증할 수 있습니다.

**2.3 JWT의 한계와 Refresh Token의 필요성**

JWT는 무상태성을 보장하지만, 한 번 발급된 토큰은 만료될 때까지 강제로 무효화할 수 없습니다. 따라서 사용자가 로그아웃하거나 토큰이 유출된 경우에도 토큰이 만료되지 않는 한 보안 위협이 존재할 수 있습니다. 이를 해결하기 위해 Refresh Token이 사용됩니다. Refresh Token은 더 긴 유효 기간을 가지고 있으며, 만료된 Access Token을 갱신하는 데 사용됩니다. Refresh Token을 서버에서 안전하게 관리함으로써 보안성과 사용자 경험을 모두 향상시킬 수 있습니다.

### 3. 왜 Redis에 저장해야 할까요?

반드시 Redis에 저장해야하는 것은 아닙니다. RDB에 저장하는 방식과 Redis에 저장하여 관리하는 방식의 차이에 대해 알아보겠습니다.

**3.1 Redis란?**

Redis(Remote Dictionary Server)는 **인메모리(In-memory) 데이터 저장소**로, 높은 성능과 확장성을 제공하는 오픈 소스 소프트웨어입니다. 주로 캐시(Cache), 세션(Session) 관리, 실시간 데이터 처리 등에 활용됩니다. Redis는 데이터베이스처럼 키-값(Key-Value) 구조로 데이터를 저장하며, 다음과 같은 특징을 가지고 있습니다.

- **In-memory 저장**: 모든 데이터를 메모리에 저장하여 매우 빠른 읽기/쓰기 성능을 제공
- **TTL(Time To Live)** 기능 지원: 데이터에 만료 시간을 설정할 수 있음
- **고가용성**: 복제(replication), 클러스터링(clustering)을 통해 분산 환경에서도 안정적으로 동작
- **다양한 데이터 타입 지원**: 문자열(String), 리스트(List), 집합(Set), 해시(Hash) 등


**3.2 RDB와 Redis 저장 방식의 차이**

| 특징               | Redis                                | RDB (관계형 데이터베이스)        |
|--------------------|---------------------------------------|----------------------------------|
| **저장 방식**       | 메모리(In-memory) 기반                | 디스크(Disk) 기반                |
| **읽기/쓰기 속도**  | 밀리초(ms) 단위의 초고속 처리         | 디스크 I/O로 인해 상대적으로 느림 |
| **데이터 만료**     | TTL로 자동 만료 지원                  | 명시적 쿼리를 통해 삭제 필요     |
| **확장성**         | 클러스터링으로 높은 확장성 제공        | 확장 시 복잡한 데이터 동기화 필요 |
| **영속성(Persistence)** | 설정에 따라 메모리 데이터를 디스크에 저장 | 기본적으로 디스크에 영구 저장   |


**3.3 왜 Refresh Token을 Redis에 저장해야 할까요?**

1. **빠른 읽기/쓰기 속도**  
   Redis는 메모리 기반으로 동작하기 때문에 토큰을 저장하거나 조회하는 속도가 매우 빠릅니다. 대규모 사용자 트래픽이 발생하는 인증 시스템에서는 매 요청마다 Refresh Token을 검증해야 하기 때문에 속도가 중요한 요소입니다. 관계형 데이터베이스(RDB)는 디스크 I/O에 의존하여 Redis보다 속도가 느릴 수 있습니다.

2. **TTL을 통한 자동 만료 관리**  
   Redis는 각 키에 TTL(Time To Live)을 설정할 수 있어, 토큰의 유효 기간이 끝나면 자동으로 삭제됩니다. 이를 통해 토큰 만료 관리를 효율적으로 처리할 수 있습니다. 반면, RDB에서는 만료된 토큰을 삭제하기 위해 별도의 배치 작업이나 쿼리가 필요합니다.

3. **확장성과 가용성**  
   Redis는 클러스터링 기능을 통해 분산 환경에서 확장성을 제공하며, 장애 발생 시 빠르게 복구할 수 있습니다. 반면, RDB는 수평 확장 시 데이터 동기화나 트랜잭션 관리가 복잡해질 수 있습니다.

4. **무상태 인증 시스템과의 궁합**  
   Redis는 무상태(stateless) 인증 시스템에서 잘 맞는 솔루션입니다. 서버가 여러 대로 구성된 환경에서는 중앙 집중형 데이터베이스에 의존하는 대신 Redis 클러스터를 사용하여 빠르고 일관된 토큰 관리를 할 수 있습니다.


**3.4 언제 RDB를 사용할까요?**

Redis가 높은 성능과 유연성을 제공하지만, 경우에 따라 RDB가 더 적합할 때도 있습니다.

- **데이터 영속성이 중요한 경우**: Redis는 주로 캐시와 세션 관리에 사용되며, 시스템 장애 시 메모리 데이터가 손실될 수 있습니다. 반면, RDB는 기본적으로 영구 저장을 보장합니다.
- **복잡한 데이터 관계가 필요한 경우**: RDB는 여러 테이블 간의 관계를 정의하고 복잡한 쿼리를 실행할 수 있는 강력한 기능을 제공합니다.



### 4. Spring Boot에서 Redis 사용 방식

Spring Boot에서는 `spring-boot-starter-data-redis`를 통해 Redis와의 통합을 간편하게 지원합니다. Redis는 인메모리 데이터 저장소로서 빠른 읽기/쓰기 속도를 제공하며, Spring Boot와 함께 사용하면 캐시(Cache), 세션(Session), 토큰(Token) 관리 등 다양한 용도로 활용할 수 있습니다. 여기서는 Redis 통합 방법과 주요 사용 방식에 대해 기술적으로 설명하겠습니다.

**4.1 프로젝트 초기 설정 (의존성 추가)**

Redis를 사용하기 위해 프로젝트의 `build.gradle` 또는 `pom.xml`에 의존성을 추가합니다. 저희 프로젝트에서는 Gradle을 사용해서 아래와 같이 의존성을 추가했습니다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

**4.2 Redis 설정**

- Spring Boot에서는 application.yml 또는 application.properties 파일에서 Redis의 연결 정보를 설정할 수 있습니다.

```properties
spring.redis.host=localhost        # Redis 서버 호스트
spring.redis.port=6379             # Redis 포트
```


- Redis의 커넥션 팩토리와 템플릿 설정을 커스터마이즈할 수도 있습니다.
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // Key와 Value에 대한 직렬화 설정
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new StringRedisSerializer());
        
        return template;
    }
}

```


**4.3 RedisTemplate을 활용한 데이터 조작**

Spring Boot의 RedisTemplate을 사용하면 Redis에 데이터를 저장하고 조회하는 작업을 쉽게 수행할 수 있습니다.

```java
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import java.util.concurrent.TimeUnit;

@Service
public class RedisService {

    private final RedisTemplate<String, Object> redisTemplate;

    public RedisService(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    // 데이터 저장 (TTL 설정 포함)
    public void saveData(String key, String value, long durationInSeconds) {
        redisTemplate.opsForValue().set(key, value, durationInSeconds, TimeUnit.SECONDS);
    }

    // 데이터 조회
    public String getData(String key) {
        return (String) redisTemplate.opsForValue().get(key);
    }

    // 데이터 삭제
    public void deleteData(String key) {
        redisTemplate.delete(key);
    }
}

```


**4.4 Spring Data Redis 리포지토리 활용**

Spring Data Redis는 엔티티 기반으로 데이터를 처리할 수 있는 리포지토리 인터페이스를 제공합니다.

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.redis.core.RedisHash;

@RedisHash(value = "example_entity", timeToLive = 3600)  // TTL 3600초 설정
public class ExampleEntity {

    @Id
    private String id;
    private String data;

    // 기본 생성자 및 getter/setter
    public ExampleEntity() {}

    public ExampleEntity(String id, String data) {
        this.id = id;
        this.data = data;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }
}

```

```java
import org.springframework.data.repository.CrudRepository;

public interface ExampleEntityRepository extends CrudRepository<ExampleEntity, String> {
    // 기본 CRUD 메서드 사용 가능
}
```

### 5. 요약

Refresh Token은 빠른 읽기/쓰기 성능과 효율적인 만료 관리가 중요한 인증 요소입니다. Redis는 인메모리 데이터 저장소로, 이러한 요구사항에 최적화되어 있습니다.

결론적으로, Redis는 인증 시스템에서 Refresh Token을 빠르고 안정적으로 관리하기 위한 최적의 솔루션입니다.
