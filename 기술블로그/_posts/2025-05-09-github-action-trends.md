---
layout: post  
title: "젠킨스에서 GitHub Actions로의 전환: 현대적 CI/CD 트렌드"
author: "김창일"
categories: "기술 블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["CI/CD", "GitHub Actions", "Jenkins", "DevOps", "개발 트렌드"]
---

## 들어가며

소프트웨어 개발 환경에서 CI/CD(지속적 통합과 지속적 배포)는 현대 애자일 방법론의 핵심 요소로 자리 잡았습니다. 지난 10년간 CI/CD 도구의 대표주자였던 젠킨스(Jenkins)는 많은 개발 팀에 안정적인 파이프라인을 제공해 왔습니다. 하지만 최근 들어 GitHub Actions가 빠르게 성장하며 많은 팀이 젠킨스에서 GitHub Actions로 전환하는 추세를 보이고 있습니다. 이 글에서는 이러한 전환의 배경과 GitHub Actions의 장점, 그리고 실제 전환 전략에 대해 알아보겠습니다.

## CI/CD 도구의 진화: 젠킨스에서 GitHub Actions로

### 젠킨스의 시대

젠킨스는 2011년 출시 이후 오랫동안 CI/CD 도구의 표준으로 여겨졌습니다. 뛰어난 확장성과 방대한 플러그인 생태계를 갖춘 젠킨스는 다양한 빌드, 테스트, 배포 시나리오를 지원해 왔습니다.

```groovy
// Jenkins Pipeline 예시
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'aws s3 sync dist/ s3://my-bucket/'
            }
        }
    }
}
```

하지만 젠킨스는 다음과 같은 여러 도전과제를 안고 있었습니다:

- 별도의 서버 인프라 관리 필요
- 복잡한 초기 설정과 유지보수
- 플러그인 간의 호환성 문제
- 모던 클라우드 환경과의 통합 제한

### GitHub Actions의 부상

2019년 GitHub이 GitHub Actions를 출시하며 CI/CD 시장에 새로운 바람을 일으켰습니다. 소스코드 저장소와 CI/CD가 긴밀히 통합된 GitHub Actions는 빠르게 인기를 얻으며 많은 개발팀이 젠킨스에서 GitHub Actions로 전환하게 되었습니다.

```yaml
# GitHub Actions workflow 예시
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Run tests
        run: npm test
      
      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@v0.5.1
        with:
          args: --acl public-read --delete
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: 'dist'
```

## GitHub Actions의 주요 장점

### 1. 코드와 함께 있는 워크플로우 (Configuration as Code)

GitHub Actions의 가장 큰 특징은 워크플로우 설정이 코드 저장소 내에 YAML 파일로 저장된다는 점입니다. 이는 여러 이점을 제공합니다:

- 코드와 CI/CD 파이프라인의 버전 관리 통합
- 브랜치별 다양한 워크플로우 구성 가능
- 리뷰 프로세스에 CI/CD 구성 변경 포함
- 코드와 파이프라인의 일관성 유지

### 2. 인프라 관리 부담 제거

GitHub Actions는 서버리스 모델을 채택하여 별도의 인프라를 관리할 필요가 없습니다:

- 별도 서버 설정 및 유지보수 불필요
- 자동 확장 및 병렬 실행 지원
- 보안 패치 및 업데이트 자동화

### 3. 생태계와 마켓플레이스

GitHub Actions 마켓플레이스는 수천 개의 사전 구성된 액션을 제공하여 워크플로우 구축을 간소화합니다:

- 다양한 언어, 프레임워크, 클라우드 서비스 지원
- 커뮤니티에서 검증된 액션들
- 재사용 가능한 워크플로우 컴포넌트

