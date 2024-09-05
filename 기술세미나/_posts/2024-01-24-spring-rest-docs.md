---
layout: post
title: 이번엔 Spring REST Docs를 써볼까?
author: 김영롱
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0124/spring-rest-docs.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [API 문서, Spring REST Docs, 기술세미나]
---
안녕하세요, *Lune* 입니다.

이번엔 **Spring REST Docs**를 기술 세미나 주제로 가져와봤는데요.

Spring REST Docs를 프로젝트에서 사용하게 된 이유부터 적용기까지 공유해보려고 합니다.

## 1. API 문서의 필요성
조금 뻔한 얘기로 시작해볼까요?<br>
개발을 하면서 API 문서들이 필요한 이유는 무엇일까요? 제가 생각해봤을 때, 그리고 검색을 해봤을 때 아래와 같은 이유들이 있었습니다.

1. API 사용법 이해
2. 개발자 간의 효율적인 협업
3. 빠른 문제 해결 및 버그 대응
4. 개발 생산성 향상

개인적으로는 일단 2번이 제일 와닿습니다. 협업할 때 같은 API 문서를 보며 얘기하면 소통이 더 잘된다고 느낀 적이 많았거든요. 

## 2. API 문서를 만드는 여러 방법들
자, 그럼 이번에는 API 문서를 만들 수 있는 방법들에 대해 이야기해볼게요.

1. 구글 공유 문서 (Docs / Sheets)
2. Notion
3. GitBook
4. Swagger
5. Spring REST Docs

위에 리스트 말고도 더 많은 방법들이 있겠지만, 제가 사용해봤거나 사용사례를 본 적 있는 것 위주로 작성해봤습니다.

### 구글 공유 문서 / Notion / GitBook
코드베이스가 아니라 수동으로 직접 문서를 작성하는 방식입니다. 장단점을 알아보자면 다음과 같습니다.

 장점              | 단점               
-----------------|------------------
 쉬운 협업 및 공유      | API 문서 작성 기능 제한적 
 커스텀 스타일 적용 쉬운 편 | 비용 문제 발생 가능      
 사용자 친화적인 UI/UX  | API 문서 자동화 어려움   

위 내용이 3가지 모두에 동일하게 적용된다고 할 수는 없지만, 보편적인 장단점 위주로 설명해보려고 했으니 조금 안 맞다고 생각이 들어도 이해해주세요 👐

### Swagger / Spring REST Docs
코드 내에 API 문서화를 위한 작업을 진행하는 방식입니다. 장단점을 알아보자면 다음과 같습니다.

 장점             | 단점              
----------------|-----------------
 자동 문서 생성       | 러닝커브 존재 및 설정 복잡 
 일관된 형식과 스타일 유지 | 커스터마이제이션 한계     
 실시간 업데이트       | 의존성 및 업그레이드 어려움 

여기서 *커스터마이제이션 한계* 란 문서의 외관이나 기능을 개발자가 원하는 대로 조정하는 데에 한계가 있다는 것을 의미합니다.

## 3. Spring REST Docs를 선택한 이유
현재 진행 중인 프로젝트에서는 왜 Spring REST Docs를 선택하게 되었을까요?

우선 첫 시작은 GitBook 이었답니다. 기능 개발이 들어가지 않은 상태에서 프론트엔드 개발자와의 빠른 협업을 위해 GitBook으로 API 문서를 만들기 시작했었죠. 그런데 초반에 잘 알아보지 않은 탓에 얼마 지나지 않아 무료 버전의 한계를 맞닥뜨리게 되었습니다. 무료 버전에서는 여러 사람이 문서 편집을 할 수 없어 공동 작업이 더 이상 불가능하게 되어버렸거든요 🥲

그 다음으로 생각한 건 API 문서 자동화가 가능한 Swagger와 Spring REST Docs였습니다. 그래서 그 둘을 비교해봤습니다.

### Swagger

 장점             | 단점                        
----------------|---------------------------
 어노테이션 기반 문서 생성 | 프로덕션 코드에 작업 필요            
 화면에서 API 테스트   | 프로덕션 코드와 동기화 안 되어 있을 수 있음 
 비교적 쉬운 적용      |

### Spring REST Docs

 장점              | 단점            
-----------------|---------------
 프로덕션 코드에 영향 없음  | 적용이 어려운 편     
 테스트 성공 시 문서 생성  | 테스트 코드 양이 늘어남 
 API 문서 최신 상태 유지 |

