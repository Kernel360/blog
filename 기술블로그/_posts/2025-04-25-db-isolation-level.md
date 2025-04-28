---
layout: post  
title: "데이터베이스 격리 수준"
author: "신윤상"
categories: "기술세미나"
banner:
  image: 
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`DataBase`]
---

# 데이터베이스 격리 수준 (Isolation Level)

## 격리 수준이란?
> 여러 트랜잭션이 동시에 처리될 때, 각 트랜잭션이 다른 트랜잭션과 **얼마나 고립되어 있는지**를 나타내는 개념입니다.  
> 즉, 어떤 트랜잭션이 다른 트랜잭션의 **변경되거나 조회 중인 데이터를 볼 수 있도록 허용할지**를 결정합니다.  
---

## 대표적인 4가지 격리 수준

| 격리 수준         | 설명                                         |
|------------------|---------------------------------------------|
| READ UNCOMMITTED | 커밋되지 않은 데이터도 읽을 수 있음           |
| READ COMMITTED   | 커밋된 데이터만 읽을 수 있음                 |
| REPEATABLE READ  | 같은 트랜잭션 내에서 동일한 쿼리 결과를 보장 |
| SERIALIZABLE     | 모든 트랜잭션을 순차적으로 실행하는 수준      |

- 위로 갈수록 동시 처리 성능은 **좋아지나 정합성은 낮아지고**,
- 아래로 갈수록 정합성은 **좋아지나 성능은 낮아집니다**.
- 일반적인 서비스에서는 보통 다음을 사용합니다:
  - **Oracle**: `READ COMMITTED`
  - **MySQL (InnoDB)**: `REPEATABLE READ`
---

## 격리 수준에 따른 부정합 (Read Anomaly)

| 격리 수준         | Dirty Read | Non-repeatable Read | Phantom Read |
|------------------|------------|----------------------|---------------|
| READ UNCOMMITTED | ✅         | ✅                   | ✅            |
| READ COMMITTED   | ❌         | ✅                   | ✅            |
| REPEATABLE READ  | ❌         | ❌                   | ✅ *(InnoDB는 ❌)* |
| SERIALIZABLE     | ❌         | ❌                   | ❌            |

---

## 격리 수준 상세 설명

### 1. READ UNCOMMITTED
> **가장 낮은 수준의 격리 수준**  
커밋 여부와 관계없이, **다른 트랜잭션의 변경 내용이 보이는** 가장 낮은 수준의 격리입니다.
실제로는 거의 사용되지 않으며, 인정받지 않는 수준입니다