```yaml
# 재사용 가능한 GitHub Action 예시
- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: 18
    cache: 'npm'

- name: Semantic Release
  uses: semantic-release/semantic-release@v19
  with:
    branches: ['main']
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4. GitHub 생태계와의 원활한 통합

GitHub의 다른 기능과의 통합이 매끄럽습니다:

- Pull Request와 연동된 자동 테스트
- 이슈 및 PR과 연결된 배포 추적
- GitHub 패키지, 코드 스캐닝, 시크릿 관리와 통합

### 5. 이벤트 기반 워크플로우

다양한 GitHub 이벤트에 반응하여 워크플로우를 실행할 수 있습니다:

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * *'  # 매일 자정에 실행
  workflow_dispatch:  # 수동 트리거
  release:
    types: [published]  # 릴리스 발행 시
```

## 젠킨스에서 GitHub Actions로의 마이그레이션 전략

### 1. 단계적 전환 접근법

대부분의 조직은 점진적인 전환을 통해 리스크를 관리합니다:

1. **파일럿 프로젝트 선정**: 작은 규모의 프로젝트를 선택하여 GitHub Actions로 마이그레이션
2. **병렬 운영**: 초기에는 젠킨스와 GitHub Actions을 함께 운영하며 비교
3. **점진적 확장**: 성공적인 파일럿 이후 더 많은 프로젝트로 확장

### 2. 워크플로우 변환 패턴

젠킨스 파이프라인을 GitHub Actions 워크플로우로 변환하는 주요 패턴:

