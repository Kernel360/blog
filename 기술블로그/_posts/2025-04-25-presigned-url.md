---
layout: post
title: "Presigned URL을 통한 파일 업로드"
author: "김찬호"
categories: "기술블로그"
banner:
  image:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ "JAVA" ]
---

## 주제: Presigned URL을 통한 파일 업로드

### 1. Presigned URL방식이란
- Presigned URL은 서버에서 인증된 임시 업로드 링크를 생성해 클라이언트에 전달하고, 클라이언트가 직접 외부 스토리지(예: AWS S3)에 파일을 업로드할 수 있도록 해주는 방식입니다.

### 2. Vivim프로젝트에서 썼던 기존의 파일 업로드 방식

```java
@PostMapping(path = "/posts/{postId}/file/stream", consumes = MediaType.MULTIPART_FORM_DATA_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
@Operation(summary = "게시글에 파일 생성 및 S3 업로드")
public ResponseEntity<ApiResponse> createPostFile(@PathVariable Long postId,
  @RequestPart("file") MultipartFile file) throws IOException {
  metadata = new ObjectMetadata();
  metadata.setContentType(file.getContentType());
  metadata.setContentLength(file.getSize());

  bucketName = "vivim-s3";
  today = LocalDate.now().toString();
  uuid = UUID.randomUUID().toString();
  extension = getExtensionFromContentType(file.getContentType());
  objectKey = "uploads/" + today + "/" + uuid + extension;
  fileName = file.getOriginalFilename();

  amazonS3Client.putObject(bucketName, objectKey, file.getInputStream(), metadata);
  String fileUrl = amazonS3Client.getUrl(bucketName, objectKey).toString();

  fileService.createPostFile(fileName, fileUrl, file.getSize(), postId);
}
```

요약하면

1. 클라이언트가 multipart/form-data형태의 파일을 WAS가 전달받는다.

2. WAS가 S3에 파일을 InputStream 형태로 직접 업로드

3. DB에는 파일의 MetaData를 저장

이런 방식으로 구현이 됐었다.

처음엔 100KB이하의 파일들로만 테스트했기 때문에 문제가 발견되지 않았었다.

추후 대용량 파일 업로드, 다운로드를 하며 문제점을 발했다.

문제점

: 1번 문제: 100GB의 대용량 파일을 업로드, 다운로드 할 때 15초 정도 시간이 걸리는데 그 시간동안 서버가 멈추는 문제가 발생했다.(서버에 부하가 많이 감)

: 2번문제: 다운로드의 경우 클라이언트가 다운로드를 누르면 다운로드 진행상황이 보이지 않고 파일 다운이 완료됐을 때만 알림이 떴었다.



해결방법

: 기존의 방식은 서버에 부하를 많이 주기 때문에 Presigned URL방식으로 바꿨다.

### 3.Presigned URL을 적용한 방식

```java

@PostMapping(path = "/posts/{postId}/file/presigned", produces = MediaType.APPLICATION_JSON_VALUE)
@Operation(summary = "게시물에 파일 업로드용 PreSigned URL 생성")
public ResponseEntity<PreSignedUrlResponse> createPostFilePresignedUrl(
    @PathVariable Long postId,
    @RequestBody FileMetadataRequest fileMetadata) {

    // S3 경로 설정
    String bucketName = "vivim-s3";
    String today = LocalDate.now().toString();
    String uuid = UUID.randomUUID().toString();
    String extension = getExtensionFromContentType(fileMetadata.getContentType());
    String objectKey = "uploads/" + today + "/" + uuid + extension;

    // PreSigned URL 생성을 위한 메타데이터 설정
    GeneratePresignedUrlRequest generatePresignedUrlRequest = new GeneratePresignedUrlRequest(
        bucketName, objectKey)
        .withMethod(HttpMethod.PUT)
        .withExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 15)); // 15분 유효

    // Content-Type 설정
    generatePresignedUrlRequest.addRequestParameter(
        Headers.CONTENT_TYPE, fileMetadata.getContentType());

    // PreSigned URL 생성
    URL preSignedUrl = amazonS3Client.generatePresignedUrl(generatePresignedUrlRequest);
    String fileUrl = amazonS3Client.getUrl(bucketName, objectKey).toString();

    // DB에 파일 정보 저장 (실제 업로드는 클라이언트가 URL을 통해 직접 수행)
    fileService.createPostFile(fileMetadata.getFileName(), fileUrl, fileMetadata.getFileSize(),
        postId);

    // 클라이언트에게 PreSigned URL 반환
    PreSignedUrlResponse response = new PreSignedUrlResponse(
        preSignedUrl.toString(),
        fileUrl,
        objectKey
    );

    return ResponseEntity.ok(response);
}
```

Presigned URL방식을 요약하면

1. 서버는 파일 업로드용 S3 URL만 생성 & DB에 파일의 MetaData저장

2. 클라이언트가 해당 URL로 직접 S3에 PUT 요청하여 파일 업로드


물론 Presigned URL 방식도 단점이 있다.

1. 업로드 실패를 서버가 감지를 못하고 클라이언트가 전송을 해줘야한다.

2. Presigned URL을 가진 사용자는 누구든지 업로드, 다운로드가 가능하다.

-> 때문에 Presigned URL의 유효시간을 15분 등 짧게 두어야 한다.
