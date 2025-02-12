---
layout: post  
title: "안티패턴 - 양방향 레이어드 아키텍처"
author: "김도훈"
categories: "기술 블로그"
banner:
image: 
background: "#000"
height: "100vh"
min_height: "38vh"
heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [`layeredArchitecture`]
---
# 안티패턴 - 양방향 레이어드 아키텍처

개발에는 정답이 없지만 유지보수성과 확장성 측면에서 좋지 않은 것으로 알려진 '안티패턴'이 존재합니다. 이번 글에서는 그중에서도 '양방향 레이어드 아키텍처(Bidirectional Layered Architecture)'에 대해 살펴보겠습니다.

## 양방향 레이어드 아키텍처(Bidirectional Layered Architecture)
'양방향 레이어드 아키텍처' 안티패턴은 레이어드 아키텍처를 따르는 프로젝트에서 자주 발생하는 문제입니다. 이는 레이어드 아키텍처에서 정의된 레이어들 간의 의존 관계에서 양방향 의존이 발생하는 경우를 의미합니다. 이를 설명하기 위해 먼저 레이어드 아키텍처에 대해 알아보겠습니다.

### 레이어드 아키텍처란?
레이어드 아키텍처는 소프트웨어 시스템을 설계하는 방식 중 하나로, '레이어(Layer)'라는 개념을 기반으로 합니다. 이 아키텍처를 따르는 애플리케이션은 개발 전 미리 레이어를 정의하며, 보편적으로 다음과 같은 세 가지 레이어를 포함합니다.

```
Presentation Layer -> Business Layer -> Infrastructure Layer
```

#### 1. 프레젠테이션 레이어(Presentation Layer)
이 레이어는 사용자와의 상호작용을 처리하고 결과를 표시하는 역할을 담당합니다. 대표적인 스프링 컴포넌트로는 컨트롤러가 있으며, 쉽게 말해 컨트롤러와 같은 컴포넌트가 모이는 곳이라고 볼 수 있습니다.

#### 2. 비즈니스 레이어(Business Layer)
비즈니스 로직을 처리하는 레이어로, 데이터 유효성 검사, 데이터 가공, 비즈니스 규칙 적용 등의 역할을 수행합니다. 이로 인해 스프링에서는 주로 서비스 컴포넌트가 이곳에 위치합니다.

#### 3. 인프라스트럭처 레이어(Infrastructure Layer)
이 레이어는 외부 시스템과의 상호작용을 담당하며, 대표적인 예로 데이터베이스가 있습니다. 스프링에서는 JDBC, JPA, 하이버네이트 등의 데이터 접근 기술이 사용되며, 이 레이어에 배치되는 코드는 주로 데이터를 저장하거나 조회하는 역할을 수행합니다. 이 때문에 좁은 의미에서 '영속성 레이어(Persistence Layer)'라고도 불립니다.


## 레이어드 아키텍처 기반 프로젝트 구성 예시
레이어드 아키텍처를 기반으로 프로젝트를 구성하는 예시를 살펴보겠습니다. 대부분의 프로젝트는 다음과 같은 패키지 구조를 따릅니다. 레이어에 대응하는 패키지를 먼저 만들고(`presentation`, `business`, `infrastructure`) 해당 패키지에 맞는 스프링 컴포넌트를 넣는 방식입니다.

```
Project/src/main/java
 ├── presentation
 │    ├── UserController.java
 │    ├── CafeController.java
 │    ├── BoardControler.java
 │    ├── PostController.java
 ├── business 
 │    ├── UserService.java
 │    ├── CafeService.java
 │    ├── BoardService.java
 │    ├── PostService.java
 ├── infrastructure  
 │    ├── UserRepository.java
 │    ├── CafeRepository.java
 │    ├── BoardRepository.java
 │    ├── PostRepository.java
 ├── Entity  
 │    ├── User.java
 │    ├── Cafe.java
 │    ├── Board.java
 │    ├── Post.java
```

이 구조를 보면 레이어에 대응하는 패키지를 만든 것을 확인할 수 있습니다. 그렇다면 레이어드 아키텍처를 이렇게 구성할 때 얻을 수 있는 장점은 무엇일까요? 가장 큰 장점은 '단순하고 직관적인 구조'라는 것입니다. 즉, 개발자가 컴포넌트를 어디에 위치시켜야 할지 고민할 필요가 없습니다. 컴포넌트의 유형만 파악하면 적절한 레이어를 바로 찾을 수 있어 개발이 쉬워집니다.

