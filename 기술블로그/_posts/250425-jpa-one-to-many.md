---
layout: post  
title: "JPA @OneToMany 삽질기: 분명 저장했는데 왜 Response에는 없을까? (feat. 양방향 연관관계)"
author: "정서연"
categories: "기술블로그"
banner:
  image: 썸네일로 넣고 싶은 이미지 링크
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`JPA`, `@OneToMany`, `Java`]
---


# JPA @OneToMany 삽질기: 분명 저장했는데 왜 Response에는 없을까? (feat. 양방향 연관관계)

안녕하세요! 최근 간단한 게시글(Article) 생성 API를 개발하면서 JPA 연관관계 매핑과 관련하여 꽤 오랜 시간 삽질을 경험이 있습니다. 게시글을 생성할 때 연관된 첨부 파일(ArticleFile)과 링크(ArticleLink) 정보도 함께 저장하는 기능이었는데, 분명 데이터베이스에는 정상적으로 데이터가 저장됨에도 불구하고, API 응답(Response)에서는 이 정보들이 누락되는 현상이 발생했습니다.

이번 포스팅에서는 제가 겪었던 문제 상황, 원인 분석 과정, 그리고 해결 방법을 공유하며 JPA의 양방향 연관관계 관리의 중요성에 대해 이야기해보고자 합니다.

## 문제 상황: DB엔 있지만 Response엔 없다?

