---
layout: post  
title: "깔끔하게 비즈니스 로직과 Swagger 관련 코드 분리하기"
author: "채지원"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["Swagger", "문서화"]
---

<br/>

### 들어가며

이번 팀 프로젝트를 진행하면서 `Swagger`라는 도구를 처음 써보게 되었습니다. 개발한 api들을 가독성 좋게 정리해서 보여주고, 테스트까지 해볼 수 있어서 만족스럽게 쓰고있어요😆

그러다가 api 호출 후 반환하는 response 형식이 무엇인지까지 작성할 수 있다는걸 알게 되었습니다!

저희 팀은 에러 코드를 노션으로 아래처럼 관리하고 있었는데, 변경 사항이 발생했을 때 관리하기도 어렵고 해당 에러 코드가 뭔지 한 눈에 확인하기 힘들었거든요.

<img width="738" alt="Image" src="https://github.com/user-attachments/assets/9a65935b-8a0a-4d24-bbcc-f824ef7291c1" />

그래서 기쁜 마음으로 아래와 같이 작성을 하려고 하는데…

![Image](https://github.com/user-attachments/assets/3203bcdc-7c16-48c3-9a58-b18ca569b0fc)

길다…

실제 controller 코드보다 훨씬 길어진 `ApiResponses` 가 가독성을 해치는 것 같았습니다.

_프론트엔드 개발자와 원활하게 소통하려면 자세한 문서를 적어야 할텐데 - 가독성이 너무 낮아진다 - 노션으로 적기에는 - 문서 관리가 힘들다…_ 의 고민을 반복하다가 이를 해결하기 위한 여러 방법이 있다는걸 알게 되었습니다. 오늘은 그 중 하나를 소개해 드릴게요.

### 스웨거 어노테이션 코드를 명세 전용 인터페이스 파일로 관리하기

- 형태

```docker
controller/
 ├── spec/
 │    ├── messageControllerSpec.java
 ├── messageController.java
 ...
```

- 예시 코드

```java
public interface MessageControllerSpec {

  @Tag(name = "문자", description = "문자 / 문자 템플릿 관련 API")

  @ApiResponses(
      value = {
          @ApiResponse(
              responseCode = "401",
              description = "유저의 JWT 토큰이 만료되어 재로그인이 필요한 경우",
              content = @Content(schema = @Schema(implementation = String.class))
          ),
          @ApiResponse(
              responseCode = "502",
              description = "외부 문자 발송 api 호출 중 에러가 발생한 경우",
              content = @Content(schema = @Schema(implementation = String.class))
          ),
          @ApiResponse(
              responseCode = "200",
              description = "정상 요청 됨",
              content = @Content(schema = @Schema(implementation = String.class))
          )
      }
  )
  ResponseEntity<String> sendMessage(@RequestBody List<SendMessageRequestDTO> request);
}
```

```java
@RestController
@RequestMapping("/api/v1/messages")
public class MessageController implements MessageControllerSpec {

  private final MessageService messageService;

  public MessageController(MessageService messageService) {
    this.messageService = messageService;
  }

  @PostMapping("")
  public ResponseEntity<String> sendMessage(@RequestBody List<SendMessageRequestDTO> request) {
    String result = messageService.sendMessage(request);
    return ResponseEntity.ok(result);
  }
}
```

위와 같은 형식으로 컨트롤러와 독립된 인터페이스 파일로 관리함으로써 실제 구현부 코드와 명세 코드를 분리할 수 있어요.

그 외 스웨거 어노테이션 코드를 각 yml로 파일로 분리하는 방법 등이 있지만, 이 방법이 가장 간단하고 가독성이 좋았습니다!

### 마치며

이 방법을 사용해서 Swagger 관련 코드와 비즈니스 로직을 깔끔하게 분리할 수 있었습니다! Swagger 문서화 작업을 보다 효율적으로 관리할 수 있어 팀원 간의 협업에도 큰 도움이 될 것 같아요.

문서화를 잘 해놓으면 프론트엔드 개발자와 소통하기에도 좋고, 기획자 등 비개발자가 API 사용법을 좀 더 이해할 수 있으니 신경써서 작성해보아요 📖
