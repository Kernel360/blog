---
layout: post  
title: "AOP를 활용한 로깅 처리"
author: "정소현"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["기술블로그"]
---

# AOP를 활용한 로깅 처리

## 개요
서비스 규모가 커질수록 로그 기록, 인증 처리, 예외 처리처럼 반복되는 코드가 여기저기 중복됩니다.
이런 공통 로직을 매번 메서드마다 작성하다 보면 코드가 지저분해지고 유지보수도 어려워집니다. 

이런 중복 코드들을 **AOP (Aspect-Oriented Programming)** 를 사용하여 관리할 수 있습니다.
AOP는 핵심 비즈니스 로직은 그대로 두고, 반복되는 부가 로직은 따로 분리해서 깔끔하게 관리할 수 있게 도와줍니다.

따라서 **Spring Framework에서 AOP를 활용해 로깅을 처리하는 방법**을 예제로 설명하겠습니다.

## AOP란?

AOP는 **핵심 비즈니스 로직과 공통 관심 사항을 분리**하는 프로그래밍 패러다임입니다.

- **핵심 로직**: 주문 처리, 사용자 정보 수정 등  
- **공통 관심 사항**: 로깅, 트랜잭션, 보안 등

공통 관심 사항을 Aspect로 분리함으로써 코드의 **가독성**과 **유지보수성**을 높일 수 있습니다.


## 언제 로깅에 AOP를 쓸까?

- 메서드 진입/종료 로그
- 실행 시간 측정
- 예외 발생 감지

이런 부분은 모든 서비스 메서드에 반복되기 때문에 AOP로 처리하는 것이 효과적입니다.


## 예제: Spring AOP를 활용한 로깅

### 1. 의존성 추가 (Gradle 기준)

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-aop'
}
```

### 2. 로깅 Aspect 작성
```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    @Pointcut("execution(* com.example.service..*(..))")
    public void serviceMethods() {}

    @Around("serviceMethods()")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().toShortString();
        long start = System.currentTimeMillis();

        logger.info(">> Entering {}", methodName);
        try {
            Object result = joinPoint.proceed();
            long elapsedTime = System.currentTimeMillis() - start;
            logger.info("<< Exiting {} ({} ms)", methodName, elapsedTime);
            return result;
        } catch (Throwable ex) {
            logger.error("!! Exception in {}: {}", methodName, ex.getMessage());
            throw ex;
        }
    }
}
```
#### 코드 설명

| 어노테이션 / 메서드          | 설명 |
|------------------------------|------|
| `@Aspect`                    | AOP 클래스를 정의함 |
| `@Pointcut`                  | 로깅할 메서드 범위 지정 (예: `com.example.service` 패키지 이하) |
| `@Around`                    | 메서드 실행 전후를 감싸서 처리 |
| `joinPoint.proceed()`       | 실제 비즈니스 로직 호출 |


### 3. 로그 예시
```scss
>> Entering MyService.doSomething()
<< Exiting MyService.doSomething() (23 ms)
```
```diff
!! Exception in MyService.doSomething(): NullPointerException
```
