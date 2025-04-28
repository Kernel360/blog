---
layout: post
title: 젠킨스 레포지토리 빌드와 배포하기
author: 김창일
banner:
  image: assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/featured.jpg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ci/cd, 기술세미나]
---

## 젠킨스 배포해보기
커널 360 여러분 만나서 반갑습니다.
저희는 이제부터 젠킨스에 직접 스프링을 배포하는 일련의
데브옵스 작업을 해보고자 합니다.

<br/><br/><br/>

### 시작하기 전에
이미 서비스가 운영중인 개인 온프레미스 서버에서 미리 설정된 서버와 프로그램을 활용합니다. 
추후에 따로 다양한 프로그램 설치 및 설정에 대해 다룰 예정입니다.

<br/><br/><br/>

## 시작하기

### 깃허브 레포지토리 포크하기
준비된 레포지토리를 본인의 계정으로 포크합니다.

{{< github repo="laterre39/spring-jenkins-template" >}}

### 프로젝트 수정하기
포크한 레포지토리를 빌드하기 전에 문제없는 배포를 위해 스프링 부트의 설정을 수정해야 합니다.

- application.yml
    - port: 지정된 포트로 변경해주세요 (ex: 8090)
    - contextPath: 지정된 id를 입력해주세요 루트주소에 추가 주소를 설정합니다.

- settings.gradle
    - rootProject.name: 프로젝트 이름을 변경해서 빌드 파일명을 중복되지 않게 설정합니다.


### 젠킨스 관리자 접속하기
부트캠프에 참가하셨다면 관리자에 의해 계정이 발급 되셨을겁니다.  
사이트에 접속하셔서 발급받은 계정을 입력해서 로그인 합니다.

![젠킨스 로그인 페이지](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-01.png)

발급받은 계정으로 로그인에 성공하면 아래와 같은 페이지에 접속이 됩니다.

![젠킨스 메인 페이지](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-02.png)

관리자에 의해 설정된 권한에 의해 수정할 수 있도록 할당받은 프로젝트만 수정하고 빌드하실 수 있습니다.  

<br/><br/><br/>

## 프로젝트 기본 설정하기
프로젝트 이름을 눌러서 프로젝트에 진입해보겠습니다.  
설정된 프로젝트 이름은 다음과 같습니다.

```md
{username}-project
```

프로젝트에 진입하면 프로젝트 메인 대시보드를 확인할 수 있습니다.
해당 프로젝트 대시보드에서 프로젝트에 대한 CICD 설정을 진행합니다.

![젠킨스 프로젝트 메인 페이지](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-03.png)

좌측 사이드 메뉴에서 `구성`을 선택해서 설정 페이지로 이동합니다.
여기서부터 설명을 자세히 보시고 따라하시거나 또는 응용해서 설정해보세요!

![젠킨스 프로젝트 설정 페이지](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-04.png)

### 설정 주의하기
{{< alert >}}
**경고!** **Enable project-based security**에 대한 **설정**을 절대 건들지 마세요!!!
{{< /alert >}}

![Enable project-based security 설정](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-05.png)

