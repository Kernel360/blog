---
layout: post  
title: Asynch in Java
author: 신종민
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0222/cover.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [기술세미나, Java, Asynch]
---

안녕하세요. Kernel360 1기 신종민입니다. 오늘 제가 준비한 내용은 자바에서의 비동기 프로그래밍에 대한 것입니다.

### 동기와 비동기 개념
우선 동기와 비동기의 기념에 대해서 설명드리겠습니다.
동기 프로그래밍은 작업이 순차적으로 진행하도록 하는 것입니다. 만약 이전의 작업이 완료되지 않으면 다음 라인은 실행이 되지 않습니다.

반면에 비동기 프로그래밍은 작업이 완료될 때까지 기다리지 않고 잠재적으로 오래 실행되는 작업을 시작하여 해당 작업이 실행되는 동안에도 다른 이벤트에 응답할 수 있게 하는 기술입니다. 작업이 완료되면 프로그램이 결과를 제공합니다.
프로그래밍이 다소 복잡하지만 자원을 효율적으로 사용할 수 있습니다. 

예를 들어서 설명드리겠습니다.
얼굴을 마주보며 상대방과 대화를 하는 경우와, 이메일을 통해 메일을 주고받으면서 대화를 하는 경우를 들 수 있습니다. 
마주보며 상대방과 대화를 하는 경우에는 내가 하는 말과 상대방의 말이 순차적으로 진행이 되죠. 한 사람의 말이 끝나면 그에 대한 적절한 응답을 내놓는 것이 예의일 것입니다. 상대방이 어떤 말을 했는데 한 시간 쯤 tv를 보다가 응답을 한다면 제대로 된 대화가 이루어질 수 없겠죠. 이러한 대화 방식을 동기적인 대화 방식이라고 할 수 있겠습니다.
반면에 이메일을 통해 대화를 주고 받는 경우를 생각해봅시다. 
제가 이메일을 처리할 때는 상대방이 언제 메일을 보냈는지에 상관없이 저는 제가 시간이 날 때 처리합니다.아주 중요한 메일이 날아오기로 예정되어 있다면 알람이 뜨는지 안 뜨는지 기다리다 메일이 오면 바로 확인하겠지만 보통은 그러지 않죠. 상대방이 메일을 보낸 시간과 제가 그 메일을 받고 행동을 취하는 시간이 일치하지 않습니다. 이런 경우 비동기적으로 메시지를 주고 받는 경우라고 생각합니다.

### 블로킹과 논블로킹 개념
다음으로는 블로킹과 논 블로킹에 대해서 말씀드리겠습니다.
블로킹과 논 블로킹은 제어권을 호출한 함수가 가지고 있는지 가지고 있지 않는지에 따라 나눌 수 있습니다.
한 쓰레드가 다른 쓰레드를 호출했을 때, 호출한 쓰레드가 여전히 제어권을 가지고 다음 작업을 수행할 수 있다면 논 블로킹이라고 합니다. 반면에 블로킹은 피 호출된 쓰레드가 작업을 완료하고 제어권을 반환할 때가지 호출한 함수는 제어권을 잃어서 다음 작업을 진행할 수 없는 상태를 나타냅니다.
예를 들어서 설명드리겠습니다.

만약 우리가 머리를 하기 위해서 헤어샵에 갔다고 생각해보겠습니다. 헤어 디자이너가 머리를 하는 동안에는 손님은 할 수 있는게 별로 없죠. 특히 저처럼 시력이 나쁘면 머리를 할 때는 안경을 벗으니 아무것도 하지 못합니다. 이런 경우는 저로서는 행동에 많이 제약이 생기고 주도권은 디자이너에게 있어서 디자이너가 빨리 머리를 완성해줘야 다른 재미있는 것을 할 수 있게 됩니다.  이렇게 한 쓰레드가 다른 쓰레드를 호출 했을 때 다른 쓰레드가 결과를 내놓고 제어권을 돌려줄 때까지 아무것도 못하는 경우를 블록(차단)됐다고 할 수 있습니다.

