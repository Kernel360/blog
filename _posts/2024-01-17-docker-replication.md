---
layout: post  
title: 배포있게 배포하기 -DB편-
author: 손민우
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0117/1.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [Docker, AWS, DB]
---

다시 돌아온 Docker 세미나 시간입니다. 어쩌다보니 오픈세미나에 이어서 이번 주제도 Docker와 관련한 주제로 선택을 하게 되었는데요. (정말 파도 파도 공부해야 하는 부분이 계속 나오더라구요.)

이전 오픈 세미나에서는 Docker의 기본적인 내용을 설명을 했다면 오늘 설명드릴 내용은 DB replication을 Docker를 통해서 적용해 보고 이를 쉽게 적용할 수 있도록 나온 서비스인 AWS RDS를

통해서 이해하는 시간을 가져보려고 합니다. 그럼 시작해보겠습니다.

## 1. Docker-Compose는 무엇인가?

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/2.png">
</div>

아마 Docker를 공부하다 보면 Docker-Compose에 대해서 이름은 들어보셨을 거라 생각됩니다. 제가 처음 Docker-Compose를 접하게 되었을 때 느꼈던 점은 사용만 할 줄 안다면 좋은 툴이 될 것 
같은데 그 과정이 쉽지 않을 것 같다는 생각이 정말 많이 들었습니다. 

그래서 이러한 생각을 좀 줄여 보고자 MySQL을 예로 들어 Docker-Compose를 작성해 보겠습니다.

## 2. Docker-Compose로 MySQL 실행하기

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/3.png">
</div>

다음 사진은 MySQL을 실행시키기 위한 docker-compose.yml 파일의 내용입니다. 다음은 설명입니다.
|옵션|설명|
|------|---|
|image|docker를 실행할 image:tag를 설정합니다. ex) ubuntu, linux, mysql, postgresql ...|
|ports|port를 설정하는 옵션으로 [host port]:[container 내부 port]를 의미합니다.|
|environment|container를 실행할 때 환경 변수를 추가한 상태로 실행할 수 있습니다. 각 이미지마다 환경 변수가 다를 수 있어 확인한 후 사용해야 합니다. ex) MYSQL_ROOT_PASSWORD: 1234 |
|command|컨테이너를 실행 한 후 내부에서 명령어를 실행해 적용할 수 있습니다.|
|volumes|마운트를 지정하게 되면 이후 컨테이너가 종료되거나 삭제되더라도 이후 연결되는 컨테이너가 같은 경로에 마운트 된다면 이전 데이터를 받을 수 있습니다.|

어떤가요? 이렇게 보면 docker-compose라는 것은 단순하게 script의 형태를 띄기 때문에 각각의 의미만 안다면 어렵지 않습니다. 이제 간단하게 해봤으니 DB replication도 진행해보도록 하겠습니다.

## 3. MySQL DB replication 구성해보기

DB replication을 진행하기 전에 간단하게 replication에 대해 알아보겠습니다.

DB replication은 데이터 저장과 백업하는 방법에 관련이 있는 데이터를 호스트 컴퓨터에서 다른 컴퓨터(컨테이너)로 복사하는 것을 의미합니다. 그렇다면 왜 필요할까요?

서비스를 운영함에 있어서 읽기에 대한 요청은 읽기 이외의 요청 (생성, 수정, 삭제)에 비해 많은 비중을 가지게 됩니다. 그렇기 때문에 읽기에 대한 요청을 전담하여 맡아줄 DB를 생성해 DB로 하여금 부하를
감소시킬 수 있도록 역할을 할 수 있습니다.

<div align="center">
<img alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/4.png">
</div>

다음 사진은 이제 진행할 db replication의 파일 구조입니다.
위 파일 구조는 master-slave가 1 대 1로 되어 있지만 설정에 따라 N 대 N이 될 수도 있습니다.

![image.jpg1](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/5.png) ![image.jpg2](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/6.png)

위의 스크립트는 순서대로 master-db와 slave-db의 Docker-Compose의 내용이다. 위에서 설명하지 않는 옵션은 다음과 같습니다.

|옵션|설명|
|------|---|
|build| context는 build를 실행한 경로를 dockerfile은 dockerfile이 위치한 경로를 입력해주면 됩니다.|
|restart|재시작과 관련한 옵션으로 현재 항상 재시작한다는 옵션으로 지정되어 있습니다.|
|networks|docker에는 network를 통해 컨테이너 간의 연결을 진행하고 데이터를 주고받을 수 있으며 기본값은 bridge network입니다.|

