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

예를 들어 설명하겠습니다. 간혹 레이어드 아키텍처를 사용하는 조직에서는 편의상 하위 레이어에 있는 컴포넌트가 상위 레이어에 존재하는 모델을 이용하는 경우가 발생합니다. 다음은 하위 레이어인 서비스 계층에서 상위 레이어인 API 레이어의 모델에 접근하는 사례입니다.

```java
@Service
@RequiredArgsConstructor
public class PostService {
    private final CafeMemberJpaRepository cafeMemberJpaRepository;
    private final BoardJpaRepository boardJpaRepository;
    private final PostJpaRepository postJpaRepository;

    @Transactional
    public Post create(
        long cafeId,
        long boardId,
        long writerId,
        PostCreateRequest postCreateRequest) { // API 요청을 받는 모델인데 비즈니스 레이어에서 사용함
        long currentTimestamp = Instant.now().toEpochMilli();
        CafeMember cafeMember = cafeMemberJpaRepository
            .findByCafeIdAndUserId(cafeId, writerId)
            .orElseThrow(() -> new ForbiddenAccessException());
        User writer = userJpaRepository
            .findById(writerId)
            .orElseThrow(() -> new UserNotFoundException());
        Cafe cafe = cafeMember.getCafe();
        Board board = boardJpaRepository
            .findById(boardId)
            .orElseThrow(() -> new BoardNotFoundException());
        Post post = new Post();
        post.setTitle(postCreateRequest.getTitle());
        post.setContent(postCreateRequest.getContent());
        post.setCafe(cafe);
        post.setBoard(board);
        post.setWriter(writer);
        post.setCreatedTimestamp(currentTimestamp);
        post.setModifiedTimestamp(currentTimestamp);
        post = postJpaRepository.save(post);
        cafe.setNewPostTimestamp(currentTimestamp);
        cafe = cafeJpaRepository.save(cafe);
        return post;
    }
}
```

`PostCreateRequest` 클래스는 API 레이어의 모델입니다. API로 들어오는 요청을 `@RequestBody` 애너테이션을 이용해 매핑하려고 만든 객체인데, 하위 레이어에 존재하는 서비스 컴포넌트로 전달해 서비스에서 이를 사용하고 있는 상황입니다.

비즈니스 레이어에 위치한 서비스 컴포넌트가 프레젠테이션 레이어에 위치한 객체에 의존하는 바람에 두 레이어 간에 양방향 의존 관계가 생겼습니다. 이처럼 레이어 간에 양방향 의존성이 생긴 상황을 가리켜 ‘양방향 레이어드 아키텍처’라고 부릅니다. 이런 일이 발생해서는 안 됩니다. 왜냐하면 이렇게 될 경우 애써 정한 레이어의 역할이 의미가 없어지기 때문입니다. 다시 말해 계층이 무너집니다.

이것은 좋게 말해 양방향 의존이지만, 실은 순환 참조입니다. 그래서 레이어 간 양방향 의존성이 생겼다는 말은 아키텍처 수준에서 순환 참조가 생겼고, 분리된 레이어가 하나로 통합됐다는 선언과 같습니다. 그러므로 양방향 레이어드 아키텍처는 레이어드 아키텍처에서 가장 중요한 계층 관계가 사라진 상황이라고 표현할 수 있습니다.

양방향 레이어드 아키텍처에서 레이어는 더 이상 레이어라 부를 수 없습니다. 레이어가 컴포넌트를 구분하는 역할밖에 하지 못하기 때문입니다. 그러니 차라리 폴더라고 부르는 편이 나을 것입니다. 나아가 양방향 레이어드 아키텍처는 아키텍처라고 볼 수조차 없습니다. 왜냐하면 우리는 폴더에 맞춰 컴포넌트를 배치하는 것을 보고 아키텍처라고 부르지 않기 때문입니다.

