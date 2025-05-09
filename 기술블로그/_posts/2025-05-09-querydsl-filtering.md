---
layout: post  
title: "QueryDSL을 활용한 조건 기반 검색 기능 구현기"
author: "조수연"
categories: "기술블로그"
banner:
  image: ""
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`QueryDSL`, `검색 기능`, `조건 검색`, `Spring`, `JPA`]
---

## 🔍 들어가며

프로젝트를 개발하면서 단순한 전체 조회가 아닌, 사용자의 입력에 따라 동적으로 조건을 조합해 검색하는 기능이 필요했습니다.  
이 글에서는 **QueryDSL을 활용하여 복잡한 조건 기반 검색 기능을 어떻게 구현했는지**에 대해 설명합니다.  
BooleanBuilder와 Pageable, 정렬, 조건 조합에 대한 실용적인 내용을 담고 있습니다.

---

## 🧩 QueryDSL이란?

QueryDSL은 Java 코드로 SQL과 유사한 타입 안전한 쿼리를 작성할 수 있게 도와주는 라이브러리입니다.  
기존 JPA의 Criteria API보다 훨씬 간결하고 가독성이 높으며, IDE의 자동 완성과 컴파일 타임 에러 방지 등의 장점이 있습니다.

---

## 🧱 사용한 기술 스택

- Spring Boot
- Spring Data JPA
- QueryDSL
- MariaDB (RDBMS)
- Kotlin 또는 Java (언어에 따라 유동적)

---

## 🛠️ 구현 목표

- 사용자 선택에 따라 동적으로 조건을 조합
- 검색 조건: 카테고리, 금액 범위, 주소, 계약 상태 등
- 페이징 처리
- 정렬 방식 선택 (최신순, 오래된순, 임박순 등)

---

## ✅ QueryDSL 설정

1. 의존성 추가 (`build.gradle`)

```groovy
implementation "com.querydsl:querydsl-jpa"
annotationProcessor "com.querydsl:querydsl-apt"
```

2. Q클래스 자동 생성 설정

```groovy
tasks.withType(JavaCompile).configureEach {
    options.annotationProcessorPath = configurations.annotationProcessor
}
```

---

## 📦 검색 조건 DTO 설계

```java
public class ContractFilterRequestDTO {
    private String category;
    private String status;
    private String address;
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
    private LocalDate startDateFrom;
    private LocalDate startDateTo;
}
```
---

## 🧠 BooleanBuilder로 조건 조립하기

QueryDSL에서는 `BooleanBuilder`를 사용해 조건을 동적으로 조립할 수 있습니다.  
사용자가 입력한 검색 조건이 존재할 때만 해당 조건을 `and()`로 추가하는 방식입니다.

```java
BooleanBuilder builder = new BooleanBuilder();

if (dto.getCategory() != null) {
  builder.and(contract.category.eq(dto.getCategory()));
  }

  if (dto.getMinPrice() != null && dto.getMaxPrice() != null) {
  builder.and(contract.price.between(dto.getMinPrice(), dto.getMaxPrice()));
  }

  if (dto.getStatus() != null) {
  builder.and(contract.status.eq(dto.getStatus()));
  }
  ```

이런 식으로 조건을 동적으로 조립하면, 모든 조건이 필수가 아닌 선택적 조건이어도 유연하게 대응할 수 있습니다.

---
## 🧭 정렬 처리

사용자 요청에 따라 정렬 기준을 변경할 수 있도록 QueryDSL에서도 `.orderBy()`를 활용합니다.  
예를 들어 최신순, 오래된순, 종료일 임박순 등을 지원하려면 다음과 같이 작성할 수 있습니다.

```java
if ("latest".equals(sort)) {
  query.orderBy(contract.createdAt.desc());
  } else if ("expiredSoon".equals(sort)) {
  query.orderBy(contract.contractEndDate.asc());
  }
  ```
정렬 기준은 프론트엔드에서 쿼리 파라미터로 전달받아 처리하면 깔끔하게 분리할 수 있습니다.

---

## 📄 결과 반환

```java
List<Contract> results = query
    .where(builder)
    .offset(pageable.getOffset())
    .limit(pageable.getPageSize())
    .fetch();
```

---

## 📌 마무리

QueryDSL은 복잡한 검색 조건을 유연하게 처리할 수 있도록 도와주는 강력한 도구입니다.  
BooleanBuilder와 동적 정렬을 적절히 활용하면 사용자 맞춤형 검색 기능을 쉽게 구현할 수 있습니다.  
실제 개발 환경에서도 다양한 조합 조건을 적용하면서 QueryDSL의 장점을 체감할 수 있었습니다.

추후에는 Predicate 방식이나 FetchJoin, 서브쿼리 등의 고급 QueryDSL 기능도 함께 적용해볼 계획입니다.