### 기능 개발이 쉬운 이유
이해를 돕기 위해 예시를 들어보겠습니다. 위와 같은 구조를 가진 프로젝트에서 새로운 기능을 개발해야 한다고 가정해봅시다. 개발자는 다음과 같은 순서로 접근하면 됩니다.

1. 엔드포인트를 위한 컨트롤러 생성
2. 컨트롤러가 실행할 서비스 생성
3. 서비스에서 데이터를 가져오기 위한 리포지토리 생성

즉, 컨트롤러, 서비스, 리포지토리만 만들면 API를 하나 추가할 수 있습니다.

또한, 각 컴포넌트의 위치도 명확합니다. `presentation`, `business`, `infrastructure` 패키지에 맞춰 컴포넌트를 배치하면 되므로, 구조적으로 깊이 고민할 필요가 없습니다. 개발자는 '엔드포인트의 형태'와 '비즈니스 로직'에 집중하면 되므로 단기적으로 개발이 쉬워지는 장점이 있습니다.

여기까지가 레이어드 아키텍처의 기본적인 구조입니다.

---

### 양방향 레이어드 아키텍처란?

이제 ‘양방향 레이어드 아키텍처’가 무엇인지에 대해 알아보겠습니다. 양방향 레이어드 아키텍처는 레이어드 아키텍처를 지향해 개발했지만, 레이어드 아키텍처가 반드시 지켜야 할 가장 기초적인 제약을 위반할 때를 지칭하는 말입니다. 그리고 여기서 가장 기초적인 제약이란 **’레이어 간 의존 방향은 단방향을 유지해야 한다’**라는 것입니다.

### 양방향 의존성 사례 1: DTO(요청 모델) 참조
 아래 예시는 서비스 컴포넌트에서 API 요청을 매핑하는 PostCreateRequest DTO를 직접 사용하는 경우입니다.

```java
@Service
@RequiredArgsConstructor
public class PostService {

  // 각 레포지토리 주입 (비즈니스 로직에 필요한 의존성)
  private final CafeMemberJpaRepository cafeMemberJpaRepository;
  private final BoardJpaRepository boardJpaRepository;
  private final PostJpaRepository postJpaRepository;
  private final UserJpaRepository userJpaRepository;
  private final CafeJpaRepository cafeJpaRepository;

  @Transactional
  public Post create(
    long cafeId,
    long boardId,
    long writerId,
    PostCreateRequest postCreateRequest) { // 프레젠테이션 레이어에서 전달된 DTO를 직접 사용함

    // 현재 타임스탬프를 가져옴 (게시글 생성 시간)
    long currentTimestamp = Instant.now().toEpochMilli();

    // 카페 멤버를 조회 (권한 검사 포함)
    CafeMember cafeMember = cafeMemberJpaRepository
      .findByCafeIdAndUserId(cafeId, writerId)
      .orElseThrow(() -> new ForbiddenAccessException());

    // 작성자(User)를 조회
    User writer = userJpaRepository
      .findById(writerId)
      .orElseThrow(() -> new UserNotFoundException());

    // 카페 정보 획득
    Cafe cafe = cafeMember.getCafe();

    // 게시판(Board)을 조회
    Board board = boardJpaRepository
      .findById(boardId)
      .orElseThrow(() -> new BoardNotFoundException());

    // 새로운 게시글(Post) 객체 생성
    Post post = new Post();

    // API 요청용 DTO에서 제목과 내용을 가져와 게시글에 설정
    post.setTitle(postCreateRequest.getTitle());
    post.setContent(postCreateRequest.getContent());
    post.setCafe(cafe);
    post.setBoard(board);
    post.setWriter(writer);
    post.setCreatedTimestamp(currentTimestamp);
    post.setModifiedTimestamp(currentTimestamp);

    // 게시글 저장
    post = postJpaRepository.save(post);

    // 카페의 최신 게시글 타임스탬프 업데이트 후 저장
    cafe.setNewPostTimestamp(currentTimestamp);
    cafe = cafeJpaRepository.save(cafe);

    return post;
  }
}

```
위 코드에서 PostCreateRequest 클래스는 프레젠테이션 레이어(컨트롤러)에서 사용하기 위해 만들어진 DTO입니다. 이를 비즈니스 레이어에서 그대로 사용함으로써 두 레이어 간에 양방향(순환) 의존성이 발생하게 되고, 이는 결국 레이어의 경계를 모호하게 만듭니다.

해결 방법 1: 레이어별 모델 구성
비즈니스 레이어에서 별도의 DTO(예, PostCreateCommand)를 사용하여 의존 방향을 단방향으로 유지합니다.

