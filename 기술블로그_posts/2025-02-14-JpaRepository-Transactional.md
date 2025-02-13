
---
layout: post  
title: "Jpa의 트랜잭션 처리가 @Transactional 없이도 이루어지는 이유 feat. Proxy"
author: "김대현"
categories: "기술블로그"
banner:
  image: "https://github.com/Kernel360/blog-image/blob/main/2025/jpa.png"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`Spring`, `JPA`, `Transactional`, `Proxy`, `ORM`]
---

Spring Data JPA를 사용할 때, `@Transactional`을 직접 선언하지 않아도 트랜잭션이 자동으로 적용되는 것을 확인할 수 있다. 어떻게 자동으로 적용되는 것일까? 그리고 이 과정에서 프록시는 어떤 역할을 할까? 
이번 글에서는 Spring Data JPA의 트랜잭션 관리 원리를 프록시(proxy)와 관계지어 알아보자.

---

# 1. JPA Respository에서 @Transactional 없이도 트랜잭션이 동작하는 이유
Spring Data JPA에서 JpaRepository를 상속받으면 기본적으로 제공되는 CRUD 메서드(`save()`, `findById()`, `delete()` 등등)는 트랜잭션이 자동으로 적용된다. 우리가 직접 `@Transactional`을 선언하지 않아도 트랜잭션이 동작하는 이유는 무엇일까?
이를 이해하려면 JpaRepository가 상속하는 인터페이스 계층도부터 살펴볼 필요가 있다.

### 1.1 JpaRepository 계층 구조

<img src="https://velog.velcdn.com/images/kdh10806/post/c4b34ca5-234f-4a2a-a857-d0475e5cfb2c/image.png" width="500">

Spring Data JPA의 JpaRespository는 여러 개의 인터페이스를 계층적으로 상속 받는다.
이 인터페이스들은 **추상적인 정의만 제공**할뿐 실제 구현체는 없다.

그럼 구현체도 없는데 어떻게 트랜잭션이 적용된 것인가? 라는 의문을 가질 수 있는데
JPA의 트랜잭션 처리가 `@Transactional`없이도 동작하는 이유는 Spring Data JPA가 제공하는 **기본 구현체** 덕분이다.


### 1.2 SimpleJpaRepository 내부의 트랜잭션 적용
SimpleJpaRepository는 **Spring Data JPA에서 제공**하는 기본 JPA 리포지토리 **구현체**이다.
즉, JpaRepository 인터페이스를 구현하는 기본 클래스이며, 개발자가 직접 구현하지 않아도 기본적인 **CRUD** 기능을 자동으로 제공해 줍니다.

```Java
 @Transactional
    public <S extends T> S save(S entity) {
        Assert.notNull(entity, "Entity must not be null");
        if (this.entityInformation.isNew(entity)) {
            this.entityManager.persist(entity);
            return entity;
        } else {
            return this.entityManager.merge(entity);
        }
    }
```

SimpleJpaRepository의 `save()` 메서드를 보면 다음과 같이 `@Transactional`이 선언되어 있다.
즉, **Spring이 SimpleJpaRepository를 자동으로 구현체로 사용(주입)하면서 트랜잭션을 적용해준다.**

### 1.3 SimpleJpaRepository의 주요 특징
1. 기본 CRUD 기능 자동 제공
	• 별도의 구현 없이 JpaRepository를 상속하는 것만으로 기본적인 CRUD 기능을 사용할 수 있다.
2. EntityManager를 사용하여 JPA 쿼리 실행
	• 내부적으로 JPA의 EntityManager 를 사용하여 데이터를 조회하고 저장한다.
3. 트랜잭션 관리 지원
	• Spring의 `@Transactional`을 활용하여 쓰기 연산은 트랜잭션을 적용하고, 읽기 연산은 기본적으로 트랜잭션 없이 동작한다.(읽기 전용 트랜잭션을 적용하려면 `@Transactional(readOnly=true)`필요)
4. 동적 쿼리 지원 (Specification)
	•	JpaSpecificationExecutor를 함께 사용하면, 동적으로 조건을 조합하여 검색할 수 있다.

# 2. 프록시와 트랜잭션의 관계
Spring의 `@Transactional`이 제대로 동작하려면 프록시(proxy)가 필요하다. 
Spring은 **AOP(Aspect-Oriented Programming)** 기법을 활용하여 프록시를 생성하고 트랜잭션을 관리하기 때문이다.

### 2.1 프록시 기반 트랜잭션 동작 과정
1. 프록시 객체가 생성된다. (Spring이 자동으로 JpaRepository 구현체를 감싼다)

> ✅ **참고**
> **JpaRepository가 프록시로 감싸지는 과정**
> 1. JpaRepository를 상속받은 인터페이스를 만들면 Spring이 이를 감지한다.
> 2. Spring은 이 인터페이스의 실제 구현체(SimpleJpaRepository)를 생성한다.
> 3. 이 구현체를 프록시로 감싼다.
> 4. 프록시는 `@Transactional`이 적용된 메서드를 호출할 때, 트랜잭션을 시작하거나 종료하는 역할을 한다.

2. 트랜잭션이 필요한 메서드를 호출하면 프록시가 먼저 실행된다.
3. 프록시가 트랜잭션을 시작한다. (begin)
4. 원래 메서드를 실행한다.
5. 정상 종료되면 트랜잭션을 커밋(`commit`), 예외가 발생하면 롤백(`rollback`) 한다.

>
✅ **참고** 
**만약 프록시 없이 @Transactional을 적용한다면?**
메서드 호출이 원본 객체로 바로 전달되므로 트랜잭션이 적용되지 않는다.
예외 발생 시 자동 롤백이 불가능하다.
여러 DB 연산이 하나의 트랜잭션으로 묶이지 않는다.



 
