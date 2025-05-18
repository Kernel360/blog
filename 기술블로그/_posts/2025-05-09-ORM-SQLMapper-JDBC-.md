---
layout: post
title: "ORM, SQLMapper, JDBC"
author: "김찬호"
categories: "기술블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["Java"]
---

## 주제: JDBC, ORM, SQLMapper

---

### 1. JPA란 무엇인가요?

객체 지향 프로그램에서 객체의 상태는 메모리에만 존재하며 프로그램 종료 시 사라집니다.
이 상태를 영속화(persistence)하려면 JDBC, SQLMapper 등을 사용해야 합니다.
그렇다면 JPA는 정확히 무엇일까요?

> **JPA** (Java Persistence API)
> 자바 객체와 데이터베이스 사이의 ORM(Object-Relational Mapping)을 지원하는 **표준 인터페이스**입니다.
> 쉽게 말해, "자바 객체를 데이터베이스 테이블처럼 다룰 수 있게 해주는 기술"입니다.

---

### 2. JDBC란?

**JDBC**(Java Database Connectivity)는 자바에서 데이터베이스에 접속하고 SQL을 실행하기 위한 표준 API입니다.
구성은 크게 4계층으로 나뉩니다:

| 계층                  | 역할                                         |
| ------------------- | ------------------------------------------ |
| Java Application    | 데이터 소스와 통신하는 Java 애플리케이션 (Applet, Servlet) |
| JDBC API            | 데이터베이스와 통신할 수 있는 인터페이스 제공                  |
| JDBC Driver Manager | 드라이버 로딩 및 커넥션 관리                           |
| JDBC Driver         | DBMS별 프로토콜로 SQL 실행                         |

#### JDBC API 주요 인터페이스

* **DriverManager**: 등록된 드라이버 목록 관리
* **Connection**: DB 연결 및 트랜잭션 관리
* **Statement**: SQL 문 실행
* **ResultSet**: 쿼리 결과 보관 및 반복 조회

#### JDBC 장점

1. Connection, Statement, ResultSet에 대한 완전한 제어권
2. 표준 API로 대부분의 DBMS 지원

#### JDBC 단점

1. SQL 문을 직접 작성해야 함
2. 중복 SQL 작성
3. 커넥션 관리·예외 처리 등 반복 코드 많아 생산성 저하

---

### 3. ORM이란?

**ORM**(Object-Relational Mapping)은 객체와 관계형 데이터베이스를 자동 매핑해 주는 기술입니다.
대표적인 구현체로 **Hibernate**, **JPA** 등이 있으며, Spring Data JPA는 JPA를 더욱 쉽게 사용할 수 있도록 도와주는 모듈입니다.

#### ORM 동작 예시

```java
// 엔티티 객체 전달만으로 INSERT 수행
entityManager.persist(member);
```

#### ORM 장점

1. **생산성 향상**: SQL을 직접 작성하지 않아도 되고, 비즈니스 로직에 집중 가능
2. **유지보수성**: 매핑 설정만으로 테이블 스키마 변경에 유연하게 대응
3. **객체 지향적 코드**: 도메인 모델 그대로 조회·저장·갱신
4. **부가 기능**: 캐시, 지연 로딩, 변경 감지 등 제공

#### ORM 단점

1. **학습 곡선**: 엔티티 매핑, 영속성 컨텍스트, JPQL 등의 개념 이해 필요
2. **복잡한 쿼리 튜닝**: JPQL/Criteria API로 작성한 복합 쿼리 최적화 필요
3. **오버헤드**: 자동 생성 SQL로 인한 불필요한 쿼리 발생 가능

---

### 4. SQLMapper이란?

**SQLMapper**는 SQL 문을 직접 작성하면서도 객체 매핑을 지원하는 기술입니다.
대표적인 프레임워크로 **MyBatis**가 있습니다.

#### SQLMapper 사용 방법

1. XML 파일에 SQL 문 정의
2. 자바 인터페이스(Mapper)와 매핑하여 실행 결과를 객체로 반환

#### SQLMapper 장점

1. **직접 제어 가능**: 복잡한 SQL 및 DBMS 고유 기능 사용에 제약 없음
2. **명시적 SQL 관리**: 작성한 SQL이 그대로 실행되어 튜닝·분석 용이
3. **경량 매핑**: 필요한 필드만 매핑하여 오버헤드 최소화

#### SQLMapper 단점

1. **DBMS 종속성**: SQL이 DBMS 벤더에 종속적일 수 있어 마이그레이션 시 부담
2. **반복 코드**: SQL 구문·파라미터 매핑 설정을 수작업으로 작성해야 함
3. **유지보수 이슈**: SQL이 분산된 XML 파일에 존재하여 프로젝트 규모 커지면 관리 복잡

---

### 5. 왜 ORM이나 SQLMapper를 써야 할까?

과거에는 개발자들이 모든 SQL을 직접 작성하여 생산성이 낮고 에러도 잦았습니다.
ORM/SQLMapper 도입으로:

* **비즈니스 로직**에 집중 가능
* **객체 지향** 장점 활용
* **생산성** 대폭 향상

특히 JPA는 SQL 축약뿐 아니라 영속성 관리, 캐싱, 변경 감지 등의 기능을 제공합니다.

---

### 6. JPA vs Hibernate vs Spring Data JPA

| 구분                  | 설명                              |
| ------------------- | ------------------------------- |
| **JPA**             | 자바 ORM의 **표준 인터페이스**            |
| **Hibernate**       | JPA를 구현한 대표 **라이브러리**           |
| **Spring Data JPA** | JPA를 쉽게 사용할 수 있도록 감싼 **추상화 도구** |

> Spring Data JPA → Hibernate → JPA 순으로 감싸고 있다고 이해하면 쉽습니다.

#### 사용 예시

```java
// Spring Data JPA
interface MemberRepository extends JpaRepository<Member, Long> {}

// JPA 직접 사용
EntityManager em = entityManagerFactory.createEntityManager();
em.persist(member);
```

JPA는 내부적으로 `EntityManagerFactory`, `EntityManager`, `EntityTransaction`으로 엔티티를 관리합니다.
