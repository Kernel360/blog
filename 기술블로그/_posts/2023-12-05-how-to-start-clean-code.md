---
layout: post
title: 클린코드, 뭐부터 시작해볼까?
author: 조형준
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2023/1205/1.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [클린코드, 기술세미나]
---
## 개요
안녕하세요, 커널360의 크루로 활동중인 조형준입니다.  
이번 기술세미나의 주제는 '클린 코드'를 선정해봤습니다.  
클린코드는 '읽기 쉬운 코드', '협업하기 좋은 코드' 등과 같은 특징을 가지고 있는데요.  
본 포스팅에서 어떤것부터 시작하면 좋을 지 찬찬히 따져보도록 하겠습니다.

## 클린코드?
![2.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/2.png?raw=true)  
개발자 필독도서라고 한번쯤은 들어보셨을 그 책입니다.
책의 이름이기도 하지만, 결과적으로는 좋은코드, 읽기 좋은 코드 등의 의미와 일맥상통한다고 생각합니다.  
클린코드에서 말하는 규칙은 여러개가 있는데, 몇가지만 톺아보자면 다음과 같습니다.  
- 함수는 하나의 기능만
- 나쁜 주석 배제
- 좋은 함수 이름
- 철저한 네이밍 컨벤션
- 고치기 쉬운 코드
- 그 외 등등..

## 적용 사례
![3.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/3.png?raw=true)  
간단하게 예제를 하나 들어보겠습니다.  
위 코드는 제가 E2E 프로젝트에서 진행했던 코드인데요, 게시글의 좋아요 여부값을 가져오는 코드입니다.  
로직상으로는 크게 문제는 없지만, 어떤 문제가 있을까요? 다음과 같습니다.
- 좋아요 여부값을 가져오는 함수에서 사용자의 이메일을 가져오고있음.
- 함수의 역할에 맞지 않게 유저 정보를 가져오는 작업을 추가로 진행중
- 자연스럽게 코드의 길이는 늘어나고, 가독성이 안좋아짐.

위와 같은 문제를 해결하기 위해서, 다음과 같이 코드를 수정해봤습니다.
![4.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/4.png?raw=true)
게시글 좋아요의 여부를 가져오는 메서드와, 로그인 유저의 id를 가져오는 메서드를 분리했습니다.  
이처럼, 클린코드는 거창한게 아닙니다. 책임을 최대한 분리해주고, 가독성 향상을 위한 노력이 클린코드를 만들어줍니다.  
하지만 이렇게 수정한 코드도 누군가에겐 dirty code가 될테죠... (주륵)
![5.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/5.png?raw=true)  

## 그렇다면 우리는 뭘 해야할까?
To-do 도 중요하지만, 더 중요한 것은 Not-To-Do 라고 생각합니다.  
하지말라는 것만 안해도, 코드가 저절로 보기 좋아집니다.  
따라서, 본 포스팅에서 소개해드리는 안티패턴을 먼저 지양하고, 주니어 레벨에서 할 수 있는 것부터 천천히 실행해봅시다.

### Arrowhead
![6.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/6.png?raw=true)
화살촉 패턴이라는 이름의 이 안티패턴은, 사진이 모든걸 설명해줍니다.  
과도한 if문으로 if문의 depth가 늘어나는 것을 지양해야 합니다. 아래와 같이 수정할 수 있겠습니다.  
![7.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/7.png?raw=true)  

### Magic numbers
![8.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/8.png?raw=true)
이 코드만 보고 무슨 로직인지 이해가 가시나요?  
이 코드는 유저의 id가 100, 즉 사장이라면 견적서를 수정할 수 있게 하라는 의미입니다.  
특정 상수가 특별한 의미를 갖지 말게 해야합니다. 다음처럼 수정할 수 있겠습니다.  
![9.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/9.png?raw=true)

### Cryptic names  
![10.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/10.png?raw=true)
이 친구도 사진이 모든걸 설명해줍니다.
`i`, `p`가 도대체 뭘까... 사람이 이해할 수 있는 변수나 메서드명을 사용하자는 것입니다. 즉, 암호를 쓰지맙시다.  
다음과 같이 수정해봅시다.
![11.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/11.png?raw=true)

### Single funtion, Multiple behaviors
![12.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/12.png?raw=true)
하나의 함수가 여러 역할, 즉 책임을 맡지 말자는 의미입니다.  
위 코드는 calculate라는 함수가 사칙연산을 수행하고 있는데요, 다음과 같이 역할을 분리해줍시다.
![13.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/13.png?raw=true)

### God class
![14.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/14.png?raw=true)
하나의 클래스가 너무 많은 역할을 하지 않도록 하는 것입니다.  
방금 전 언급한 안티패턴과 일맥상통합니다.  

### Silencing Error/Exceptions
![15.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/15.png?raw=true)
이 패턴은 실제 개발을 하면서 많이 겪게되는 문제일 것 같은데요.  
조용한 에러와 예외를 만들지 말자는 것입니다. 말로는 조금 감이 안잡히실텐데, 위 코드의 문제점이 무엇일까요?  
해당 코드는 유저가 작성한 파일을 모두 삭제하고, 유저를 비활성화 시키는 로직입니다.  
그러나 모종의 이유로 try문에서 실행이 되지 않더라도, 유저를 비활성화 시키는 로직이 실행되게 됩니다.  
한 두명인 경우 큰 문제는 안되겠지만, 대규모 시스템이라면 당연히 문제가 되겠죠? 이런 코드는 아래와 같이 바꿔볼 수 있겠습니다.  
try-catch문을 지워서 에러가 터지면 바로 종료되게 하거나, 파일 삭제를 삭제될때까지 계속 돌리는겁니다.
![16.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/16.png?raw=true)

### Bad Comment
![17.png](https://github.com/Kernel360/blog-image/blob/main/2023/1205/17.png?raw=true)
> Good code is self-documenting  
> 좋은 코드는 그 자체로 이미 훌륭한 주석이다  

불필요한 주석이나, 너무 긴 주석은 자제하는 것이 좋습니다.  
다만, 도메인 지식이나 구성원들이 알면 좋은 정보들은 주석으로 남겨주면 좋습니다.

## 마치며
이렇게 클린코드를 뭐부터 시작해야 할지, 안티패턴들을 학습해보는 시간을 가졌습니다.  
간단하게 정리를 해보겠습니다.  
- 클린코드 법칙을 실천하기 앞서, 안티 패턴 코드를 지양하자.
- 클린코드를 만드는 방법보다 좀 더 쉬울 것.
- 안티 패턴 코드를 지양하면서, 클린코드 법칙을 천천히 적용하자.
- 뭐가 됐건, 스스로 가독성이 좋은 코드라고 느껴질 때까지 리팩토링해보자.

여러분들도 코드를 작성하시다가, 해당 포스트의 내용을 떠올리면서 코드를 작성한다면, 클린코드에 한층 더 가까워질 수 있을 것이라 생각합니다!  

**참고영상**  
[코드에 이런 짓만 안 해도 욕먹지 않는 개발자가 될 수 있어요! 미국 개발자들을 열받게 하는 코드 패턴 공개.](https://youtu.be/ixOk13jC50w?si=fAOotcSJAd8jalAd)