(제목에 드러나있듯이) 결과적으로 Spring REST Docs를 사용하기로 결정을 했습니다. 이번 프로젝트에서는 테스트 코드를 작성하기로 했었고, 프로덕션 코드에 영향이 없고 늘 현행화가 가능하다는 점에 이끌렸거든요.

## 4. Spring REST Docs 적용기
이제부터는 Spring REST Docs를 적용했던 과정을 하나씩 설명해보겠습니다.<br>
참고로 저는 Asciidoctor & MockMvc 방식을 사용했는데요. 공식 문서에 따르면 문서 작성을 위해 Asciidoctor와 Markdown을 지원하고 있고 MockMvc, WebTestClient, REST Assured 방식의 테스트 예시를 보여주고 있습니다.

### 1) build.gradle 설정
build.gradle에 추가되어야 하는 내용 및 설명은 아래 코드로 대체합니다. 
```groovy
plugins {
  // Asciidoctor 플러그인 적용
  id "org.asciidoctor.jvm.convert" version "3.3.2"
}

ext {
  // 생성된 snippets 출력 위치를 정의하는 snippetsDir 속성을 구성
  snippetsDir = file('build/generated-snippets')
}

configurations {
  // asciidoctorExt 구성을 선언
  asciidoctorExt
}

dependencies {
  // asciidoctorExt 구성에서 spring-restdocs-asciidoctor에 대한 종속성을 추가
  asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
  // MockMvc 테스트 방식을 사용
  testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}

tasks.named('test') {
  // 테스트 작업을 실행하면 출력이 snippetsDir에 기록된다는 것을 Gradle이 인식하도록 함
  outputs.dir snippetsDir
}

asciidoctor {
  // asciidoctor 작업을 구성
  dependsOn test
  configurations 'asciidoctorExt'
  baseDirFollowsSourceFile()
  inputs.dir snippetsDir
}

asciidoctor.doFirst {
  // asciidoctor 작업이 수행될 때 가장 먼저 수행
  delete file('src/main/resources/static/docs')
}

task copyDocument(type: Copy) {
  // html 파일 복사
  dependsOn asciidoctor
  from file("build/docs/asciidoc")
  into file("src/main/resources/static/docs")
}

bootJar {
  dependsOn copyDocument
}
```

### 2) 테스트 코드 작성
첫 번째 *테스트 코드* 외에는 구현 방식에 따라 다를 수 있으므로 가볍게 봐주시면 됩니다.
```java
/** 테스트 코드 **/
result.andExpect(status().isOk())
      .andDo(document(
              "commoncode/get-common-codes",
              getDocumentRequest(),
              getDocumentResponse(),
              // pathParameters, queryParameters, requestFields, responseFields는 필요 시 각각 작성
              pathParameters(
                      parameterWithName("codeName").description("코드명")
              ),
              responseFields(beneathPath("value").withSubsectionId("value"),
                      fieldWithPath("codeNo").type(JsonFieldType.NUMBER).description("코드번호"),
                      fieldWithPath("codeName").type(JsonFieldType.STRING).description("코드명"),
                      fieldWithPath("description").type(JsonFieldType.STRING).description("설명").optional()
              )
));
```
```java
/** Utils 만들어 사용 **/
public interface RestDocumentUtils {

  static OperationRequestPreprocessor getDocumentRequest() {
    return preprocessRequest(modifyUris().scheme("http")
                                         .host("spring.restdocs.test") // 실제 host 아님
                                         .removePort(), prettyPrint());
  }

  static OperationResponsePreprocessor getDocumentResponse() {
    return preprocessResponse(prettyPrint());
  }
}
```
```java
/** 추상 클래스 작성 **/
@WebMvcTest({
  CommonCodeController.class,
})
@AutoConfigureRestDocs  // 통합 테스트에서 Spring REST Docs를 활성화하고 구성하는 데 사용
public abstract class ControllerTest {

  @Autowired
  protected MockMvc mockMvc;

  @Autowired
  protected ObjectMapper objectMapper;

  @MockBean
  protected CommonCodeService commonCodeService;
}

// 아래와 같이 상속받아 사용
// class CommonCodeControllerRestDocsTest extends ControllerTest {
```

### 3) 테스트 성공
위와 같이 작성한 테스트 코드가 통과하게 된다면 build/generated-snippets 경로 하위에 adoc 확장자 파일들이 여러 개 생성된 것을 확인할 수 있습니다. 파일을 하나씩 선택해서 보면 Asciidoc 문법에 맞게 작성된 내용과 미리보기를 확인할 수 있답니다.

