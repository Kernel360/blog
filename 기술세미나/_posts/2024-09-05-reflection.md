---
layout: post
title: 'reflection'
author: '이재윤'
categories: '기술세미나'
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ['자바','reflection']
---


JAVA의 Reflection을 알아보기 전 용어를 정리하겠습니다.

#### 바인딩( Binding )
> 컴퓨터 프로그래밍에서 각종 값들이 확정되어 더 이상 변경할 수 없는 구속(bind) 상태가 되는 것.

###### 정적 바인딩( static binding )
> 컴파일 시점에 값과 타입이 정해지는 것

###### 동적 바인딩 ( dynamic binding )
> 런타임 시점에 값과 타입이 정해지는 것


#### 컴파일 ( Comfile )
> 소스 코드를 다른 프로그램이나 기게( H/W )가 처리하기 용이한 형태로 변환하는 과정
> <br>이 과정중에 오류가 발생한다면 ' 컴파일 에러 '

#### 런타임( Runtime )
> 컴퓨터 프로그램이 실행되고 있는 환경 또는 동작되는 동안의 시간
> <br>이 과정중에 오류가 발생한다면 ' 런타임 에러 '


---

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUfOFmyrTifGF38gvGKmSiQ3vWIqrRT5TEOjfp9AVwk8aDv6H1VbsIKqAehOS7LkKtpPV0VyvuOVqy-PAfnmhm5eJP2Tn_DbxV21IIObv6dtV3sCjR8MquFybgRy9fYjusdqtqv6Chgjl8lL6EiSpwdmc71DvwNg=s2048?key=U26xYz3X5V_UspB8XcM_vw)

리플렉션은 자바가 1996년 처음 등장하고 그 다음 버전에 추가되었습니다. 지금까지 많은 기능이 추가 되고 또 삭제가 되었습니다. 필요가 없거나 안 좋다면 사라졌을텐데 어떤 특징때문에 아직까지 사라지지 않고 사용되고 있을까요?

## Reflection이란?

>Runtime에 동적으로 특정 Class의 정보를 추출할 수 있는 프로그래밍 기법

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdbFkeKr8uXitaIK_BvsM4MpDG59I7VMM413ejcb0h_e7z_vEmurgrDSDgMjhTzqh_BVuq7AO6HD9fIE2etGLoEksAjpdmKRDF6zBf_Dg5EoBzjgXmzcZ4IEb5eWHp3Hn4kWiRNzRZL1vgS_xSMBqjP6_Xy6M4=s2048?key=U26xYz3X5V_UspB8XcM_vw)
<br>위 그림은 JAVA17 버전 API 문서로 다음과 같이 소개를 하고 있습니다.

- 클래스와 객체에 대한 반사적인(reflective) 정보를 얻기 위한 클래스와 인터페이스를 제공
- 이를 활용해 객체를 생성하거나 메소드를 호출하는 등의 작업을 수행

여기서 말하는 반사적인(reflective) 정보란 무엇일까요?


## Reflection 동작 과정

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUfjUsRd4TJ0wc-nvf0HqvBr4EP2qIP7MYm8Xfx-6KFz5Lw2kApJBgUDYj1rSmufLQkREOBo0VUfyspRvT8Vl0der6zds_9sjpbh70gA-ox0kiCngsoSmUSTKogHuy_eVy7YuFdDHTZalSUI_ndovTukmSLEd_vA=s2048?key=U26xYz3X5V_UspB8XcM_vw)

저희가 일반적으로 코드를 작성하고 실행하게 된다면 `소스 코드`가 자바 컴파일러에 의해서 `클래스 파일`로 변환이 되고 클래스 로더에 의해서 `Runtime Data Areas` 의 `Method Area` 에 적재가 됩니다.

리플렉션은 런타임 시점에  `Method Area` 에 저장된 클래스 정보를 클래스 로더를 통하여 사용하게 됩니다.
일반적인 코드와 다르게 역순으로 진행하고 동적 바인딩이 되는 것을 알 수 있습니다.

일반적인 컴파일 과정
- 소스 코드 -> 클래스 파일 -> Method Area
- 정적 바인딩
  <br>reflection 과정
- Method Area  접근 -> 객체 생성
- 동적 바인딩

> Method Area
> <br>JAVA 7 : Heap영역 - PermGen
> <br>JAVA 8 : Native memory - Metaspace

## Reflection Method

java의 reflect 패키지를 이용하면 클래스 , 생성자 , 필드 , 메소드 등 여러 정보에 접근을 할 수 있습니다.
그중 메소드의 공통된 특징 몇가지를 알아보겠습니다.

- getXXX( )
  - public
- getDeclareXXX( )
  - 모든 값
- setAccessible( boolean flag )
  - flag를 'true'로 설정시 접근제어자를 무시


이런 메소드를 보게 되면 보안에 해당하는 접근제어자를 무시 할 수 있는 특성이 있고 이를 활용하여 접근이 불가능한 객체를 테스트에 이용하는 경우도 있습니다. 단점으로 객체지향 설계 원칙인 캡슐화가 깨질 수 있고 보안 상 문제가 될 수도 있습니다.



## 객체 생성 예제

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdW7ajW3K9y-IiwTQXhiH-P6YueQlh9I9vEx9HkmYhSft8tdmMaVDkHAhL0gFSGIuF0iy6fJI0FvIrZK7jxPyQ7-hslRHMB9POefimJSrewy8Rwx1kDCtre3Rxv8ULWLOeA6HjvFZ1bso-Epb7o5wsIGgnaOiA=s2048?key=U26xYz3X5V_UspB8XcM_vw)
<br>객체 생성 예제를 간단하게 알아보겠습니다.
1. `class`를 알고 있을 경우 해당 클래스의 메타데이터에 접근하여 객체를 생성합니다.
2. 문자열로 주어진 클래스 이름을 사용하여 `Class` 객체를 동적으로 로드합니다.
3. `carClass`에서 기본 생성자를 호출하여 `Car`의 인스턴스를 생성합니다. 클래스가 기본 생성자를 갖고 있어야 하며, 접근이 가능해야 합니다.


## Reflection 정리

장점
- 유연한 코드 작성
  - 동적 바인딩
    단점
- 캡슐화 위반
- 성능 저하
  - 최초 실행시 JIT 컴파일러의 최적화를 받지 못함
- 런타임 오류


자바 리플렉션은 프로그램의 유연성과 확장성을 높여주는 강력한 기능입니다. 하지만 성능 저하, 보안 문제, 안정성 저하 등의 단점도 있으므로, 신중하게 사용해야 합니다.
리플렉션은 `스프링 프레임워크` , `자바 직렬화` , `테스트 코드 작성` 등 다양한 분야에서 활용됩니다. 왜냐하면 리플렉션을 통해 런타임에 동적으로 코드를 조작할 수 있기 때문입니다.
이러한 이유로, 리플렉션은 자바 개발자가 반드시 이해하고 있어야 할 중요한 개념 중 하나입니다.