이번에는 헤어샵이 아니라 카페를 가서 음료를 주문하고 결제하고 진동벨을 받은 상황을 생각해보겠습니다. 이런 경우 진동벨이 울릴 때까지 매장에서 마음껏 돌아다니면서 친구와 대화를 할 수도 있고 자리를 잡아서 코딩을 하거나 책을 읽을 수도 있습니다. 이렇게 다른 프로세스나 쓰레드에 작업을 맡긴 후에도 제어권이 있어서 다른 작업을 할 수 있는 경우 논 블로킹(비차단) 이라고 합니다. 만약에 음료를 주문했을 때 다른 데 가지 못하고 기다렸다가 음료가 완성되면 받아서 가야 한다면 이 경우는 블로킹에 해당한다고 볼 수 있습니다.

### 두 개념 간 비교
저는 다음과 같이 동기와 비동기는 다른 쓰레드의 결과를 기다리는 지 여부에 대한 것이고, 블로킹 논 블로킹은 다른 스레드를 실행했을 때 제어권을 잃고 아무것도 할 수 있는지, 반대로 다른 쓰레드의 행동과는 관계없이 다른 작업을 진행할 수 있는지에 대한 것이라고 할 요약하게 되었습니다.

### 비동기를 왜 사용하는가?
그러면 왜 비동기를 알고 사용해야 할까요?
즉 어떤 장점이 있을까요?
성능을 향상시킬 수 있습니다. 비동기는 여러 작업을 동시에 병렬로 실행할 수 있기 때문에 전체적인 시스템의 성능을 향상시킬 수 있습니다. 특히 I/O 작업의 경우 대기 시간이 길어질 수 있기 떄문에 동기적으로 처리한다면 상당히 많은 대기 시간이 발생할 수 있습니다. 
비동기적으로 처리할 때 I/O를 기다리는 동안 그 결과와 상관 없는 다른 작업을 실행한다면 더 빠른 속도로 일을 처리할 수 있을 것 같습니다. I/O 외의 다른 자원을 소모하는 작업을 함으로써 효율적인 자원 관리도 가능하겠죠.
응답성이 향상됩니다. 예를들어서 클라이언트의 http 요청을 서버가 처리하고 이를 응답 받아 클라이언트가 다른 작업을 수행하는 경우를 생각해봅시다. 만약 동기적으로 이러한 작업을 처리한다면 클라이언트는 요청을 보내고 그 요청이 서버에서 처리되어 응답을 생성하고 클라이언트에 전송하여 클라이언트가 응답을 수신하고 처리한 다음에야 다음 작업을 수행할 수 있습니다.

### 자바에서 비동기 처리를 위한 API
- ExecutorService와 Future
- CompletableFuture
- Java NIO(Non-blocking I/O)
- 기타

이제 마지막으로 자바에서의 비동기 처리를 위한 api에 대해서 알아보겠습니다. 

자바에서 비동기 프로그래밍을 지원하기 위해 다양한 API와 라이브러리가 있습니다. 여기서는 수많은 라이브러리 중 다음과 같은 몇 가지를 소개하고자 합니다.

java.util.concurrent 패키지에 있는 ExecutorService와 Future, CompletableFuture, ForkJoinPool 등이 있으며
java.nio, 그리고 Spring의 WebFlux 등이 있습니다.

이 중 저는 여기서는 Future와 CompletableFuture에 대해서 알아보겠습니다.
수많은 라이브러리가 있지만 여기서는 가장 기본적인 앞의 2 가지만 알아보고자 합니다.

### Future & ExecutorService
- 자바 5부터 도입된 인터페이스, 
- get() 메서드를 호출하여 작업이 완료되었는지 확인하거나 작업의 결과를 가져올 수 있습니다.
- 실행자(ExcecutorService)를 활용하여 Future 객체 생성