docker-compose.yml파일 외에도 my.cnf와 Dockerfile이 있는데 이때 Master와 Slave의 차이는 파일 경로와 읽기 전용 옵션 외에는 동일하여 Master를 기준으로 보겠습니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/7.png">
</div>

Dockerfile에서 내용은 다음과 같습니다.

|옵션|설명|
|------|---|
|FROM|컨테이너의 환경을 지정하는 옵션으로 mysql이미지를 실행한다는 의미입니다.|
|ADD|host에 위치한 파일을 컨테이너 내부 위치에 추가한다는 의미입니다. mysql의 configure 파일을 교체하여 내가 정한 규칙대로 실행되도록 합니다.|

my.cnf 파일의 내용입니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/8.png">
</div>

현재 있는 파일에서 초록색 표시가 되어있는 read_only 옵션은 Slave-db가 가지고 있는 옵션으로 읽기 전용으로 사용하기 위한 옵션입니다.

이렇게 준비가 되었다면 docker-compose up -d 명령어를 통해 실행을 시키면 다음과 같은 log와 설정을 볼 수 있습니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/9.png">
</div>

성공적으로 build가 되었다면 이제 db로 접속하여 설정을 진행해주어야 합니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/10.png">
</div>

replication에 대한 유저를 생성하고 권한을 부여하는 과정입니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/11.png">
</div>

권한 부여 이후에는 Slave DB에서 Master DB로 바라볼 수 있도록 설정을 수정합니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/12.png">
</div>

설정을 진행한 이후 아래 명령어를 통해 동일한 내용이 나온다면 성공적으로 설정된 것을 볼 수 있습니다. 

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/13.png">
</div>

Docker를 사용하여 replication을 진행하는 과정은 이렇게 마무리할 수 있습니다. 긴 과정일 수 있지만 실제로는 dockerfile과 docker-compose.yml, my.cnf만 여러 개 둔다면
규모가 큰 DB replication도 가능하기 때문에 오히려 구성하기 어렵지 않았다고 생각됩니다. 

하지만 개발자 입장에서 개발에 더 몰입할 수 있도록 간단하게 구성할 수 있는 서비스가 AWS RDS입니다.  

## 4. AWS RDS란?

AWS RDS(Amazon Relational Database Service)는 aws에서 제공되는 서비스로 MySQL 뿐만 아니라, PostgreSQL, Aurora, Oracle 등 여러 DB를 간단하게 구현할 수 있습니다.
단일 구성 뿐 아니라 위에서 진행했던 DB replication도 보다 쉽게 구성할 수 있습니다. 그럼 AWS RDS로도 DB replication을 구성해 보도록 하겠습니다.

## 5. AWS RDS replication 구성해보기

처음 AWS에 접속하여 RDS로 이동하면 다음과 같은 화면을 볼 수 있고 데이터베이스 생성 버튼을 누릅니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/14.png">
</div>

그 다음 여러 엔진 유형 중 제가 생성하고자 하는 MySQL을 선택하고 Database명, 비밀번호 등을 설정합니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/15.png">
</div>

이때 외부에서 접속하고자 한다면 퍼블릭 엑세스를 예로 채크해야 합니다. 이렇게 설정을 마치고 생성을 진행하면 5 ~ 10분 정도를 기다리면 생성이 완료 됩니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/16.png">
</div>

생성이 완료되면 생성된 DB를 선택하고 작업에서 읽기 전용 복제본 생성을 진행하고 위의 생성과정을 반복합니다.

<div align="center">
<img width="692" alt="image" src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0117/17.png">
</div>

이렇게 하면 자동으로 RDS 내에서 master와 slave가 적용되며 db를 동일하게 백업하고 저장되는 것을 확인할 수 있습니다.
Docker를 사용하는 것도 간단해 보였지만 AWS RDS를 사용하니 더욱 간단하게 구현을 마무리할 수 있었습니다.

## 6. 마무리

결론은 AWS RDS로 구현하는 것이 간단하다가 아닌 Docker를 통해서 구성하는 것을 토대로 AWS RDS를 이해하는 시간이 되었으면 좋겠습니다. (물론 개념적인 부분에 대한 설명이 부족하지만 ...)
이러한 과정을 반복하는 것 만으로도 전체적인 흐름을 이해하는데 도움이 될 것이라고 생각합니다. 
끝까지 읽어주셔서 감사합니다.
