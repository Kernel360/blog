---
layout: post  
title: "GitHub actions"
author: "정인재"
categories: "기술 블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["기술블로그"]
---

# GitHub Actions + ECR + EventBridge + Lambda를 활용한 무중단 ECS 배포 자동화

## 📝 개요

운영 중인 실시간 차량 관제 시스템의 백어드 서버를 무중단으로 배포하기 위해, CI/CD 파이프라인을 재설계했습니다. 기존에는 GitHub Actions에서 직접 ECS로 배포 요청을 보내는 방식이었지만, 이를 AWS 서비스 중심의 이벤트 기반 구조로 전환해서 안정성과 확장성을 개정했습니다.

이번 글에서는 **GitHub Actions → Docker Build & Push to ECR → EventBridge → Lambda → ECS Force Deploy** 순서로 이루어진 자동화 배포 파이프라인을 구성하는 방법을 소개합니다.

---

## 🔧 사용한 기술 스택

* GitHub Actions: CI 파이프라인 실행
* Amazon ECR: Docker 이미지 저장소
* Amazon EventBridge: 이벤트 라우트팅
* AWS Lambda: 배포 트리거 로직 실행
* Amazon ECS (Fargate): 서비스 호스팅

---

## 📌 전체 아키텍쳐

```
[ GitHub Actions ]
        |
        v
[ Docker Build & Push to ECR ]
        |
        v
[ EventBridge (on ECR push) ]
        |
        v
[ Lambda (ECS forceNewDeployment 실행) ]
        |
        v
[ ECS 서비스 무중단 재배포 ]
```

---

## 1️⃣ GitHub Actions: 이미지 빌드 & ECR 푸시

`.github/workflows/deploy.yml` 예시:

```yaml
name: CI - Build and Push to ECR

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-northeast-2

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build and push Docker image
      run: |
        IMAGE_URI=${{ secrets.ECR_REPO }}:latest
        docker build -t $IMAGE_URI .
        docker push $IMAGE_URI
```

> 💡 GitHub에서는 ECS 접근 권한 없이, ECR까지의 권한만 설정합니다.

---

## 2️⃣ EventBridge: ECR 이미지 푸시 이벤트 감지

ECR에서 이미지가 푸시되면 자동으로 이벤트가 발생하며 EventBridge가 이를 감지합니다.

### 이벤트 패턴 예시:

```json
{
  "source": ["aws.ecr"],
  "detail-type": ["ECR Image Action"],
  "detail": {
    "action-type": ["PUSH"],
    "repository-name": ["where-car-backend"],
    "image-tag": ["latest"]
  }
}
```

> 📌 위 이벤트에 매칭되면 Lambda가 자동 실행됩니다.

---

## 3️⃣ Lambda: ECS 서비스 재배포

EventBridge로부터 트리거된 Lambda 함수는 ECS 서비스에 강제 배포 요청을 보내됩니다.

### Lambda 함수 코드 (Python, Boto3):

```python
import boto3
import os

def lambda_handler(event, context):
    ecs = boto3.client('ecs')
    
    cluster = os.environ['CLUSTER_NAME']
    service = os.environ['SERVICE_NAME']

    print(f"Triggering ECS deployment for service: {service}")

    response = ecs.update_service(
        cluster=cluster,
        service=service,
        forceNewDeployment=True
    )

    return {
        'statusCode': 200,
        'body': 'ECS force deployment triggered successfully'
    }
```

### 환경 변수 설정:

* `CLUSTER_NAME=where-car`
* `SERVICE_NAME=rest-service`

---

## ✅ 결과

* `main` 브랜치에 push → GitHub Actions가 Docker 이미지 빌드 후 ECR에 푸시
* ECR 푸시 이벤트 감지 → EventBridge가 Lambda 실행
* Lambda가 ECS에 `forceNewDeployment` 요청
* ECS가 무중단으로 새 탭스크를 실행하고 서비스 반영

---

## 💡 이 방식의 장점

| 항목       | 개정 내용                                              |
| -------- | -------------------------------------------------- |
| 역할 분리    | GitHub Actions는 ECR까지만 접근, ECS는 AWS 내부에서만 제어됨      |
| 확장성      | Lambda에 알림(Slack, SNS), 롤박 처리, 헬스체크 등 다양한 기능 추가 가능 |
| 보안 강화    | IAM 최소 권한 원칙 적용, 권한 분리로 안정성 확장                     |
| 유지보수 편의성 | EventBridge 조건만 바꾼면 서비스 분기 및 버전 관리에 유용하게 대응 가능     |

---

## 📌 참고 링크

* [GitHub Actions Docs](https://docs.github.com/en/actions)
* [Amazon ECR Events](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr-eventbridge.html)
* [Boto3 ECS Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ecs.html)
* [EventBridge Docs](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html)

---

## 📌 마무리

이 구조는 **GitHub Actions는 오직 빌드와 이미지 푸시 역할만 수행하고**, 그 이후 모든 배포 로직은 **AWS 내부 이벤트로 처리**하게 됩니다.
덕분에 보안은 무료, 운영 확장성 참명적으로 많이 유니티화되고, Slack 알림/배포 실패 처리 등 다양한 후처리도 유연히 연동할 수 있습니다.

> 🚀 ECR을 중심으로 한 이벤트 기반 배포 환경은 아프토로도 가장 선호되는 구조가 될 것입니다.
