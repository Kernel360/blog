---
layout: post  
title: "트랜잭션 완전 정복: DB부터 Spring까지"  
author: "권승목"  
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["transaction"]
---

# 트랜잭션이란?

트랜잭션(Transaction)은 데이터베이스에서 **하나의 논리적 작업 단위**를 말합니다. 여러 개의 쿼리가 묶여 하나의 작업처럼 실행되며, 모두 성공하거나 하나라도 실패하면 전체가 취소되어야 하는 특성을 가집니다. 대표적인 예로 **은행 이체**가 있습니다.

트랜잭션의 기본 목적은 데이터의 **일관성 유지**입니다.

---

# ACID란?

트랜잭션이 만족해야 할 네 가지 특성입니다.

- **Atomicity (원자성)**  
  모든 작업은 전부 성공하거나 전부 실패해야 한다.

- **Consistency (일관성)**  
  트랜잭션 전후의 데이터 상태는 항상 일관돼야 한다.

- **Isolation (격리성)**  
  여러 트랜잭션이 동시에 실행돼도 각각이 영향을 받지 않아야 한다.

- **Durability (지속성)**  
  커밋된 트랜잭션의 결과는 시스템 장애가 발생해도 보존돼야 한다.

---

# Dirty Read, Non-Repeatable Read, Phantom Read

트랜잭션 격리성 문제를 설명할 때 자주 나오는 3가지 현상입니다.

- **Dirty Read**  
  다른 트랜잭션이 아직 커밋하지 않은 데이터를 읽는 경우

- **Non-Repeatable Read**  
  같은 쿼리를 두 번 실행했는데 결과가 달라지는 경우 (다른 트랜잭션이 데이터를 수정함)

- **Phantom Read**  
  같은 조건의 `SELECT`에서 결과 행 수가 달라지는 경우 (다른 트랜잭션이 행을 추가/삭제함)

---

# 트랜잭션의 격리 수준 (Isolation Level)

| 수준 | 설명 | 현상 방지 |
|------|------|------------|
| **READ UNCOMMITTED** | 커밋 전 데이터도 읽을 수 있음 | 없음 |
| **READ COMMITTED** | 커밋된 데이터만 읽음 | Dirty Read 방지 |
| **REPEATABLE READ** | 같은 쿼리는 항상 같은 결과 | Non-Repeatable Read 방지 |
| **SERIALIZABLE** | 완벽한 격리 | Phantom Read 방지 |

MySQL은 기본적으로 `REPEATABLE READ`, PostgreSQL은 `READ COMMITTED`가 기본입니다.

---

# 스프링 트랜잭션

Spring은 선언적 트랜잭션 관리를 제공합니다.  
주로 `@Transactional` 어노테이션을 사용하며, 아래와 같은 방식으로 적용됩니다:

```java
@Transactional
public void doSomething() {
    // DB 작업
}
