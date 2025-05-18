---
layout: post
title: Fixture Monkey
author: 손현준
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2024/0130/0.png?raw=true?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [FixtureMonkey, 테스트코드, 네이버]
---

안녕하세요. 저는 이번 Kernel360의 기술세미나에서 Fixture Monkey를 주제로 발표하게된 손현준입니다.

Fixture Monkey는 2023년 10월 네이버 오픈소스로 1.0.0 버전으로 release 되었습니다.

Fixture Monkey 네이버페이 Platform lab에서 만든 PBT(Property Based Testing) 도구로 실제 네이버 사내에서 활용중인 도구입니다.

<br>

## 1. Fixture Monkey 소개
![참고자료1](https://github.com/Kernel360/blog-image/blob/main/2024/0130/1.png?raw=true?raw=true)

Fixture Monkey는 제어 가능한 임의 테스트 개체를 쉽게 만들도록 설계된 Java 및 Kotlin 라이브러리입니다.

필요한 테스트 픽스처 생성을 쉽고 빠르게 도와줘서 테스트 작성을 단순화하는 데 중점을 둡니다. 

Fixture Monkey는 필요한 테스트 개체를 손쉽게 생성하고 원하는 구성에 맞게 쉽게 사용자 정의할 수 있도록 도와줍니다.



## 2. 기존 테스트 코드의 문제점?


### 1. 우리가 작성하는 테스트에서도 엣지 케이스가 존재한다.

우리가 작성하는 테스트 에서도 엣지 케이스가 존재합니다. 예를 들어, 코딩테스트를 볼 때, 테크스 코드를 통과하지 못하면, 많은 경우에

반례를 찾아내야 합니다. 특정 엣지 케이스의 경우에 코드에 에러를 가져다 주기 때문이죠.

인프라로 인한 장애를 제외하고 대부분의 장애는 엣지케이스에서 발생합니다.

![참고자료2](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/2.png?raw=true?raw=true)
![참고자료3](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/3.png?raw=true?raw=true)

보통 테스트코드를 작성하면 위 그림과 같이 코드를 작성합니다. 그러나, 위와같은 테스트 코드는
유명무실한 코드입니다. 마치 알고리즘 코딩 테스트 문제를 푸는데, 주어진 예시코드만 통과시키는
코드와 같습니다. 실제 코딩테스트에서는 예상치 못한 예외로 인해서 테스트 통과를 하지 못하는 경우가 많이 있습니다. 테스트시에 만든 예제(given)는 무의식적으로 성공에 편향될수 밖에 습니다. 알고리즘 테스트 처럼 제약조건이 명시적으로 들어나 있지 않기 때문에 엣지케이스를 찾기도 어렵다는 문제를 가집니다.


![참고자료4](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/4.png?raw=true)

### 2. 객체가 가진 데이터 항목이 매우 많다면?! 혹은 변동성이 크다면?

네이버 쇼핑 주문서 객체의 도메인 필드는 7709개의 필드를 가지고 있다고 합니다.
자바에서 객체의 경우는 많은 외부의 코드 변동성에 대비해서 확장성에 신경쓰는 코드를 작성합니다. 예를 들어, inteface를 사용한다든지, 디자인패턴을 활용하는 방식이 있습니다. 그러나 테스트 코드는 이런 변동성에 매우 취약합니다. fixture monkey는 이런 변동성에도 대비할수 있는 확장성을 제공할 수 있습니다.

## 3. Fixture Monkey 필요성,장점

![참고자료5](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/5.png?raw=true)

  - fixture monkey는 간편하게 다양한 제약조건을 가진 테스트 객체를 생성할수 있습니다. 네이버 주문서처럼 수많은 데이터를 가진 객체를 종류별로 테스트 코드를 만드는 것은 쉬운 일이 아닙니다. fixture monkey는 이를 위한 최고이 도구입니다.
  - 설정한 제약조건을 검증할 수도 있습니다. 예를 들어 entity 객체의 validation 기능을 검증할수 있습니다. 또한, 테스트케이스마다 객체를 다르게 제어해, 다양한 테스트 환경을 제공하기도 좋습니다.
  - fixture monkey는 객체에 랜덤값을 던져줍니다. 이 임의성은 일반적은 케이스에서 생각하지 못한 예외값을 던져, 사용자가 만들 수 있는 다양한 예외상황에 대비하는 코드를 작성할 수 있도록 도와줍니다.

![참고자료6](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/6.png?raw=true)

  - 위 코드에서 Order Class는 다양한 validation 조건을 가지고 있는 객체입니다.
  - fixture monkey는 그 값의 다양한 조건을 맞춘 Order class를 매우쉽게 만들어 냅니다.

![참고자료7](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/7.png?raw=true)

  - fixture monkey는 객체 내부의 또다른 객체값의 생성도 위 코드와 같이 아주 쉽게 할 수 있습니다.
  - 또한 테스트의 반복시행으로 임의 생성 객체의 다양한 장점을 활용해, 최대한 많은 예외케이스를 손쉽게 테스트 할수도 있습니다.

![참고자료8](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/8.png?raw=true)

  - fixture monkey의 데이터는 위 그림처럼 다양한 범위의 예상치 못한 입력값이 입력되므로, 유효하지 않은 값이 입력되는 경우를 테스트할 수 있습니다.
  - 또한 가끔 빠뜨린 예외처리도 잡아낼 수 있는 기회를 제공합니다.

## 4. Fixture Monkey의 단점
![참고자료9](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/9.png?raw=true)
![참고자료10](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/10.png?raw=true)

  - fixture monkey는 위 그림처럼 Rest Doc Api와 함께 사용할때, 랜덤 데이터가 그대로 보이게 되는데, 이는 문서가독성을 헤치는 요인이 될 수 있습니다.
  - 출시된지 얼마되지 않은 API 이기 때문에 철저히 공식문서에만 의존해서 학습할 수 있다는 점은 단점으로 볼 수 있습니다.
  - 출시 된지 얼마 안되어, 안정성에 대한 검증이 되지 않았다고 볼 수 있습니다.(그러나, 실제 네이버페이 사내에서 활발하게 사용하고 있는 것으로 봐서, 어떤 측면에서는 안전성에 대한 검증이 끝났다고도 볼 수 있을 것 같습니다.)

## 5. 간단한 사용법

**![참고자료11](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/11.png?raw=true)**

  - fixturea monkey를 사용하기 앞서, fixture monkey 객체부터 만들어 주어야 합니다. fixture monkey는 객체 생성방법을 다양한 방식으로 제공합니다. 한편, 모든 객체 생성방법을 다 활용하는 방법 은 위 코드와 같이 FailoverIntrospector 객체를 활용하면 됩니다.

![참고자료12](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0130/12.png?raw=true)

  - 위와 같이 간단하게 다양한 객체 리스트 샘플을 만들수도 있습니다.

---
**참고자료, 사진출처** 

https://naver.github.io/fixture-monkey <br>
https://github.com/naver/fixture-monkey <br>
https://jiwondev.tistory.com/272 <br>
https://tv.naver.com/v/23650158

---