### 레포지토리 입력
포크한 [탬플릿 레포지토리](#깃허브-레포지토리-포크하기) URL을 복사해서 GitHub project 탭을 체크해서 입력합니다. 

![GitHub project 설정](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-06.png)

### JDK 선택
미리 만들어둔 탬플릿 레포지토리의 경우 JDK 17버전을 기반으로 개발되었습니다.  
그래서 `openJDK-17` 버전을 선택합니다.

![openJDK 17 선택](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-07.png)

#### JDK 선택과 관련해서
실제로 개발 환경에 따라 다양한 JDK 버전을 사용하기 때문에 젠킨스에서 지원해서 활용할 수 있도록 제공하고 있습니다.  
그 이외에도 다양한 프로덕션 환경에 대응하도록 방대한 설정을 플러그인 및 빌트인 시스템으로 제공하고 있습니다.

<br/><br/><br/>

## 소스코드 관리 설정하기
해당 설정탭에서 레포지토리에 대한 기본 설정을 진행합니다.  
중요하니까 잘 따라서 설정해주세요

![소스 코드 관리](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-08.png)

### Git 설정하기
위에서 설정한 레포지토리 입력과 동일하게 포크한 레포지토리를 `Repositories`에 `Repository URL` 칸에 입력합니다.

### Credentials 설정하기
포크한 레포지토리를 관리하기 위한 권한을 생성합니다.  
`Add`버튼을 누르고 `Jenkins` 탭을 선택해서 권한 생성 페이지를 오픈합니다.

{{< alert >}}
**알림** 계정은 생성 즉시 암호화 되기 때문에 관리자가 알 수 없습니다.
{{< /alert >}}

#### Github 토큰 발급하기
1. 본인의 깃허브 페이지로 이동해서 `Settings`를 클릭해서 계정 설정페이지로 이동합니다.
2. 좌측의 사이드바에서 제일 아래의 `Developer Settings`을 클릭해서 개발자 설정으로 이동합니다.
3. `Personal access tokens`에서 `tokens (classic)`를 선택해서 토큰을 생성합니다.
4. `Generate new token`을 클릭하고 `Generate new token (Classic)`을 선택합니다.
5. 중요한 설정으로 발급하기 전에 비밀번호를 요구하기 때문에 비밀번호를 입력해서 인증합니다.
8. `Note`는 토큰의 이름값 `Expiration`은 만료일 설정입니다. 
9. `Select scopes`에서는 젠킨스 빌드만들 위한 설정으로 `repo`와 `admin:repo_hook`만 선택해서 생성합니다.
10. 만들어진 토큰은 생성된 이후에만 조회할 수 있기 때문에 기록해두어야 합니다 다만 절대 외부로 유출해서는 안됩니다.

![New personal access token](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-09.png)

#### Credentials 추가
위에서 [발급한 토큰](#github-토큰-발급하기)을 사용해서 계정을 생성합니다. 

![Add Credentials]assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-10.png)

`Domain`과 `Kind`, `Scope` 를 이미지와 동일하게 선택하고 나머지 칸의 경우 아래 정보를 참고해서 작성해주세요!

| 컨텍스트 | 설명 |
| -------| ----|
| `Username` | 젠킨스에서 다양한 출력에 표시될 유저이름을 입력합니다.|
| `Password` | 깃허브에서 발급받은 토큰을 입력합니다. |
| `ID` | 사용중인 깃허브의 아이디를 입력합니다. |
| `Description` | 젠킨스에서 표시될 설명을 입력합니다. |

설정을 완료하고 선택을 하면 다음과 같이 표시됩니다.

![Credentials](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-11.png)

`Branches to build` 를 이제 설정합니다.  
실제로 빌드를 진행하는 브랜치를 작성해주면 됩니다.

별도로 브랜치를 설정해서 개발하지 않았기 때문에 `main` 브랜치를 작성합니다.
만약에 별도로 브랜치를 설정해서 개발한다면 실제로 사용할 브랜치를 작성해주세요!
```md
*/main
```

`Add Branch`를 눌러서 빌드가 진핼 될 브랜치를 추가할 수 있습니다.

`Repository browser`의 경우 사용할 레포지토리 사이트를 선택할 수 있는데 자동으로 선택해주세요 또는 Github를 선택해도 동일합니다.

<br/><br/><br/>

## 빌드 유발 설정하기
빌드 유발의 경우 특정 이벤트를 캐치하면 젠킨스에서 설정한 [소스 코드 관리](#소스코드-관리-설정하기) 레포지토리를 빌드합니다.

![빌드 유발](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-12.png)

### 옵션 선택하기
`GitHub hook trigger for GITScm polling`를 선택합니다.  
위 기능의 의미는 `git`에서 `push` 이벤트를 감지하게 되면 `hook`을 보내게 되어 젠킨스에서 자동으로 빌드를 진행하라는 옵션입니다.

<br/><br/><br/>

## Build Steps 설정하기
빌드시 진행될 각종 설정들을 개발자가 직접 지정할 수 있습니다.  
사실 위에 설정들은 기본적인 설정들이고 실질적인 설정 부분은 바로 여기에서 설정한다고 봐도 무방합니다.

### Invoke Gradle script 설정하기
탬플릿의 경우 자바의 스프링 부트를 기반으로 개발했기 때문에 전용 빌드 툴을 활용합니다.  
이미 설정한 옵션 그대로 설정해주세요!

- `Gradle Version`에 `Gradle-8.1` 선택 
- `Tasks`에 `bootjar` 선택

`Tasks`의 경우 자바 빌드와 동일하게 다양한 빌드 옵션을 선택할 수 있습니다. 

![Invoke Gradle script](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-13.png)

### Execute shell 설정하기
쉘 스크립트 작성을 통해서 `jar` 파일을 서버에 실행해주는 작업을 시작해보겠습니다.

![Execute shell](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-14.png)

#### 배포 스크립트 전체
스크립트에 대한 설명을 하기전에 전체 스크립트를 보여드리겠습니다.  
일단 전체적인 흐릅에 대해서 파악해 보세요

```shell
# 자동 그래들 빌드 배포
pid=$(ps -ef | grep {jar-file-name}\* | grep -v "grep" | grep -v $0 | awk '{print $2}') # 기존 서버 PID 확인
if [ -z $pid ] # 앱이 작동중인지 체크
then
 echo 앱이 작동중이 아닙니다. # 앱이 작동중 이 아니면 다음으로
else
 kill -9 $pid # 앱이 작동중이면 PID로 강제 종료
 echo 앱을 성공적으로 종료 했습니다, 프로세스 아이디: $pid.
fi
cd build/libs # 빌드 파일 위치로 이동
sudo rm -rf /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name}/{jar-file-name}*.jar # 기존 빌드 파일 삭제
sudo mv {jar-file-name}*.jar /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name} # 신규 빌드 파일을 운영 디렉터리로 이동
cd /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name} # 운영 디렉터리로 이동
BUILD_ID=dontKillMe nohup java -jar {jar-file-name}*.jar 2>> /dev/null >> /dev/null & # 빌드 파일 배포
echo ============ 새로운 빌드 성공적으로 배포중!!! [PID:$!] ============
```

#### 배포 스크립트 설명
이제 스크립트에 대해서 설명을 진행하겠습니다.

- {jar-file-name}: 설정한 자바파일 이름을 입력해주세요.
- {dir-name}: 지정받은 폴더 이름을 입력해주세요.

##### Jar 파일이 서버에 실행중인지 확인 후 종료
서버에서 실행중이던 기존 Jar 프로세스를 찾아서 종료합니다.
```shell
pid=$(ps -ef | grep {jar-file-name}\* | grep -v "grep" | grep -v $0 | awk '{print $2}') # 기존 서버 PID 확인
if [ -z $pid ] # 앱이 작동중인지 체크
then
 echo 앱이 작동중이 아닙니다. # 앱이 작동중 이 아니면 다음으로
else
 kill -9 $pid # 앱이 작동중이면 PID로 강제 종료
 echo 앱을 성공적으로 종료 했습니다, 프로세스 아이디: $pid.
fi
```

##### 젠킨스 빌드 경로로 이동
기본 젠킨스 빌드 경로로 빌드 파일을 수정하기 위해 이동합니다.
```shell
cd build/libs # 빌드 파일 위치로 이동
```

##### 기존 빌드 파일 삭제
기존 빌드파일을 운영 폴더에서 삭제합니다.
```shell
sudo rm -rf /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name}/{jar-file-name}*.jar # 기존 빌드 파일 삭제
```

##### 새로 빌드된 파일 이동
새롭게 빌드된 파일을 운영 폴더에 이동합니다.
```shell
sudo mv {jar-file-name}*.jar /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name} # 신규 빌드 파일을 운영 디렉터리로 이동
```

##### 운영 경로로 이동
운영 폴더로 이동합니다.
```shell
cd /home/{dir-name}/Server/Webserver/work.laterre.dev/public_html/{dir-name} # 운영 디렉터리로 이동
```

##### 최신 빌드 파일 실행
새로 빌드된 운영파일을 실행합니다. 
```shell
BUILD_ID=dontKillMe nohup java -jar {jar-file-name}*.jar 2>> /dev/null >> /dev/null & # 빌드 파일 배포
```

##### 빌드 실행 성공 출력문
빌드에 성공하면 문자열을 출력합니다.
```shell
echo ============ 새로운 빌드 성공적으로 배포중!!! [PID:$!] ============
```

명시해둔 "{id}" 를 제외한 나머지 스크립트 코드는 건들지 말아주세요 오류가 날 가능성이 큽니다.

<br/><br/><br/>

## 빌드하기
여기까지 읽으면서 실습하셨다면 모든 절차가 끝났습니다. 
저장하고 프로젝트를 빌드해서 사이트를 배포해보세요  

배포에 성공하셨다면 `Build History`에 초록색 체크 표시가 나타나게 됩니다.

![Build History](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-15.png)

빌드에 성공하셨다면 사이트에 접속해서 직접 API 테스트를 진행해주세요!  
사이트 세부 경로도 아이디명으로 생성되어 있습니다!

### 배포 후 사이트 접속

#### 접속시 첫 화면
![Build site #1](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-16.png)

#### 회원가입 페이지
![Build site #2](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-17.png)

#### 회원가입 완료 페이지
![Build site #3](assets/images/post/2025-04-25-젠킨스-스프링-배포-해보기/jenkins-img-18.png)