![gradle-build](https://github.com/Kernel360/blog-image/blob/main/2024/0124/gradle-build.png?raw=true)

### 4) API 문서 틀 작성
위에서 본 문서 조각들(snippets)을 그대로 활용할 수 있다면 좋겠지만, HTML 형태의 API 문서로 만들어 주기 위해서는 아직 추가적인 작업이 남아있습니다. 한 데 모아주는 작업을 해줘야 하는데요. 우선 저는 *index.adoc* 이라는 파일을 만들어 API별 adoc 파일을 include 하는 구조를 사용했습니다. 참고로 index.adoc 파일은 *src/docs/asciidoc 하위* 에 만들어주었는데 이 경로가 gradle을 사용할 경우의 기본 경로랍니다.

```asciidoc
// index.adoc
= API Document
// Asciidoc 문서의 구조와 스타일 정의
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

// adoc 파일 include
include::overview.adoc[]
include::sample-api.adoc[]
```
```asciidoc
// sample-api.adoc
== 샘플 API

[[공통코드-조회]]
=== 공통코드 조회

==== Request
include::{snippets}/commoncode/get-common-codes/path-parameters.adoc[]

===== HTTP Request 예시
include::{snippets}/commoncode/get-common-codes/http-request.adoc[]

==== Response
include::{snippets}/commoncode/get-common-codes/response-fields-value.adoc[]

===== HTTP Response 예시
include::{snippets}/commoncode/get-common-codes/http-response.adoc[]
```

### 5) API 문서 생성
이제 거의 마무리 단계인데요. gradle의 bootJar 작업을 실행시키면 build.gradle에 작성한 copyDocument 작업을 거쳐서 미리 지정한 resources/static/docs 경로 하위에 build 내에 생성되어 있던 html 파일이 복사되어 들어오고, 서버를 띄웠을 때 도메인 하위 /docs/index.html 경로로 접속해 API 문서를 확인할 수 있게 된답니다.

![api-doc](https://github.com/Kernel360/blog-image/blob/main/2024/0124/api-doc.png?raw=true)

### 6) API 문서 살펴보기
최종적으로 만들어진 문서의 형태는 다음과 같습니다. 좌측에 ToC가 있어 원하는 API로의 이동이 간편하고 화면도 깔끔하지 않나요?

![index-html](https://github.com/Kernel360/blog-image/blob/main/2024/0124/index-html.png?raw=true)

## 5. 정리
지금까지 저와 함께 API 문서부터 Spring REST Docs 적용기까지 살펴봤는데요. 읽기에 어떠셨을지 궁금합니다 😄

Spring REST Docs를 적용하면서 여러 차례 시행착오를 거치며 보낸 시간이 생각나네요. 처음 접하는 부분이 많아 공식 문서, 영상, 블로그 등을 참고했는데 그 과정에서 조금씩 이해하게 되고 원하는 결과를 만들어낼 수 있어서 개인적으로 뿌듯하고 좋았던 경험이었습니다.

아 그리고 이번 세미나를 준비하면서 추가적인 정보를 찾다가 알게 된 건데 Swagger와 Spring REST Docs의 장점만 취해서 API 문서화를 할 수 있는 방법도 있다고 해요. 가능하다면 다음에는 이 방식을 사용하거나 시간이 될 때 변경해볼 수 있으면 좋을 것 같다는 생각이 드네요.

## 6. 참고자료
- [Spring REST Docs](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/)
- [Spring REST Docs에 날개를… (feat: Popup)](https://techblog.woowahan.com/2678)
- [Spring REST Docs를 사용한 API 문서 자동화](https://hudi.blog/spring-rest-docs)
- [RestDocs로 API 문서화하기](https://velog.io/@junho5336/RestDocs%EB%A1%9C-API-%EB%AC%B8%EC%84%9C%ED%99%94%ED%95%98%EA%B8%B0)
- [[ 스프링 기반 REST API ] 스프링 REST Docs 소개](https://youtu.be/BFme2t9KSwA?si=ziyQ3jC1l-tQ57Md)
- [[10분 테코톡] 승팡, 케이의 REST Docs](https://youtu.be/BoVpTSsTuVQ?si=VG3mhS5b32l1EY_s)
- [[Spring] restdocs + swagger 같이 사용하기](https://velog.io/@hwsa1004/Spring-restdocs-swagger-%EA%B0%99%EC%9D%B4-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
