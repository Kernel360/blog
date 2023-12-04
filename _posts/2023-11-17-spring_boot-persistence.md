---
layout: post
title: 스프링과 데이터 영속성(Persistence)
author: 김원상
categories: 기술세미나
banner:
  image: /assets/images/database_picture.jpg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [java, 데이터영속성, JDBC, SQLMapper, ORM, 기술세미나]
---

안녕하세요. 저는 이번 Kernel360의 기술세미나에서 자바진영에서의 데이터 영속성 라는 주제로 발표하게된 김원상입니다.

데이터 영속성은 Layered-Architecture의 가장 밑단의 Data Layer와 Domain Model사이를 잇는 Persistence계층에 대한 내용이라 보시면 되겠습니다.

![레이어드 아키텍처](https://github.com/Kernel360/blog-image/blob/main/2023/1117/03_blog_image.png?raw=true)

## 1. 데이터 영속성이란?

데이터 영속성은 어플리케이션이 닫혀도 그로 인해 생성된 데이터가 비휘발성(non-volatile) 저장소에 기록되어 사라지지 않는 것을 의미합니다.

간단하게 말씀드리자면, 세션의 열고 닫음 사이에도 어플리케이션에서 생성된 데이터가 DB와 같은 영속적인 저장소에 입력되어 다음 세션에서도 이를 확인하고 활용할 수 있다는 뜻이 되겠습니다.

이를 위한 기술로 Java 진영에는 API와 Persistence Framework를 지원한다고 볼 수 있는데, 가장 많이 활용되는 각각의 범주로 JDBC API와 SQL Mapper(예, MyBatis), Object Relational Mapper(예, Hiberate)가 있다고 볼 수 있겠습니다.

![영속성 개념도](https://github.com/Kernel360/blog-image/blob/main/2023/1117/02_blog_image.png?raw=true)

## 2. 자바진영의 영속성 프레임워크 타임라인

이처럼 원활한 서비스의 구성을 가능케한다는 점에서 Persistence 계층과 이를 위한 기술은 매우 중요하다고 할 수 있겠는데요. 위에 예시를 든 세 기술의 발전 양상에 대한 타임라인을 조사해보았습니다.

우선, JDBC API는 Sun Microsystems에 의해 1997년 처음 릴리즈가 된 이후 2017년까지 안정적으로 유지되고 있습니다.

MyBatis는 Apache재단의 iBATIS가 전신이며 2010년에 Apache재단은 프로젝트를 MyBatis로 이관했습니다. 현재까지 잘 유지가 되고 있는 기술입니다.

Hibernate는 iBATIS보다 조금 더 일찍 2001년 Gavin King에 의해 시작되었으며, 현재까지 가장 많이 사용되는 표준 자바 ORM이라 할 수 있겠습니다.

![타임라인 그림](https://github.com/Kernel360/blog-image/blob/main/2023/1117/04_blog_image.png?raw=true)

## 3. JDBC API와 유념하면 좋은 사항들

JDBC(Java Database Connectivity)는 자바 프로그램과 데이터베이스를 연결시켜주는 가장 밑단에서 작용하는 자바 API라고 볼 수 있습니다. 자바 어플리케이션을 제작하는 과정에서 여러 종류의 데이터베이스를 사용할 수 있는데, 각 데이터베이스마다 다른 드라이버를 매개로 하여 거의 동일한 코딩 과정으로 영속성을 부여할 수 있다는 것이 특징이 되겠습니다.

다만, 기본 SQL문법으로 동작하는 여러 데이터 입출력은 동일하지만 **데이터베이스마다 상세한 쿼리 문법은 다르니** 이를 유념하셔야겠습니다.

관계형 데이터베이스 뿐만 아니라 MongoDB같은 NoSQL도 지원하여 폭넓은 사용성을 제공합니다.

![JDBC 그림](https://github.com/Kernel360/blog-image/blob/main/2023/1117/05_blog_image.png?raw=true)

## 4. SQL Mapper(MyBatis)는 어떻게 사용할까?

MyBatis를 사용하여 Java의 비즈니스 로직 구현 부분과 SQL쿼리를 분리할 수 있습니다. 보통은 XML형식의 파일에 각각의 메서드와 실행시키고자 하는 SQL문을 매핑시켜놓습니다. 또한 Java코드내에서도 이를 수행할 수 있는데 아래 그림의 좌측 하단 부분의 코드가 이에 해당한다고 볼 수 있습니다. 이러한 경우 파일 형식을 통하여 물리적으로 Java코드와 SQL을 분리하는 SQL Mapper의 이점은 다소 상실하는 것 같습니다.

또한, SQL Mapper는 Java 개발자들에게 많은 편이성을 제공하였으나 코딩 과정이나 컴파일 도중에 SQL문의 오류를 캐치하기 어렵다는 한계가 있습니다.

![MyBatis 그림](https://github.com/Kernel360/blog-image/blob/main/2023/1117/06_blog_image.png?raw=true)

## 5. ORM(Spring Data JPA)과 영속성 부여 Layer

이러한 단점을 보완하는 기술이라 볼 수 있는 것이 ORM이라 볼 수 있습니다.

SQL Mapper는 데이터 접근 부분을 SQL과 매핑시켰다면, ORM은 객체와 테이블을 잇는, 조금 더 양 끝단의 편의성을 개발자에게 제공해준다고 할 수 있습니다.

다만, 여기서도 알아두면 좋은 점은 ORM을 사용하는 것이 JDBC API와 완전히 무관하지 않다는 점입니다. 아래 그림을 보시면 Spring Data JPA를 사용하여 영속성 계층을 구현한다고 하더라도 가장 밑단의 관계형 데이터베이스와 통신하는 작업은 JDBC API에서 수행하게 됩니다. 이를 염두에 두어 이후 관련 기술에 대한 깊은 지식을 탐구할 때 참고할 필요가 있겠습니다.

![Spring Data JPA 그림](https://github.com/Kernel360/blog-image/blob/main/2023/1117/07_blog_image.png?raw=true)

## 6. 끝으로

JDBC API, SQL Mapper, ORM은 서로가 마치 기술의 발전 계보를 잇는 것처럼 보입니다. SQL Mapper와 ORM은 서로 별개의 기술로 존재하지만 JDBC API를 밑단에 두고 각각 장단점이 존재한다고 할 수 있겠습니다.

ORM은 상당히 고도의 추상화를 제공하여 개발자들의 개발 속도를 높여주었다고 할 수 있습니다. 그렇다면 ORM만 사용하는 것은 항상 옳을까요?

저의 의견은 꼭 그렇지는 않다는 것입니다. 아래 그림을 보시면 ORM이 트렌디한 기술임에는 분명하지만 MyBatis에 대한 관심이 상당수의 동아시아 국가에서 보여지고 있는 것도 사실입니다. 따라서 여러 기술에 대하여 열린 자세와 익숙한 기술에 대한 비판적인 태도는 개발자에게 중요하다고 여겨집니다.

![끝으로 그림 01](https://github.com/Kernel360/blog-image/blob/main/2023/1117/08_blog_image.png?raw=true)

또한, 저는 이번 발표를 준비하면서 다음과 같은 궁금증이 생겼습니다.

1. ORM에서 객체와 테이블을 매핑시켰다는 것이 구체적으로 어떤 의미인가?

2. Spring Data JPA가 만능이 아니라면 어떤 대안이 있겠는가?

3. NoSQL을 사용할 경우 어떤 식으로 영속성 계층을 구현할 것인가?

등등 ...

![끝으로 그림 02](https://github.com/Kernel360/blog-image/blob/main/2023/1117/09_blog_image.png?raw=true)

공부하다보면 물음이 끝없이 이어집니다.

이로써 데이터 영속성에 대한 대략의 설명을 드립니다.

조금 더 심화되고 실용적인 지식으로 다시 찾아뵙겠습니다.

감사합니다.

---

**참고자료**

[MongoDB Document](https://www.mongodb.com/databases/data-persistence)

[10분 테코톡: 아마찌의 ORM vs SQL Mapper vs JDBC](https://www.youtube.com/watch?v=VTqqZSuSdOk)

[MyBatis 공식 문서](https://mybatis.org/mybatis-3/ko/getting-started.html)

[[신버전] 한 번에 끝내는 Spring 완.전.판 초격차 패키지 Online](https://fastcampus.co.kr/dev_online_spring)

[[한 번에 끝내는 데이터 엔지니어링 초격차 패키지 Online]
Hibernate는 어떻게 나의 커리어를 망쳤는가?](https://www.toptal.com/java/how-hibernate-ruined-my-career)

[관계형 데이터베이스가 Hibernate와 Spring Data를 피할 때 생기는 이점](https://itnext.io/advantages-of-not-using-spring-data-and-hibernate-with-relational-data-8a509faf0c48)

---
