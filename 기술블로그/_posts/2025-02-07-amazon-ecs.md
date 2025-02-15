---
layout: post  
title: "Amazon ECS(Elastic Container Service)을 활용한 운영 환경 구성"  
author: "김진우"  
categories: "백엔드 기술블로그"  
banner:  
  background: "#000"  
  height: "100vh"  
  min_height: "38vh"  
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"  
  tags: ["AWS", "ECS", "ECR", "Docker", "CI/CD", "Jenkins"]
---

# Amazon ECS를 활용한 무중단 배포 파이프라인 구축

저희는 운영 환경에 **Amazon ECS(Amazon Elastic Container Service)**를 도입하여 컨테이너 오케스트레이션을 활용하고 있습니다. 또한, **GitHub와 Jenkins를 연동**하여 CI/CD 파이프라인을 구축함으로써 효율적인 배포 환경을 구성했습니다.

이 글에서는 **GitHub에서 메인 브랜치에 코드가 머지되었을 때 ECS가 최신 이미지를 자동으로 가져와 롤링 업데이트 방식으로 무중단 배포되는 과정**과 **문제 발생 시 롤백 전략**을 설명합니다.

---

## 1. ECS 기반 CI/CD 파이프라인 개요

### 🔹 전체 배포 흐름

1. **GitHub에서 코드가 메인 브랜치에 머지**되면 Jenkins가 파이프라인을 실행합니다.
2. Jenkins에서 **Docker로 애플리케이션을 빌드**합니다.
3. 빌드된 Docker 이미지를 **Amazon Elastic Container Registry(ECR)**에 Push합니다.
4. ECS는 **ECR에 등록된 최신 이미지를 가져와 실행**합니다.
5. **Application Load Balancer(ALB)를 이용해 롤링 업데이트 방식으로 무중단 배포**를 진행합니다.
6. **문제 발생 시 자동 롤백 전략**을 적용합니다.

---

## 2. Jenkins와 ECS를 활용한 배포 과정

### 🏗 1. Docker로 애플리케이션 빌드
```sh
# Jenkins에서 실행될 Docker 빌드 명령어
docker build -t my-app:latest .
```

### 📌 2. ECR에 Docker 이미지 Push
```sh
# AWS ECR 로그인 (Jenkins에서 수행)
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com

# ECR에 태그 및 Push
docker tag my-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com/my-app:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com/my-app:latest
```

### 🚀 3. ECS에서 최신 이미지 가져오기
ECS 서비스는 태스크 정의(Task Definition)의 컨테이너 이미지를 **ECR에서 가져와 업데이트**합니다.
```sh
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
```

### 🔄 4. 롤링 업데이트 및 롤백 전략
ECS는 **ALB(Application Load Balancer)**를 활용하여 트래픽을 점진적으로 신규 컨테이너로 전환하는 **롤링 업데이트(Rolling Update)** 방식으로 무중단 배포를 수행합니다.
- 새로운 컨테이너가 실행되면 ALB는 헬스 체크(Health Check)를 수행합니다.
- 정상 상태가 확인되면 기존 컨테이너에서 새로운 컨테이너로 트래픽을 이동합니다.
- 기존 컨테이너는 종료됩니다.

#### 🚨 롤백 전략
만약 배포 중 문제가 발생하면 **자동 롤백**이 적용되어 기존 안정적인 상태로 복구할 수 있습니다.
- ALB 헬스 체크가 실패할 경우 자동으로 이전 이미지로 복구합니다.
- Jenkins 파이프라인에서도 **빌드 실패 또는 헬스 체크 실패 시 롤백 명령**을 실행하도록 구성합니다.

```sh
# ECS 서비스 롤백 예시
aws ecs update-service --cluster my-cluster --service my-service --desired-count 2 --rollback-deployment
```

---

## 3. ECS CI/CD 파이프라인 구성의 장단점

### ✅ 장점
- **자동화된 배포**: GitHub, Jenkins, ECS, ECR을 연동하여 **CI/CD 파이프라인을 구축**함으로써 코드 변경 사항이 자동으로 배포됩니다.
- **무중단 배포**: ALB와 롤링 업데이트를 활용하여 **배포 중에도 애플리케이션이 정상 동작**합니다.
- **효율적인 리소스 관리**: Fargate 기반 ECS를 사용하면 **서버 관리 없이 컨테이너 실행**이 가능하며, EC2 기반 ECS를 사용하면 **비용 최적화 및 유연한 확장**이 가능합니다.

### ❌ 단점
- **비용 문제**: Fargate는 서버 관리가 필요 없다는 장점이 있지만, **EC2 기반 ECS보다 비용이 높을 수 있습니다**. 현업에서는 비용 최적화를 위해 EC2 기반으로 운영하는 경우가 많습니다.
- **제한적인 커스터마이징**: ECS는 Kubernetes(EKS)보다 커스터마이징 옵션이 제한적이며, Kubernetes의 풍부한 생태계를 활용할 수 없습니다.

---

## 4. 결론

Amazon ECS와 CI/CD 파이프라인을 활용하면 **자동화된 컨테이너 배포 환경을 구축**할 수 있습니다. 특히 **GitHub, Jenkins, ECR, ECS, ALB를 조합**하면 코드 변경 사항을 신속하고 안전하게 배포할 수 있습니다.

운영 중인 환경에서 **배포 자동화를 고민하고 있다면, 무중단 배포가 가능한 롤링 업데이트 방식의 CI/CD 파이프라인을 구축**하는 것을 추천드립니다. 단, **비용과 커스터마이징 요구사항을 고려**하여 ECS와 EKS 중 적합한 솔루션을 선택하는 것이 중요합니다.

> **🔎 참고**: Amazon ECS 공식 문서를 확인하려면 [AWS ECS 공식 문서](https://docs.aws.amazon.com/ecs/)를 참고하세요.
---