Future와 CompletableFuture는 모두 자바에서 비동기 및 병렬 프로그래밍을 지원하는 클래스입니다.

Future 는 자바5에서 도입된 인터페이스 입니다.  Future 인터페이스는 비동기 작업의 결과를 나타내는 데 사용되지만, 그 자체로는 작업을 실행하는 기능을 제공하지 않습니다.
따라서 ExecutorService, 또는 CompletionService와 같은 실행자를 활용하여 작업을 실행하면 실행자로부터 Future 객를 얻어낼 수 있고, Future 의 get 메서드를 통해 작업의 결과를 가져오거나 작업이 완료되었는지 확인할 수 있습니다.

ExecutorService는 자바에서 멀티스레드 환경에서 작업을 관리하고 실행하기 위한 인터페이스를 제공하는 유틸리티입니다. 주요 목적은 스레드의 생성 및 관리, 작업을 효율적으로 분배하고 관리하는 것입니다. ExecutorService는 스레드 풀을 사용하여 작업을 처리하며, 스레드 풀은 재사용 가능한 스레드의 집합입니다.

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor submit ( () > "Hello, ");
String result = tuture.get(): // 블로킹 메서드, 작업 완료 대기
System.out.println (result);
executor.shutdown();
```
ExecutorService를 사용할 때 주의해야 할 점이 있습니다. 작업을 완료한 후에는 반드시 shutdown 또는 shutdownNow 메서드를 호출하여 자원을 해제해야 합니다. 스레드 풀을 효과적으로 활용하기 위해서는 적절한 구성과 스레드 개수 설정이 중요합니다. 또한, Future 객체를 사용하여 작업의 상태를 확인하고 결과를 얻을 수 있습니다. Java의 java.util.concurrent 패키지에는 다양한 유형의 ExecutorService가 제공되며, Executors 클래스를 사용하여 간편하게 생성할 수 있습니다. newFixedThreadPool, newCachedThreadPool, newSingleThreadExecutor 등의 메서드를 통해 다양한 종류의 스레드 풀을 생성할 수 있습니다.

### CompletableFuture
- 자바 8에서 추가
- Future의 모든 기능 포함
- 개선된 기능 제공(콜백, 예외 처리, 조합, 합치기)

CompletableFuture는 자바 8에서 추가된 클래스로, Future를 확장하여 비동기 작업을 더 편리하게 다룰 수 있도록 개선된 기능을 제공합니다. 이는 콜백, 예외 처리, 조합, 합치기 등을 포함하여 강력하고 편리한 비동기 프로그래밍을 지원합니다. 그래서 자바 8 이상에서는 CompletableFuture를 사용하는 것이 더 좋을 것으로 보입니다.

### CompletableFuture의 특징
1. 비동기적인 작업 조작
2. 작업 조합과 체이닝
3. 작업 조건 및 예외 처리
4. 여러 CompletableFuture 합치기
5. Timeout 및 기본값 설정
6. 취소 및 종료 관리


#### 1. 비동기적인 작업 조작
CompletableFuture는 비동기적으로 실행되는 작업을 생성하고 조작하는 데 사용됩니다. supplyAsync, runAsync와 같은 정적 메서드를 사용하여 비동기 작업을 생성할 수 있습니다.
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 비동기 작업 수행
    return "Hello, CompletableFuture!";
});
```

#### 2. 작업 조합과 체이닝
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 비동기 작업 수행
    return "Hello, CompletableFuture!";
});
```
또 작업을 조합하고 체이닝을 할 수 있습니다.
thenApply, thenAccept, thenRun 등의 메서드를 사용하여 작업을 조합하고 연결할 수 있습니다. 이를 통해 여러 비동기 작업을 순차적으로 또는 병렬로 실행할 수 있습니다.

#### 3. 작업 조건 및 예외 처리
그리고 다음과 같이 whenComplete,메서드나  exceptionally 등의 메서드를 사용하여 작업이 완료된 후의 조건 및 예외 처리를 수행할 수 있습니다.
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 비동기 작업 수행
    return "Hello, CompletableFuture!";
});

future.whenComplete((result, exception) -> {
    if (exception != null) {
        System.err.println("Exception: " + exception.getMessage());
    } else {
        System.out.println("Result: " + result);
    }
});
```