그렇다면 레이어 간에 양방향 참조가 생겼을 때 이를 해결하는 방법은 무엇일까요? 다양한 방법이 있지만 여기서는 크게 두 가지 방법을 소개하겠습니다.

## 1. 레이어 별 모델 구성
첫 번째 해결 방법은 레이어별로 모델을 따로 만드는 것입니다. 이전 코드의 상황을 예시로 들자면 비즈니스 레이어에서 사용할 `PostCreateCommand` 모델을 추가로 만드는 것입니다. 이 모델은 프레젠테이션 레이어에 존재하는 `PostCreateRequest` 모델에 대응하는 모델입니다.

```java
@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class PostCreateRequest {
    private String title;
    private String content;
}
```

```java
@Data
@Builder
public class PostCreateCommand {
    private String title;
    private String content;
    private Long writerId;
}
```

### 명명 규칙
- `~Request` 클래스는 API 요청을 처리하는 모델입니다.
- `~Command` 클래스는 서비스에 생성, 수정, 삭제 요청을 보낼 때 사용하는 DTO입니다.

이처럼 `Request`와 `Command` 클래스를 구분하고 컨트롤러는 서비스에 요청을 보낼 때 `PostCreateRequest` 클래스를 `PostCreateCommand` 클래스로 변환한 후 서비스 메서드를 호출합니다. 이를 통해 의존 방향이 단방향이 되고 순환 참조가 사라집니다.

### 추가적인 장점
이 방법을 사용하면 API 요청 본문(Request Body)과 서비스 컴포넌트에서 사용하는 DTO를 분리할 수 있습니다. 예를 들어, `PostCreateRequest` 클래스에 `writerId` 라는 필드가 있다고 가정해 봅시다. 클라이언트가 보내는 이 값을 신뢰할 수 있을까요? 악의적인 사용자가 이를 조작하여 요청을 보낼 수 있기 때문에 신뢰할 수 없습니다. 따라서 `PostCreateRequest` 클래스에서는 `writerId` 같은 멤버 변수를 포함하지 않고, `PostCreateCommand` 클래스에서 이 값을 갖게 합니다.

```java
PostCreateCommand.builder()
    .title(postCreateRequest.getTitle())
    .content(postCreateRequest.getContent())
    .writerId(userPrincipal.getId())
    .build();
```

즉, `PostCreateCommand` 클래스에 필요한 정보를 반드시 `PostCreateRequest` 클래스에서 가져올 필요가 없습니다. 대신 `UserPrincipal` 같은 신뢰할 수 있는 객체에서 가져오도록 합니다. 이는 객체의 역할을 분리하는 효과를 가집니다.

### 단점
이 방식에도 단점이 있습니다. 대표적인 단점은 작성해야 하는 코드의 양이 늘어난다는 것입니다. 예를 들어, `Post` 클래스와 연관된 데이터 모델이 다음과 같이 늘어납니다.
- `Post`
- `PostCreateRequest`
- `PostCreateCommand`

만약 수정에 사용할 DTO까지 추가한다면 데이터 모델은 더욱 증가할 것입니다. 결과적으로 (CRUD를 위한 4개의 DTO * 레이어 개수)만큼 클래스가 추가될 수 있습니다.

이처럼 작성해야 하는 코드가 늘어나는 것은 조직 관점에서 비용 증가로 이어질 수 있습니다. 따라서 모델을 설계할 때는 적절한 균형을 유지하는 것이 중요합니다.

## 2. 공통 모듈 구성
두 번째로 소개할 방법은 공통으로 참조하는 코드를 별도의 모듈로 분리하는 것입니다. 다시 말해 모든 레이어가 단방향으로 참조하는 공통 모듈을 만들고, `PostCreateRequest` 클래스 같은 모델을 거기에 배치하는 것입니다.

예를 들어, `PostCreateRequest` 클래스를 `core` 패키지로 옮기고 모든 레이어가 이 `core`라는 모듈에 의존하도록 변경하면 어떻게 될까요?
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