```java
// 프레젠테이션 레이어의 DTO
@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class PostCreateRequest {
  // 클라이언트에서 전달되는 게시글 생성 요청 데이터
  private String title;
  private String content;
}
```
```java
// 비즈니스 레이어 전용 DTO
@Data
@Builder
public class PostCreateCommand {
  // 서비스 로직에서 사용하는 게시글 생성 데이터 모델
  private String title;
  private String content;

  // writerId는 신뢰할 수 있는 인증 정보를 통해 할당됨
  private Long writerId;
}
```
컨트롤러에서는 API 요청을 받은 후, PostCreateRequest를 PostCreateCommand로 변환하여 서비스 메서드를 호출합니다.

```java
@PostMapping("/posts")
public ResponseEntity<PostResponse> createPost(
  @RequestBody PostCreateRequest request,
  @AuthenticationPrincipal UserPrincipal userPrincipal) {

  // 프레젠테이션 DTO(PostCreateRequest)를 서비스 전용 DTO(PostCreateCommand)로 변환
  PostCreateCommand command = PostCreateCommand.builder()
    .title(request.getTitle())               // 요청 DTO에서 제목 추출
    .content(request.getContent())             // 요청 DTO에서 내용 추출
    .writerId(userPrincipal.getId())           // 인증 정보를 통해 작성자 ID 설정
    .build();

  // 비즈니스 레이어의 서비스 메서드 호출 (cafeId, boardId는 URL 또는 기타 파라미터에서 전달받는다고 가정)
  Post post = postService.create(cafeId, boardId, command);

  // 결과를 PostResponse로 감싸서 클라이언트에 반환
  return ResponseEntity.ok(new PostResponse(post));
}

```
이렇게 하면 각 레이어에서 사용하는 모델이 분리되어 의존성이 단방향이 됩니다.

### 양방향 의존성 사례 2: 비즈니스 레이어(Service)에서 프레젠테이션 레이어(Controller)를 직접 참조
DTO 참조 외에도 더 심각한 문제는 비즈니스 로직을 수행하는 서비스 클래스에서 컨트롤러를 직접 참조하는 경우입니다. 이는 레이어 간 역할 분리가 무너지며, 컨트롤러에 정의된 기능(주로 사용자 인터페이스 관련 로직)이 비즈니스 로직에 영향을 주게 됩니다.

예를 들어 아래와 같은 코드를 보겠습니다.

```java
@RestController
@RequestMapping("/notifications")
@RequiredArgsConstructor
public class NotificationController {

  /**
   * 알림 메시지를 포맷팅하여 반환하는 메서드
   */
  @GetMapping("/format")
  public String formatNotification(@RequestParam String message) {
    // UI에 맞춘 포맷팅 처리 (예: 접두어 추가 등)
    return "Notification: " + message;
  }
}
```

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

  // ★ 문제: NotificationController는 프레젠테이션 레이어에 속하는데, 이를 서비스에 직접 주입받음
  private final NotificationController notificationController;

  public void sendNotification(String message) {
    // 비즈니스 로직 수행 중 컨트롤러의 포맷팅 메서드를 호출함 → 계층 간 역할 혼재 발생
    String formattedMessage = notificationController.formatNotification(message);

    // 추가적인 알림 전송 비즈니스 로직
    System.out.println("Sending notification: " + formattedMessage);
    // 예를 들어, 이메일 전송이나 푸시 알림 전송 로직이 이어질 수 있음
  }
}
```
### 문제점
#### 1. 레이어의 책임 혼재
비즈니스 로직(Service) 내에서 프레젠테이션 관련 기능(여기서는 주문 로그 기록)을 호출하게 되면, 두 레이어의 책임이 뒤섞이게 됩니다. 이는 유지보수 및 확장성에 큰 악영향을 미칩니다.서비스가 프레젠테이션 레이어의 기능(여기서는 메시지 포맷팅)을 직접 호출함으로써, 두 레이어의 역할이 섞여버립니다.

#### 2. 순환 참조 가능성
만약 컨트롤러가 내부적으로 서비스를 호출하고, 서비스가 다시 컨트롤러의 메서드를 호출하면 순환 참조가 발생하여 애플리케이션의 구조가 무너집니다.만약 컨트롤러가 내부적으로 서비스를 호출하는 경우, 양쪽이 서로를 참조하게 되어 순환 참조 문제가 발생할 수 있습니다.

#### 3. 테스트 어려움
서비스 단위 테스트 시 프레젠테이션 레이어 의존성이 포함되면, 목(mock) 객체 구성이나 테스트 설정이 복잡해집니다.

### 해결 방법
이 경우에도 기본 원칙은 레이어 간 단방향 의존성 유지입니다.

컨트롤러와 서비스의 역할을 명확히 분리해야 합니다.
만약 메세지 포맷팅 기능과 같이 여러 레이어에서 공통으로 사용할 로직이 있다면, 이를 별도의 유틸리티 클래스나 **공통 모듈(core)**로 분리하는 것이 좋습니다.
아래는 알림 메시지 포맷팅 기능을 전용 컴포넌트로 분리한 예입니다.

```java
@Component
public class NotificationFormatter {