#### 4. 여러 CompletableFuture 합치기
thenCombine, thenAcceptBoth 등의 메서드를 사용하여 여러 CompletableFuture를 합칠 수 있습니다.
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combinedFuture = future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2);

```

#### 5. Timeout 및 기본값 설정
completeOnTimeout, orTimeout 메서드를 사용하여 작업이 일정 시간 내에 완료되지 않을 때 기본값을 제공하거나 예외를 발생시킬 수 있습니다.
future의 get의 특징과 비교해서 앞에서 알아본 블로킹, 논 블로킹의 관점에서 가장 다른 부분이 여기라고 생각합니다.
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 일부 시간이 오래 걸리는 작업
    return "Hello, CompletableFuture!";
}).completeOnTimeout("Default Value", 1, TimeUnit.SECONDS);

```

#### 6. 취소 및 종료 관리
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 일부 작업 수행
    return "Hello, CompletableFuture!";
});

boolean canceled = future.cancel(true);
```

cancel, isCancelled, isCompletedExceptionally 등의 메서드를 사용하여 CompletableFuture를 취소하거나 종료 여부를 확인할 수 있습니다.

### CompletableFuture와 Future의 차이점
CompletableFuture 가 Future와 다른 점 또는 개선된 점을 말씀드리겠습니다.

먼저 Future 인터페이스에 정의된 get 메소드에 대한 것인데요, ExecutorService의 submit 메서드가 반환하는 Future 객체의 get 메서드는 작업이 완료될 때까지 블로킹이 되는 반면에,
CompletableFuture의 get 메서드는 Future의 get메서드와 달리 기본적으로 블로킹되지 않습니다. 때문에 비동기 작업이 완료되지 않더라도 다른 작업을 계속 수행할 수 있습니다.

예외 처리에서도 차이를 보입니다.
Future의 get 메서드는 호출시 예외가 발생하면 ExecutionException으로 감싸기는 반면에 CompletableFuture의 get 메서드는 CompletionException을 던집니다.

다음은 취소 처리에 관한 것입니다.
Future와 CompletableFuture 모두 작업 취소를 위해 cancel 메서드를 제공하지만, CompletableFuture는 completeExceptionally 메서드를 사용하여 예외를 발생시키면서 취소할 수 있습니다.

이러한 점들때문에 자바5에서 도입된 Future를 확장하여 자바 8 CompletableFuture를 통해 더욱 비동기 작업을 효과적으로  다룰 수 있게 되었습니다.


### 결론

| 특징              | 동기                    | 비동기                     |
|----------------|-----------------------|--------------------------|
| 진행 방식       | 순차적 (Sequential)    | 비순차적 (Non-Sequential) |
| 작업 완료 대기 | 대기 (Blocking)        | 비대기 (Non-Blocking)     |
| 호출 방식       | 동기적 호출 (Synchronous) | 비동기적 호출 (Asynchronous) |
| 작업 순서       | 이전 작업 완료 후 다음 작업 | 다른 작업과 동시에 진행   |
| 장점            | 코드 간단, 직관적         | 다중 작업 효과적 처리, 대기 시간 활용 |
| 단점            | 대기 시간으로 인한 지연 가능 | 코드 복잡성 증가, 디버깅 어려움 |

제가 준비한 것은 여기까지 입니다! 들어주셔서 감사합니다. 소중한 시간을 내주셔서 감사드립니다.


[참고 문헌]
- [MDN](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Asynchronous/Introducing)
- [오토봇 팩토리님 블로그](https://private.tistory.com/24)
- [인파님 블로그](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC)