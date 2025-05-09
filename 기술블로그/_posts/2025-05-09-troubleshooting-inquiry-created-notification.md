---
layout: post
title: "Race Condition 해결기: 트랜잭션 커밋 이후 알림 전송 보장하기"
author: "박병찬"
categories: "기술블로그"
banner:
  image: "/assets/images/transaction-commit-banner.jpg"
  background: "#1f1f1f"
  height: "100vh"
  min_height: "40vh"
  heading_style: "font-size: 4em; font-weight: bold; text-decoration: underline"
  tags: ["Spring", "Java", "Transaction", "Async", "Race Condition"]
---

# Race Condition 해결기: 트랜잭션 커밋 이후 알림 전송 보장하기

## 📚 목차

1. [문제 상황 개요](#1-문제-상황-개요)
2. [시스템 처리 흐름과 구조](#2-시스템-처리-흐름과-구조)
3. [증상: 알림이 먼저 저장이 나중](#3-증상-알림이-먼저-저장이-나중)
4. [원인 분석: 비동기 처리와 트랜잭션 미보장](#4-원인-분석-비동기-처리와-트랜잭션-미보장)
5. [해결 전략 1: CompletableFuture를 이용한 체이닝](#5-해결-전략-1-completablefuture를-이용한-체이닝)
6. [해결 전략 2: 트랜잭션 커밋 이후 후처리](#6-해결-전략-2-트랜잭션-커밋-이후-후처리)
7. [개선 코드 비교](#7-개선-코드-비교)
8. [회고 및 실무 팁](#8-회고-및-실무-팁)

---

## 1. 문제 상황 개요

고객 문의 폼이 제출되면, 내부 알림(Notification)과 이메일 전송이 함께 이뤄지도록 만들었어요.  
이메일은 `@Async`로 비동기로 보내고, SSE는 즉시 알림을 쏘는 구조였는데요.  
운영 중에 **알림이 안 보이거나 null로 오는 현상**이 간헐적으로 생기더라고요.

## 2. 시스템 처리 흐름과 구조

요약하자면 흐름은 이랬습니다:

1. 고객 문의 내용을 DB에 저장
2. 알림(Notification) 객체 생성 후 저장
3. 이메일은 비동기로 전송
4. SSE 알림은 바로 전송

⚠️ 그런데 문제는, 트랜잭션이 커밋되기 전에 SSE가 먼저 실행돼버리는 경우가 있었어요.  
그 결과, **DB에 알림이 저장되기도 전에 알림 전송이 먼저 일어나는 상황**이 생긴 거죠.

## 3. 증상: 알림이 먼저? 저장이 나중?

이런 문제가 나타났어요:

- 이메일은 큐 기반이라 괜찮았고요
- SSE는 아래처럼 이상한 증상을 보였어요
  - null 알림이 뜸
  - 알림 요청 시 404 Not Found 에러
  - URL이 깨진 채로 노출

📌 공통점은 대부분 **Notification이 저장되기 전에 SSE가 먼저 실행된 것** 같았어요.

## 4. 원인 분석: 비동기 처리와 트랜잭션 미보장

Spring의 `@Async`는 메인 쓰레드를 기다리지 않아요.  
그래서 **작업 순서를 보장하지 못하는 문제가 생기고**,  
SSE는 DB 저장 직후 실행되지만, 이때 트랜잭션이 커밋되지 않았다면  
**DB에는 아직 반영되지 않은 상태일 수도 있어요.**

### 핵심 문제는 이거예요:

- ✅ 비동기 메서드 사이의 **실행 순서가 보장되지 않음**
- ✅ 트랜잭션 **커밋 전에 후처리 로직이 실행됨**

## 5. 해결 전략 1: `CompletableFuture`를 이용한 체이닝

```java
public void handleCustomerInquiry(InquiryRequest request) {
    Notification notification = saveNotification(request);

    CompletableFuture
        .completedFuture(notification)
        .thenAcceptAsync(this::sendEmailAsync)
        .thenRun(() -> sendSSE(notification));
}
```

### 👍 장점

- `completedFuture()`로 비동기 흐름을 제어할 수 있어요
- `thenAcceptAsync()`로 이메일은 비동기로 보내고
- `thenRun()`을 통해 **이메일 전송 완료 후에 SSE를 보내는 순서 보장**이 가능해요

## 6. 해결 전략 2: 트랜잭션 커밋 이후 후처리 (`TransactionSynchronizationManager`)

```java
@Transactional
public void sendInquiryNotification(InquiryCreatedEvent event) {
    Notification saved = notificationStore.create(...);

    TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
        @Override
        public void afterCommit() {
            notificationExecutor.send(saved); // 안전한 시점
        }
    });
}
```

### 👍 장점

- 트랜잭션이 **커밋된 이후에만 실행**돼서,
- Notification이 **DB에 확실히 반영된 시점에 전송 가능**해요
- Race Condition을 근본적으로 방지할 수 있어요

> 💡 참고로 이 방식은 반드시 `@Transactional` 영역 안에서만 사용할 수 있어요!

## 7. 개선 코드 비교

### 🔴 기존 코드 (문제 있음)

```java
notificationService.save(notification);
emailService.sendAsync(notification);  // 비동기
sseService.send(notification);         // 동기 → race condition 발생 가능
```

### 🟢 개선 코드 1 - `CompletableFuture`

```java
CompletableFuture
    .completedFuture(notificationService.save(notification))
    .thenAcceptAsync(emailService::sendAsync)
    .thenRun(() -> sseService.send(notification));
```

### 🟢 개선 코드 2 - `TransactionSynchronizationManager`

```java
Notification saved = notificationService.save(notification);
TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
    @Override
    public void afterCommit() {
        emailService.sendAsync(saved);
        sseService.send(saved);
    }
});
```

## 🤔 회고 및 실무 팁

- **비동기 작업의 순서를 절대 믿지 마세요!**
- 실시간 알림처럼 타이밍이 중요한 건 **무조건 트랜잭션 커밋 이후** 처리하는 게 안전합니다
- `CompletableFuture`는 **비동기 흐름 제어**에 좋아요
  `TransactionSynchronizationManager`는 **DB 반영을 확실하게 보장**해줍니다
- 상황에 따라 둘 중 하나 또는 **둘 다 병행**해서 사용할 수도 있어요

## 📌 정리

| 전략                | 장점                     | 주의할 점                                    |
| ------------------- | ------------------------ | -------------------------------------------- |
| `CompletableFuture` | 순서 보장, 비동기 체이닝 | 트랜잭션 상태는 직접 확인이 어렵다는 점      |
| `afterCommit()`     | 트랜잭션 이후 보장       | 반드시 `@Transactional` 영역 안에서만 동작함 |

트랜잭션과 비동기 작업을 같이 쓸 땐 항상
**"이 로직이 언제 실행되는가?"** 를 명확하게 파악해야 합니다.

이번 경험 덕분에 알림 시스템의 신뢰도를 높일 수 있었고,
비슷한 문제로 고민 중인 분들에게도 도움이 되었으면 좋겠네요. 😊