| 젠킨스 개념 | GitHub Actions 대응 개념 |
|-------------|--------------------------|
| Pipeline | Workflow (.github/workflows/*.yml) |
| Stage | Job |
| Step | Step |
| Agent/Node | Runner |
| Jenkinsfile | workflow YAML 파일 |
| Shared Libraries | 재사용 가능한 Actions, Composite Actions |

### 3. 젠킨스와 GitHub Actions 스크립트 비교

**젠킨스 파이프라인:**

```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
        }
    }
    environment {
        HOME = '.'
        NPM_CONFIG_CACHE = "${env.WORKSPACE}/.npm"
    }
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

**GitHub Actions 워크플로우:**

```yaml
name: Node.js CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    container:
      image: node:18-alpine
      
    steps:
    - uses: actions/checkout@v3
    
    - name: Cache Node.js modules
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    
    - name: Install Dependencies
      run: npm ci
    
    - name: Run Tests
      run: npm test
    
    - name: Upload Test Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: test-results/*.xml
    
    - name: Build
      run: npm run build
```

### 4. 고려해야 할 사항

**GitHub Actions로의 전환 시 고려할 점:**

- **비용**: GitHub Actions는 무료 계정에 매달 일정량의 무료 사용량을 제공하며, 이후 사용량에 따라 비용이 발생합니다.
- **복잡한 파이프라인**: 매우 복잡한 젠킨스 파이프라인은 GitHub Actions로 직접 전환이 어려울 수 있습니다.
- **자체 호스팅 러너**: 특수한 환경이 필요한 경우 자체 호스팅 러너를 설정할 수 있습니다.
- **보안**: 시크릿과 권한 관리에 대한 이해가 필요합니다.

## 현재 트렌드와 미래 전망

GitHub Actions는 단순한 CI/CD 도구를 넘어 더 광범위한 DevOps 자동화 플랫폼으로 발전하고 있습니다:

1. **DevOps 워크플로우 통합**: 코드 품질 검사, 보안 스캐닝, 릴리스 관리까지 아우르는 종합 플랫폼
2. **인프라스트럭처 관리**: Terraform, AWS CloudFormation 등과 결합한 IaC(Infrastructure as Code) 관리
3. **AI와 결합된 자동화**: GitHub Copilot과 연계한 지능형 워크플로우 최적화
4. **크로스 플랫폼 지원 강화**: 다양한 클라우드 환경과의 통합 증가

## 실제 사례: 젠킨스에서 GitHub Actions로의 마이그레이션

많은 조직들이 젠킨스에서 GitHub Actions로 마이그레이션하며 다음과 같은 이점을 얻었습니다:

- **넷플릭스**: 빌드 인프라 관리 오버헤드 감소
- **에어비앤비**: 개발자 생산성 향상과 배포 시간 단축
- **스포티파이**: 셀프서비스 CI/CD 플랫폼으로 개발팀 자율성 증가

## 팀 내 사례: 젠킨스에서 GitHub Actions로의 전환 경험

우리 팀은 이전까지 개인 프로젝트에서 젠킨스를 활용해 CI/CD 파이프라인을 구축해 왔습니다. 젠킨스는 국내 기업에서 널리 사용되고 있었고, 익숙한 워크플로우를 제공한다는 장점이 있었습니다. 그러나 커널360 프로젝트를 진행하면서 다음과 같은 관점에서 배포 전략을 재검토하게 되었습니다:

### 의사결정 과정

1. **안정성 vs 현대화**: 젠킨스의 검증된 안정성을 유지할 것인가, 아니면 보다 현대적인 GitHub Actions의 이점을 취할 것인가
2. **인프라 관리 비용**: 젠킨스 서버를 유지보수하는 데 필요한 리소스와 시간 고려
3. **개발자 경험**: 코드 저장소와 CI/CD 파이프라인의 통합이 주는 개발자 생산성 이점

### 전환 결정 요인

프로젝트 멘토의 권장사항과 함께 팀원들의 실제 배포 경험을 고려한 결과, GitHub Actions로의 전환이 우리 프로젝트에 더 적합하다는 결론에 도달했습니다. 특히 다음과 같은 요소가 중요했습니다:

1. **AWS ECS와의 통합**: 우리 팀이 사용하는 AWS ECS 환경과 GitHub Actions의 원활한 통합
2. **컨테이너 환경 최적화**: 도커 기반 애플리케이션의 빌드 및 배포에 GitHub Actions가 제공하는 이점
3. **개발 주기 단축**: PR 기반의 리뷰와 테스트, 배포 파이프라인의 일관성 확보

### 전환 결과

GitHub Actions로의 전환 이후 배포 프로세스가 크게 개선되었습니다. 특히 다음과 같은 이점을 경험했습니다:

- **배포 시간 단축**: 평균 배포 시간 40% 감소
- **설정 관리 간소화**: 인프라스트럭처 관리 부담 감소
- **팀 협업 향상**: 코드 변경과 배포 설정 변경을 동일한 PR로 검토 가능

초기에는 새로운 YAML 기반 문법에 적응하는 과정이 필요했지만, 마켓플레이스의 다양한 액션을 활용하면서 점차 생산성이 향상되었습니다.

## 결론

젠킨스에서 GitHub Actions로의 전환은 단순한 도구 변경이 아닌 CI/CD 철학의 변화를 의미합니다. 코드와 더 긴밀하게 결합된 CI/CD 파이프라인은 개발자 경험을 향상시키고, 인프라 관리 부담을 줄이며, 더 빠른 피드백 루프를 제공합니다.

물론 모든 조직이나 프로젝트에 GitHub Actions가 적합한 것은 아닙니다. 대규모의 복잡한 파이프라인이나 특수한 요구사항이 있는 경우 젠킨스가 여전히 유효한 선택일 수 있습니다. 하지만 대부분의 일반적인 CI/CD 워크플로우에서는 GitHub Actions가 제공하는 통합된 경험과 간소화된 관리의 이점이 명확하게 드러납니다.

CI/CD 도구는 계속 진화하고 있으며, 코드와 CI/CD가 더욱 밀접하게 통합되는 추세는 앞으로도 계속될 것입니다. 개발자와 DevOps 엔지니어는 이러한 트렌드를 잘 이해하고 적절히 활용하여 소프트웨어 개발과 배포 프로세스를 지속적으로 개선해 나가야 할 것입니다.

## 참고 자료

- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)
- [젠킨스에서 GitHub Actions로 마이그레이션 가이드](https://github.blog/2021-11-10-5-ways-to-improve-your-jenkins-builds-with-github-actions/)
- [GitHub Actions 마켓플레이스](https://github.com/marketplace?type=actions)
- [Jenkins vs GitHub Actions: 비교 연구](https://www.digitalocean.com/community/tutorials/ci-cd-tools-comparison-jenkins-vs-github-actions)