  /**
   * 알림 메시지를 포맷팅하는 기능을 수행
   */
  public String format(String message) {
    return "Notification: " + message;
  }
}
```
```java
@Service
@RequiredArgsConstructor
public class NotificationService {

  // NotificationFormatter를 통해 알림 메시지 포맷팅 기능을 분리함
  private final NotificationFormatter notificationFormatter;

  public void sendNotification(String message) {
    // NotificationFormatter를 사용하여 포맷팅하므로 프레젠테이션 레이어에 의존하지 않음
    String formattedMessage = notificationFormatter.format(message);

    // 알림 전송 비즈니스 로직 수행
    System.out.println("Sending notification: " + formattedMessage);
  }
}
```
```java
@RestController
@RequestMapping("/notifications")
@RequiredArgsConstructor
public class NotificationController {

  // 별도의 알림 전송 로직이 필요하다면, NotificationService를 호출하는 방식으로 처리
  private final NotificationService notificationService;

  @PostMapping("/send")
  public ResponseEntity<String> sendNotification(@RequestBody String message) {
    notificationService.sendNotification(message);
    return ResponseEntity.ok("Notification sent.");
  }
}
```
## 2. 공통 모듈 구성
두 번째로 소개할 방법은 공통으로 참조하는 코드를 별도의 모듈로 분리하는 것입니다. 다시 말해 모든 레이어가 단방향으로 참조하는 공통 모듈을 만들고, `PostCreateRequest` 클래스 같은 모델을 거기에 배치하는 것입니다.

예를 들어, 첫 번째 예시에서 `PostCreateRequest` 클래스를 `core` 패키지로 옮기고 모든 레이어가 이 `core`라는 모듈에 의존하도록 변경하면 어떻게 될까요?
```
Project/src/main/java
 ├── core  ← 공통 모듈
 │    ├── dto
 │    │    ├── PostCreateDTO.java
```
공통 모듈로 분리한다는 전략은 범용적으로 사용할 수 있는 유틸성 클래스들을 한곳에 모아둘 때 유용합니다.

물론 이 같은 설명이 영 석연치 않을 수도 있습니다. 왜냐하면 이는 문제를 해결한 것이 아니라 회피한 것처럼 보이기 때문입니다.
- `core` 모듈은 레이어라고 봐야 할까요?
- `core` 모듈이 레이어라면 모든 레이어가 바라보는 하나의 레이어를 두는 것은 괜찮을까요?

‘공통 모듈 구성’에서 제시한 해결책으로 공통 코드를 한 곳에 모으라는 것은 공통으로 참조할 수 있는 모듈을 만들어 보라는 것이지 공통된 레이어를 만들라는 의미가 아닙니다. 레이어드 아키텍처에서 말하는 ‘레이어 간 통신은 인접한 레이어끼리 이뤄져야 한다’라는 제약은 ‘레이어’에서 발생하는 제약입니다. 레이어와 모듈은 다릅니다. 이러한 맥락에서 `core`는 모듈이며 레이어가 아닙니다. 그저 ‘순환 참조가 발생했을 때 공통 모듈로 분리할 수 있다’는 뜻입니다.

---

### 결론

양방향 레이어드 아키텍처는 레이어 간 의존성이 단방향으로 유지되지 않는 문제를 초래하며, 이는 유지보수성과 확장성을 저하시킵니다. 이를 방지하기 위해 레이어 별 모델을 분리하거나 공통 모듈을 구성하는 방법을 사용할 수 있습니다.

결국, 중요한 것은 레이어드 아키텍처의 본질을 유지하면서도 현실적인 해결책을 적용하는 것입니다. 프로젝트의 규모와 팀의 개발 방식에 따라 적절한 방법을 선택하는 것이 중요합니다.

양방향 의존성을 제거하고 단방향 구조를 유지하는 것은 코드 품질을 높이고 장기적인 유지보수 비용을 줄이는 핵심 원칙이므로, 이를 염두에 두고 개발해야 합니다.
