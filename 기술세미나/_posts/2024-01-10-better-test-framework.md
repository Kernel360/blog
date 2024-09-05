---
layout: post
title: 더 나은 테스트로 인도해줄 친구들
author: 조형준
categories: 기술세미나
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0110/1.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [Mockito, Jacoco, 테스트코드, 기술세미나]
---
## 개요
안녕하세요, 커널 360 2차 기술세미나의 포문(for문아님)을 열게된 조형준입니다.  
저번 포스팅에선 클린코드에 관한 내용을 다뤘는데요, 이번 포스트는 더 좋은 테스트코드를 도와주는 프레임워크를 소개해드리겠습니다!

## 테스트코드, 이제 좀 짰어요!
![2.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/2.png?raw=true)  
ControllerTest..  
ServiceTest..  
RepositoryTest.. 등등..  
여러 테스트코드를 작성했겠다. 이제 좀 테스트코드를 알 것 같아요!  

과연 그럴까?  

![3.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/3.png?raw=true)
![4.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/4.png?raw=true)
![5.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/5.png?raw=true)  
  
네.. 테스트 커버리지는 박살이 났습니다. 이대로 출시하게 된다면 뭐.. 결과는 뻔하겠죠  
  
![6.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/6.png?raw=true)

## 테스트도 일이다. 효율적으로 하자.
이러나저러나, 테스트코드도 어쨌든 공수입니다. 따라서 우리는 빠르게 테스트코드를 작성하고, 테스트 커버리지도 효율적으로 올릴 방법을 알아보도록 합시다.  

### Mockito  
우린 지금까지 실제 객체를 매핑하고, DB 커넥션까지 실제 DB에 연결해서 사용했었습니다.  
하지만, 이러한 테스트 방식을 모든 테스트코드에 적용한다면 다음과 같은 이슈가 있습니다.

- 실제 객체를 테스트에 사용하면 너무 복잡해지고 의존성이 높아져..
- 테스트를 할 때마다 Spring application을 올리기엔 너무 오래걸림..
- 기존 가짜(Mock)객체 프레임워크는 어렵고.. 문법도 복잡해!
 
![7.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/7.png?raw=true)
이를 해결하기 위해 현재 자바진영에선 가장 대표적인 Mockito 프레임워크를 사용하고 있습니다.  
실제 객체 대신 Mock객체를 설정해, 테스트를 빠르고 간결하게 도와줍니다!
Mock이 무엇인가? 에 대해선 제 [개인 블로그](https://kkkapuq.tistory.com/146)에 작성해두었으니 참고하시면 좋을듯합니다.

한마디로 요약하자면, Mock은 프록시, 즉 가짜 객체를 만들기 때문에 리소스 효율이 좋고, 성능이 빠르기에 행위 검증의 단위테스트에 매우 적합합니다.
간단하게 사용 방법을 알아보시죠. 간단한 PostDto 객체를 생성해줍니다.

![8.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/8.png?raw=true)

이후 Mockito의 when 문법으로, `findById()`를 호출하면 `existingPostDto.entity()`가 와야해! 라고 선언해줍니다.

![9.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/9.png?raw=true)

이처럼 어떤 행위의 인과관계에 대해 명시적으로 선언해주고, 실제로 실행시켰을 때 프로그램이 해당 로직을 정상적으로 수행하면, 테스트에 성공하게 됩니다.  
Mockito의 활용법에 대해선 많은 자료가 있으니, 본 포스팅에선 따로 다루지 않도록 하겠습니다.

## 테스트 커버리지는 어캄?
![10.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/10.png?raw=true)
이 IntelliJ는 무료로 해줍니다.  
IntelliJ 한정으로 무료로 제공해줍니다만, 매우 부실합니다. 간편하지만, 그만큼 커버리지 검증이 약합니다.

> 테스트할 패키지 우클릭  
> More Run/Debug  
> Run ‘~~’ with Coverage

### Jacoco
![11.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/11.png?raw=true)
`Java Code Coverage`의 약자로, 자바의 대표적인 커버리지 체크 라이브러리입니다.
커버리지 기준을 설정할 수 있고, (테스트 커버리지 80% 미만이면 빌드가 안되게 한다던가)
html, xml, csv 등 파일로 출력이 가능해서, 현업에서도 많이 쓰고 있는 오픈소스입니다!  
세팅 방법은 [여기](https://techblog.woowahan.com/2661/) 에서 확인 후 적용해보시면 좋을 듯 합니다.

테스트코드를 실행하고, build/jacoco/index.html의 파일을 실행시켜보면, 이렇게 퍼센티지로 확인이 가능합니다.  
![12.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/12.png?raw=true)  
  
이런식으로 코드 내 메서드 단위로도 확인이 가능하기 때문에, 좀 더 디테일한 테스트코드 작성이 가능해집니다.  

![13.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/13.png?raw=true)
![14.png](https://github.com/Kernel360/blog-image/blob/main/2024/0110/14.png?raw=true)  

- 초록색 : 테스트가 진행 된 부분
- 노란색 : 조건, 결정 커버리지가 모두 충족되지 않은 부분
- 빨간색 : 커버리지가 진행되지 않은 부분

## 마치며
이번 포스팅에선 테스트를 조금 더 윤택하고, 퀄리티있게 작성해주는 Mockito / Jacoco에 대해 알아봤습니다.  
Jacoco의 유일한 단점... 그것은 커버리지 100%를 채우지 못하면 불-편해짐...
