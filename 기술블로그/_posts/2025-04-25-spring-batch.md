---
layout: post  
title: "스프링배치"
author: "정인재"
categories: "기술블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["Batch"]
---

# Spring Batch 실전 사용기 – 대용량 데이터를 안전하게 처리하는 법

> “서버가 뻗지 않으면서도, 수십만 건의 데이터를 정해진 시간에 처리할 수 있을까?”

대용량 데이터를 안전하게 처리해야 하는 순간은 생각보다 자주 찾아온다. 나 역시 **차량 관제 시스템에서 하루 수만 건씩 쌓이는 로그를 집계**하고, 이를 **정기적으로 통계 테이블로 이관**하는 작업을 맡으며 Spring Batch를 실무에 도입했다.  
이 글에서는 내가 **직접 Spring Batch를 실전에 적용하며 배운 점, 주의할 점, 그리고 성능까지** 모두 공유한다.

---
# Spring Batch 실전 사용기 – 대용량 데이터를 안전하게 처리하는 법

> “서버가 뻗지 않으면서도, 수십만 건의 데이터를 정해진 시간에 처리할 수 있을까?”

대용량 데이터를 안전하게 처리해야 하는 순간은 생각보다 자주 찾아온다. 나 역시 **차량 관제 시스템에서 하루 수만 건씩 쌓이는 로그를 집계**하고, 이를 **정기적으로 통계 테이블로 이관**하는 작업을 맡으며 Spring Batch를 실무에 도입했다.  
이 글에서는 내가 **직접 Spring Batch를 실전에 적용하며 배운 점, 주의할 점, 그리고 성능까지** 모두 공유한다.

---

## ✅ 왜 Spring Batch인가?

- 트랜잭션 단위로 청크 처리 가능
- 재시작/에러/skip 처리 로직 내장
- 멀티 쓰레드/병렬 처리 지원
- 설정만 잘 하면 운영 자동화까지 가능

---

## 📦 기본 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| **Job** | 전체 배치 실행 단위 |
| **Step** | Job을 구성하는 단위 작업 |
| **Reader** | 데이터를 읽는 컴포넌트 (ex. DB, CSV 등) |
| **Processor** | 읽은 데이터를 가공 |
| **Writer** | 가공된 데이터를 저장 |

---

## ⚙️ 실전 구조 예시 (CarLog → CarLogSummary 이관)

```text
1. main DB에서 CarLog 읽기
2. 통계 가공 (ex. 이동거리 계산, 시간대별 집계 등)
3. stat DB의 CarLogSummary 테이블에 저장
```

```java
@Bean
public Step carLogToSummaryStep() {
    return stepBuilderFactory.get("carLogToSummaryStep")
        .<CarLog, CarLogSummary>chunk(1000) // 청크 단위로 처리
        .reader(carLogReader())
        .processor(carLogSummaryProcessor())
        .writer(carLogSummaryWriter())
        .build();
}
```

---

## 💡 실무에서 얻은 팁

### 1. 청크 크기 (chunk size)는 무조건 실측하자

- 너무 작으면 느리고, 너무 크면 OutOfMemory!
- 나는 처음 100 → 500 → 1000까지 올려가며 TPS 확인
- TPS 15000 시스템에서는 1000 청크가 안정적이었음

### 2. 트랜잭션 범위 명확히 인지할 것

- `chunk()` 단위가 트랜잭션 범위임
- `@Transactional(readOnly = true)`을 reader에 쓰면 성능 개선됨

### 3. Reader는 Stream 기반으로

- `JpaPagingItemReader`보다 `JdbcCursorItemReader` 추천
- 커넥션 풀 부담 줄이려면 오프타임에 실행하거나 배치 전용 커넥션 구성

### 4. Restart 가능하게 만들기

- 동일한 Job이 재시작되면 어디서부터 다시 할지 지정해야 함
- `JobParameters`를 고유하게 설정하지 않으면 이전 실행 이력에 영향

```java
JobParameters params = new JobParametersBuilder()
        .addLong("timestamp", System.currentTimeMillis())
        .toJobParameters();
```

### 5. 실패를 감지하고 복구하라

```java
@Bean
public Step stepWithSkip() {
    return stepBuilderFactory.get("stepWithSkip")
        .<Item, Item>chunk(100)
        .reader(reader())
        .processor(processor())
        .writer(writer())
        .faultTolerant()
        .skip(CustomException.class)
        .skipLimit(100) // 최대 100건까지는 스킵
        .build();
}
```

---

## 🔎 모니터링과 로그는 선택이 아닌 필수

- Spring Actuator와 연동해 배치 상태 확인
- JobExecutionListener를 활용한 시작/종료 로그 기록
- Prometheus + Grafana 연동도 추천 (성능 추이 파악)

---

## 🧪 성능 측정 결과

| 청크 크기 | 처리 건수 | 소요 시간 |
|----------|----------|-----------|
| 100      | 30,000   | 120초     |
| 500      | 30,000   | 50초      |
| 1000     | 30,000   | 32초      |

> 🧠 단순히 빠른 게 아니라 **안정적으로 재처리 가능하고**, **리소스 폭주 없는** 구조가 더 중요하다.

---

## 🧵 마무리: 실무에서 필요한 배치는 결국 "안전성"

Spring Batch는 강력하지만, 설정이 복잡해보여 처음엔 진입 장벽이 있다. 그러나 일단 한 번 구조를 잡아놓으면, 운영 안정성이나 확장성 면에서 큰 이점을 준다.

- **TPS가 높은 시스템에서도 배치는 비동기적 안전망 역할**
- **모든 데이터를 한 번에 처리할 필요 없음 – 나눠서, 천천히, 안전하게**

---

## 📚 함께 보면 좋은 확장 주제

- `Spring Cloud Task`로 경량 배치 처리하기
- `Quartz`와 스케줄링 통합
- `Multi-threaded Step`, `Partitioning`, `Remote Chunking` 심화
- S3, Kafka 등 외부 시스템과 연동하는 Reader/Writer 구현법

---

_읽어주셔서 감사합니다. 실무에서 Spring Batch를 도입하며 겪은 경험을 공유했는데, 도움이 되셨다면 댓글이나 공감 부탁드려요!_