#### 예시
![READ UNCOMMITTED](https://github.com/Kernel360/blog-image/blob/main/2025/0425/READ%20UNCOMMITTED.png?raw=true)

1. 사용자 A는 `emp_no = 5000000`, `first_name = 'Lara'`인 사원을 INSERT합니다.
2. 사용자 B는 A가 아직 커밋하지 않았음에도 해당 사원의 데이터를 조회합니다.
3. 이후 A가 ROLLBACK을 하더라도, B는 이미 데이터를 조회했기 때문에 이를 기반으로 잘못된 작업을 이어갈 수 있습니다.  

이러한 현상을 **Dirty Read**라고 합니다.
### Dirty Read

- **커밋되지 않은 데이터를 다른 트랜잭션이 읽는 현상**입니다.
- 데이터가 나타났다가 사라지는 등 **정합성 문제가 심각**하게 발생할 수 있습니다.

---

## 2. READ COMMITTED

> **커밋된 데이터만** 다른 트랜잭션에서 읽을 수 있도록 허용하는 격리 수준입니다.  
> Oracle의 기본 격리 수준이며, 온라인 서비스에서 자주 사용됩니다.
#### 예시
![READ COMMITED](https://github.com/Kernel360/blog-image/blob/main/2025/0425/READ%20COMMITED.png?raw=true)
1. 사용자 A는 `emp_no = 500000`, `first_name = 'Lara'`인 사원의 이름을 `Toto`로 변경합니다.
2. 변경된 `Toto`는 즉시 employees 테이블에 반영되며, 기존 값인 `Lara`는 **Undo 영역**에 백업됩니다.
3. A가 아직 커밋하지 않은 상태에서, 사용자 B가 해당 사원을 조회하면 `Toto`가 아닌 `Lara`가 조회됩니다.  

이는 employees 테이블이 아닌 Undo 영역에서 데이터를 가져오기 때문입니다.

### Undo 영역

> `READ COMMITTED` 격리 수준에서 중요한 개념 중 하나는 바로 **Undo 영역**입니다.

- Undo 영역은 **UPDATE**나 **DELETE** 명령으로 변경되기 전의 데이터를 저장하는 공간입니다.
- **INSERT**의 경우, 새로 추가된 데이터의 **row ID**를 저장하여 메모리에 직접 접근할 수 있도록 합니다.
- Undo 영역은 크게 두 가지 용도로 사용합니다:
  1. 트랜잭션이 실패했을 때 **롤백(rollback)** 처리를 가능하게 함
  2. **격리 수준을 유지**하면서도 **높은 동시성(concurrency)**을 제공

즉, 어떤 트랜잭션에서 변경된 내용은 커밋되기 전까지는 **다른 트랜잭션에서 직접 조회할 수 없으며**,  
다른 트랜잭션은 **Undo 영역에 저장된 이전 값을 기준으로 데이터를 조회**하게 됩니다.

사용자 A가 변경한 내용을 커밋하면, 그제서야 다른 트랜잭션에서도 `Toto` 값을 조회할 수 있게 됩니다.
> 이러한 구조 덕분에 `READ COMMITTED`는 **Dirty Read는 방지**되지만,  
> **동일한 트랜잭션 내에서 반복 조회 결과가 달라지는 Non-repeatable Read 현상은 여전히 발생**할 수 있습니다.

### NON-REPEATABLE READ

> **하나의 트랜잭션 내에서 동일한 SELECT 쿼리를 두 번 실행했을 때, 서로 다른 결과가 반환되는 현상**입니다.

이 현상은 **트랜잭션이 실행되는 동안 다른 트랜잭션이 데이터를 변경(UPDATE 또는 DELETE)**함으로써 발생합니다.

즉, **같은 키 값을 가진 Row를 반복해서 조회했지만, 그 사이에 값이 변경 또는 삭제되어 결과가 달라지는 경우**를 말합니다.

#### 예시
![NON-REPEATABLE READ](https://github.com/Kernel360/blog-image/blob/main/2025/0425/NON-REPEATABLE%20READ.png?raw=true)

1. 사용자 B가 트랜잭션을 시작한 후, `first_name = 'Toto'`인 사원을 조회 → 결과 없음
2. 사용자 A가 `emp_no = 500000`인 사원의 `first_name`을 `'Lara'` → `'Toto'`로 수정하고 커밋
3. 사용자 B가 같은 조건으로 다시 조회하면 → `Toto`인 사원이 1건 조회됨

1. 사용자 B는 트랜잭션을 시작한 뒤, `first_name = 'Toto'`인 사원을 조회합니다.  
   → 이때는 일치하는 데이터가 **없습니다.**

2. 사용자 A는 `emp_no = 500000`인 사원의 `first_name`을 `'Lara'`에서 `'Toto'`로 수정하고 커밋합니다.

3. 사용자 B가 같은 조건(`first_name = 'Toto'`)으로 다시 조회합니다.  
   → 이번에는 해당 조건을 만족하는 **사원 1건이 조회됩니다.**

> 동일한 SELECT 쿼리를 두 번 실행했지만, 그 사이에 데이터가 변경되어 결과가 달라진 것입니다.  
**트랜잭션 내에서는 항상 동일한 조회 결과를 보장해야 한다**는 `REPEATABLE READ`의 정합성 조건에 어긋납니다.   

---

## 3. REPEATABLE READ
> **같은 트랜잭션 내에서는 항상 동일한 쿼리 결과를 보장**하는 격리 수준입니다.  
> MySQL InnoDB의 기본 격리 수준입니다.

![REPEATABLE READ](https://github.com/Kernel360/blog-image/blob/main/2025/0425/REPEATABLE%20READ.png?raw=true)
### 예시
- 기존에 `INSERT`되어 있던 데이터의 트랜잭션 ID는 `6`이라고 가정합니다.

- 사용자 A는 `emp_no = 500000`인 사원의 이름을 `'Lara'`에서 `'Toto'`로 변경하려 합니다.
- 이 시점에 사용자 B도 동일한 사원을 조회하게 됩니다.

- **사용자 A의 트랜잭션 ID: 12**
- **사용자 B의 트랜잭션 ID: 10**

---

1. 사용자 A가 `emp_no = 500000` 사원의 이름을 `'Toto'`로 변경한 후 커밋합니다.

2. 사용자 B는 해당 사원을 **변경 전(A의 커밋 전)과 변경 후에 각각 한 번씩 SELECT**하지만,  
   **항상 'Lara' 값을 조회합니다.**

이는 `REPEATABLE READ`에서 **트랜잭션이 시작된 시점(ID 10)** 보다 **늦게 생성된 변경 사항(ID 12)** 은 B의 트랜잭션에서는 **조회되지 않기 때문**입니다.  
즉, **사용자 B가 부여받은 10번 트랜잭션 안에서 실행되는 모든 SELECT 쿼리는, 트랜잭션 번호가 10보다 작은(=이전 시점의) 변경 내용만 볼 수 있습니다.**  
> 단, `REPEATABLE READ`에서도 **PHANTOM READ**는 여전히 발생할 수 있습니다.
> 
### PHANTOM READ

> **트랜잭션 내에서 동일한 조건의 SELECT 쿼리를 반복 실행했음에도, 조회되는 행(row)의 개수가 달라지는 현상**을 말합니다.  
> 이는 주로 **다른 트랜잭션이 새로운 행을 INSERT하거나 DELETE할 때 발생**합니다.

#### 📌 예시

![PHANTOM READ](https://github.com/Kernel360/blog-image/blob/main/2025/0425/PHANTOM_READ.png?raw=true)

1. Transaction 2가 트랜잭션을 시작하고, 특정 조건(e.g. `age > 30`)으로 SELECT를 수행 → 2개의 row 조회
2. 동시에 Transaction 1이 트랜잭션을 시작하고, 같은 조건에 해당하는 새로운 row를 INSERT 후 커밋
3. Transaction 2가 동일한 조건으로 다시 SELECT → 이번엔 3개의 row 조회

> 이처럼 트랜잭션 내부에서 **보였다가 안 보였다가 하는 레코드 변경 현상**을 **Phantom Read**라고 합니다.  
`REPEATABLE READ` 격리 수준에서도 **단일 레코드에 대한 일관성은 보장**되지만,  
**범위 조건을 사용하는 SELECT**에서는 **새로 삽입된 행이 결과에 포함될 수 있어** Phantom Read가 발생합니다.

#### Gap Lock과 해결 방안

MySQL의 InnoDB 스토리지 엔진은 `REPEATABLE READ` 상태에서도 **Gap Lock**을 사용해  
일정 범위 내의 **삽입을 차단**함으로써 Phantom Read를 방지할 수 있습니다.

하지만 일반적인 SELECT는 락을 걸지 않기 때문에, 아래와 같은 방법이 필요합니다:

- `SELECT ... FOR UPDATE`
- `SELECT ... LOCK IN SHARE MODE`  

이와 같은 쿼리는 실제 레코드(현재 버전)를 기준으로 읽으며,
**Undo 영역에서 이전 버전을 참조하는 일반 SELECT와는 다르게 동작**합니다.

→ 즉, **Undo 영역에서는 락이 불가능하기 때문에**, 위와 같은 명시적 락 기반 SELECT를 사용해야만 **정합성을 확보**할 수 있습니다.

---

## 4. SERIALIZABLE

> **가장 엄격하고 정합성이 뛰어난 격리 수준**입니다.  
> 모든 트랜잭션을 마치 **순차적으로(Serial)** 실행하는 것처럼 동작하도록 강제합니다.


#### 특징

- 트랜잭션 간 **완전한 고립(Isolation)** 을 보장합니다.
- Dirty Read, Non-repeatable Read, Phantom Read 등 **모든 부정합이 발생하지 않습니다.**
- 동시에 동일한 레코드에 접근하는 트랜잭션은 **충돌하거나 대기하거나 실패**합니다.
- 높은 정합성을 제공하지만, 그만큼 **동시 처리 성능은 가장 낮습니다.**

#### 동작 방식

기본적으로 SELECT 문은 다른 격리 수준에서는 **레코드 잠금을 걸지 않고** 실행됩니다.  
하지만 `SERIALIZABLE`에서는 **단순한 SELECT조차도 공유 잠금(읽기 잠금)을 획득**합니다.

- SELECT가 공유 잠금을 획득한 경우  
  → 다른 트랜잭션은 해당 레코드를 **수정(쓰기 잠금)** 할 수 없습니다.
- 반대로 쓰기 작업이 먼저 잠금을 획득한 경우  
  → SELECT조차도 **락 대기 상태** 에 놓이게 됩니다.

즉, **하나의 트랜잭션이 읽거나 쓰는 레코드는 다른 트랜잭션에서 절대 접근할 수 없습니다.**

#### 사용 시 주의점

- 높은 수준의 정합성이 필요한 경우에는 적합합니다.
- 그러나 대부분의 일반적인 시스템에서는 너무 과도한 락으로 인해 **성능 저하, 데드락 위험, 락 경합** 등의 문제가 발생할 수 있어 주의해야 합니다.