먼저, 핵심 엔티티인 `Article`의 구조는 다음과 같습니다. `ArticleFile`과 `ArticleLink`와 `@OneToMany` 양방향 연관관계를 맺고 있습니다.

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class Article extends BaseEntity {

    private String title;

    private String content;

    @Enumerated(EnumType.STRING)
    private PriorityType priority;

    private LocalDateTime deadline;

    @Enumerated(EnumType.STRING)
    private ArticleStatus status;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "stage_id", nullable = false)
    private Stage stage;

    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private List<Comment> commentList = new ArrayList<>();

    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private List<ArticleFile> articleFileList = new ArrayList<>();

    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private List<ArticleLink> articleLinkList = new ArrayList<>();

    // 부모 게시글을 위한 필드 (답글이 부모 게시글을 참조)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_article_id")  // 부모 댓글을 참조하는 외래키
    private Article parentArticle;

    // 자식 게시글 리스트 (양방향 관계에서 부모 게시글이 자식 게시글을 가질 수 있게 설정)
    @OneToMany(mappedBy = "parentArticle", cascade = CascadeType.ALL)
    private List<Article> childArticles = new ArrayList<>();

    // 부모 게시글이 없으면 일반 게시글, 있으면 답글
    public boolean isChildComment() {
        return parentArticle != null;
    }

    @Builder
    public Article(String title, String content, PriorityType priority, LocalDateTime deadline, Member member, Stage stage, ArticleStatus status,
                   List<ArticleFile> articleFileList, List<ArticleLink> articleLinkList) {
        this.title = title;
        this.content = content;
        this.priority = priority;
        this.deadline = deadline;
        this.member = member;
        this.stage = stage;
        this.status = status;
        this.articleFileList = articleFileList != null ? articleFileList : new ArrayList<>();
        this.articleLinkList = articleLinkList != null ? articleLinkList : new ArrayList<>();
    }

}
```

문제가 발생했던 초기 서비스 코드의 로직은 다음과 같았습니다.

```java
@Transactional
public ArticleModifyResponse createArticle(ArticleModifyRequest request, UserDetailsImpl userDetails) {
    // ... (사용자, 스테이지 등 조회 로직 생략)

    // 1. Article 엔티티 생성 및 저장
    Article article = Article.builder()
            .title(request.getTitle())
            .content(request.getContent())
            // ... (다른 필드 설정)
            .member(member)
            .stage(stage)
            .status(ArticleStatus.PENDING)
            .build();

    article = articleRepository.save(article); // Article 영속화

    // 2. ArticleFile 처리 (존재하는 경우)
    if (request.getFileList() != null) {
        for (ArticleFileDTO fileDTO : request.getFileList()) {
            ArticleFile file = ArticleFile.builder()
                    .name(fileDTO.getName())
                    .url(fileDTO.getUrl())
                    .article(article)
                    .build();
            articleFileRepository.save(file); // ArticleFile 저장
        }
    }

    // 3. ArticleLink 처리 (존재하는 경우)
    if (request.getLinkList() != null) {
        for (ArticleLinkDTO linkDTO : request.getLinkList()) {
            ArticleLink link = ArticleLink.builder()
                    .urlAddress(linkDTO.getUrlAddress())
                    .urlDescription(linkDTO.getUrlDescription())
                    .article(article)
                    .build();
            articleLinkRepository.save(link); // ArticleLink 저장
        }
    }

    // 4. Response DTO 생성 및 반환
    return ArticleModifyResponse.fromEntity(article);
}
```

이 코드는 다음과 같은 순서로 동작합니다.

1.  `Article` 객체를 생성하고 `articleRepository.save()`를 호출하여 영속화합니다.
2.  요청에 `fileList`가 있으면, 각 `ArticleFileDTO`를 `ArticleFile` 엔티티로 변환합니다. 이때 `article` 필드에 1번에서 저장된 `article` 객체를 설정하여 외래 키 관계를 맺어줍니다. 그리고 `articleFileRepository.save()`를 호출하여 `ArticleFile`을 저장합니다.
3.  `linkList`에 대해서도 2번과 동일한 과정을 반복합니다.
4.  마지막으로, 영속화된 `article` 객체를 기반으로 `ArticleModifyResponse` DTO를 생성하여 반환합니다.

## 문제 발생

Postman으로 다음과 같은 요청을 보냈습니다.

```json
{
    "title": "Test Article with Links",
    "content": "Content here.",
    "priority": "MEDIUM",
    "deadLine": "2024-12-31T23:59:59",
    "stageId": 1,
    "linkList": [
        {
            "urlAddress": "https://example-link.com",
            "urlDescription": "Example Link 1"
        }
    ],
    "fileList": [
        {
            "name": "example.pdf",
            "url": "https://example-file.com/file.pdf"
        }
    ]
}
```

DB를 확인해보면 `article`, `article_file`, `article_link` 테이블에 데이터가 정상적으로 생성되고 외래 키도 잘 설정되어 있었습니다. 하지만 API 응답은 다음과 같았습니다.

```json
{
    "status": "success",
    "code": "200",
    "message": "게시글 생성 성공",
    "data": {
        "title": "Test Article with Links",
        "content": "Content here.",
        // ... (다른 필드)
        "stageId": 1,
        "fileList": [], // 비어 있음!
        "linkList": []  // 비어 있음!
    }
}
```

분명히 `ArticleFile`과 `ArticleLink`를 저장했는데, 왜 응답 DTO에는 `fileList`와 `linkList`가 빈 배열(`[]`)로 나올까요?

## 원인 분석: JPA 영속성 컨텍스트와 객체 상태

이 문제의 핵심은 **JPA의 영속성 컨텍스트(Persistence Context)와 객체 상태 관리 방식**에 대한 이해 부족이었습니다.

1.  `articleRepository.save(article)` 호출 시, `article` 객체는 영속성 컨텍스트에 의해 관리되는 **영속 상태(Managed State)**가 됩니다.
2.  이후 `articleFileRepository.save(file)`과 `articleLinkRepository.save(link)`를 호출하면, `file`과 `link` 객체도 영속 상태가 됩니다. 이때 `file.article`과 `link.article`에 설정된 `article` 객체를 참조하여 데이터베이스에 외래 키(`article_id`)가 올바르게 저장됩니다. **여기까지는 DB 관점에서는 문제가 없습니다.**
3.  **하지만,** `file`과 `link` 객체를 저장하는 행위가, **메모리 상에 있는 `article` 객체의 `articleFileList`나 `articleLinkList` 컬렉션을 자동으로 업데이트해주지는 않습니다.** JPA는 `@ManyToOne` (자식 -> 부모) 관계, 즉 연관관계의 주인(Owning Side) 쪽의 변경을 감지하여 외래 키를 관리하지만, `@OneToMany` (부모 -> 자식) 관계인 컬렉션 쪽(Inverse Side)은 개발자가 직접 동기화해주어야 할 때가 많습니다. 특히, 같은 트랜잭션 내에서 부모 객체를 저장한 *직후*에 자식 객체를 별도로 저장하고, 그 *직후*에 부모 객체의 컬렉션 정보를 사용해야 할 경우 이 문제가 두드러집니다.
4.  따라서 서비스 메서드의 마지막 단계에서 `ArticleModifyResponse.fromEntity(article)`를 호출할 때 사용되는 `article` 객체는, 1번에서 영속화된 상태 그대로이며, 이후에 저장된 `file`이나 `link` 정보가 `articleFileList`나 `articleLinkList`에 **메모리상으로 추가되지 않은 상태**입니다.
5.  결과적으로, `article` 객체의 컬렉션 필드들은 여전히 비어있는 `ArrayList`를 참조하고 있었고, 이것이 DTO로 변환되면서 빈 배열(`[]`)로 응답이 나갔던 것입니다. (`CascadeType.ALL`은 영속성 전이를 의미할 뿐, 메모리상 컬렉션 동기화를 보장하지 않습니다.)

## 해결 방법: 양방향 연관관계의 동기화

이 문제를 해결하기 위해서는 데이터베이스에 자식 엔티티를 저장할 때, **메모리 상의 부모 객체에도 해당 자식 객체를 명시적으로 추가**하여 객체 그래프의 상태를 일치시켜야 합니다.

수정된 서비스 코드는 다음과 같습니다.

```java
@Transactional
public ArticleModifyResponse createArticle(ArticleModifyRequest request, UserDetailsImpl userDetails) {
    // ... (사용자, 스테이지 등 조회 로직 생략)

    // 1. Article 엔티티 생성 및 저장
    Article article = Article.builder()
            .title(request.getTitle())
            .content(request.getContent())
            // ... (다른 필드 설정)
            .member(member)
            .stage(stage)
            .status(ArticleStatus.PENDING)
            .build();

    // Article 저장 (이 시점에서 article은 영속 상태가 됨)
    // save()를 먼저 호출할 필요는 없음. 어차피 트랜잭션 종료 시점에 변경 감지로 저장됨.
    // 하지만 명시적으로 ID를 얻거나 하고 싶다면 호출해도 무방. 여기서는 호출했다고 가정.
    article = articleRepository.save(article);

    // 2. ArticleFile 처리 (존재하는 경우)
    if (request.getFileList() != null) {
        for (ArticleFileDTO fileDTO : request.getFileList()) {
            ArticleFile file = ArticleFile.builder()
                    .name(fileDTO.getName())
                    .url(fileDTO.getUrl())
                    // .article(article) // 연관관계 편의 메서드에서 설정하므로 생략 가능
                    .build();
            // ArticleFile 저장 + Article의 List에도 추가 (양방향 동기화)
            // articleFileRepository.save(file); // CascadeType.ALL 이므로 부모 저장 시 함께 저장됨
            article.addArticleFile(file); // 연관관계 편의 메서드 사용!
        }
    }

    // 3. ArticleLink 처리 (존재하는 경우)
    if (request.getLinkList() != null) {
        for (ArticleLinkDTO linkDTO : request.getLinkList()) {
            ArticleLink link = ArticleLink.builder()
                    .urlAddress(linkDTO.getUrlAddress())
                    .urlDescription(linkDTO.getUrlDescription())
                    // .article(article) // 연관관계 편의 메서드에서 설정하므로 생략 가능
                    .build();
            // ArticleLink 저장 + Article의 List에도 추가 (양방향 동기화)
            // articleLinkRepository.save(link); // CascadeType.ALL 이므로 부모 저장 시 함께 저장됨
            article.addArticleLink(link); // 연관관계 편의 메서드 사용!
        }
    }

    // articleRepository.save(article); // CascadeType.ALL 이라면 이 시점에 자식들도 함께 저장됨
    // 만약 위에서 articleRepository.save()를 호출하지 않았다면,
    // 트랜잭션 종료 시점에 변경 감지(Dirty Checking)에 의해 article과 추가된 file, link 들이 저장됨.

    // 4. Response DTO 생성 및 반환
    // 이제 article 객체의 articleFileList와 articleLinkList는 메모리 상에서도 최신 상태임.
    return ArticleModifyResponse.fromEntity(article);
}
```

**핵심 변경 사항:**

1.  `ArticleFile` 또는 `ArticleLink` 객체를 생성한 후, `articleFileRepository.save(file)` 또는 `articleLinkRepository.save(link)`를 호출하는 대신 (또는 호출하더라도), **`article.getArticleFileList().add(file)`** 과 **`article.getArticleLinkList().add(link)`** 코드를 추가했습니다.
2.  (개선) `Article` 엔티티에 **연관관계 편의 메서드**(`addArticleFile`, `addArticleLink`)를 만들어 사용하는 것이 좋습니다. 이 메서드 내부에서 리스트에 자식 객체를 추가하고, 동시에 자식 객체에도 부모 객체(`this`)를 설정해줌으로써 양방향 관계를 한 번에 안전하게 설정할 수 있습니다.
3.  (개선) `CascadeType.ALL` (또는 `PERSIST`) 옵션이 설정되어 있다면, 부모 엔티티(`article`)가 영속화될 때 자식 엔티티(`file`, `link`)도 함께 영속화됩니다. 따라서 `articleFileRepository.save(file)` 이나 `articleLinkRepository.save(link)` 를 명시적으로 호출할 필요가 없습니다. 부모 객체에 자식 객체를 추가한 후, 트랜잭션이 커밋될 때 JPA가 알아서 처리해줍니다. (만약 `save()`를 호출한다면, 자식 객체에 부모 참조가 설정된 이후에 호출해야 합니다.)

이렇게 수정하고 API를 다시 호출하자, 드디어 원하던 대로 `fileList`와 `linkList`에 데이터가 포함된 응답을 받을 수 있었습니다.

```json
{
    "status": "success",
    "code": "200",
    "message": "게시글 생성 성공",
    "data": {
        "title": "Test Article with Links",
        "content": "Content here.",
        // ... (다른 필드)
        "stageId": 1,
        "fileList": [ // 정상적으로 포함됨!
            {
                "name": "example.pdf",
                "url": "https://example-file.com/file.pdf"
                // ... (ArticleFileDTO의 다른 필드)
            }
        ],
        "linkList": [ // 정상적으로 포함됨!
            {
                "urlAddress": "https://example-link.com",
                "urlDescription": "Example Link 1"
                // ... (ArticleLinkDTO의 다른 필드)
            }
        ]
    }
}
```

## 결론 및 교훈

이번 경험을 통해 JPA에서 양방향 연관관계를 다룰 때 다음과 같은 점을 명심해야 한다는 것을 배웠습니다.

1.  **객체 상태와 DB 상태의 동기화:** JPA는 영속성 컨텍스트를 통해 객체 상태를 관리합니다. 데이터베이스에 외래 키를 저장하는 것과 별개로, 메모리 상의 객체 그래프(특히 `@OneToMany` 컬렉션)는 개발자가 명시적으로 관리해주어야 할 때가 있습니다.
2.  **연관관계의 주인(Owning Side):** JPA는 일반적으로 `@ManyToOne` 쪽(외래 키를 가진 쪽)을 연관관계의 주인으로 보고, 이쪽의 변경을 기준으로 DB 업데이트를 수행합니다.
3.  **양방향 관계 설정의 책임:** 양방향 관계를 설정했을 때는, 연관관계의 주인뿐만 아니라 반대쪽(`@OneToMany` 컬렉션)에도 값을 설정해주어야 메모리 상의 객체 상태가 일관성을 유지합니다. 이를 위해 **연관관계 편의 메서드**를 적극 활용하는 것이 좋습니다.
4.  **CascadeType의 역할:** `CascadeType`은 영속성 전이에 관한 옵션으로, 부모 엔티티의 영속성 상태 변화(저장, 삭제 등)를 자식 엔티티에게 전파하는 역할을 합니다. 이것이 메모리 상의 컬렉션 동기화를 자동으로 보장하지는 않습니다.

JPA는 편리하지만, 내부 동작 원리(특히 영속성 컨텍스트와 객체 상태 관리)에 대한 이해가 부족하면 예상치 못한 문제에 직면할 수 있습니다. 이번 기회를 통해 JPA 양방향 매핑과 영속성 관리에 대해 더 깊이 학습해야겠다는 다짐을 하게 되었습니다.

혹시 비슷한 문제를 겪고 계신 분들께 이 글이 조금이나마 도움이 되었기를 바랍니다.

**참고 자료:**

*   (추천 영상) 김영한님의 JPA 강의 중 연관관계 매핑 파트
*   (추천 영상) [https://www.youtube.com/watch?v=brE0tYOV9jQ](https://www.youtube.com/watch?v=brE0tYOV9jQ) 

