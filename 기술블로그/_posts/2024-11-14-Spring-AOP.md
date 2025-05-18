---
layout: post
title: 'Spring AOP'
author: '이재윤'
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ '자바','SpringAOP' ]
---

# Spring AOP

**AOP란?**
> 관점 지향 프로그래밍
> <br>횡단관심사의 분리를 허용함으로써 모듈성을 증가시키는 것이 목적인 프로그래밍 패러다임

여기서 나오는 횡단 관심사란 무엇일까요?

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdaUrh85x6g7lGsXEFCRgN_HA572UxHyG3UWLJlSi4-XJDRy3-lRd9s3Z239cvbvLT5GILIxkzxgxFzH2UjnL52XzlRyRvOeYIMEo_AbaSIfUMEzwZ3EUIkzl9MAQqJPexPZT395Q=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)

여기서 부가기능을 횡단 관심사(Cross-Cutting Concerns)라고 합니다.
횡단 관심사란 여러 모듈에 걸쳐있는 공통의 기능을 의미합니다.
예를 들어 로깅, 트랜잭션 관리, 보안/인증 등이 횡단 관심사에 해당합니다.

하나의 클래스가 있고 해당 클래스를 추적하기 위해 로그를 찍을 때, 클래스의 변경이 일어나게 된다면 어떻게 될까요?<br>
연관된 로그가 적다면 문제가 없지만 50개, 100개라면 여기저기 퍼져있는 로그를 찾으며 수정해 주어야 할 것입니다.<br>

기존에 이런 문제가 있었고 이를 해결하기위하여 핵심 요소와 부가 요소를 분리하여 한 곳에서 관리하게 됩니다.

## AOP 용어

AOP에서 등장하는 용어가 있습니다. 해당 용어를 한번 알아보도록 하겠습니다.

### Target

> 핵심 기능을 담고있는 모듈로서 부가 기능을 부여할 대상

주로 핵심 비즈니스 로직이 구현된 클래스나 메서드가 Target이 됩니다.

### Aspect

> 일련의 관심사를 모듈화하는 클래스

### Pointcut

> 어드바이스를 적용 할 타겟의 메서드를 선별하는 표현식

- execution
- within
- args
- this
- @Annotation

포인트컷 표현식에 쓰이는 포인트컷 지시자입니다. 이외에도 여러가지가 있습니다. 
주로 execution을 많이 사용하므로 어떻게 쓰는지 한 번 알아보도록 하겠습니다.

#### execution
![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUes9-b8kWKU7Hd5QekBsUUuD4Buz6KBqzwUqlpsHySTBkMplVHCQvsv3WtoKJNEswDp00vOvaBnIPq5qiRF6wlv054Yik26TGpeeQdtiBGf9JolMwuuknj0rWBo-0ZoDuwMXcSmuA=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)
execution의 위 그림과 같은 구조로 되어있고, 위의 표현식을 해석하면<br>

>리턴 타입은 모두 aspects.trace.demo 패키지 하위 모든 클래스와 메소드, <br>
> 그리고 메서드의 파라미터 타입과 파라미터 수에 상관없이 적용

`*` : 모두 허용<br>
`..` 갯수가 상관없음

### Advice

>특정 Join Point(메소드)에 실행되는 Aspect

Aspect가 어느 시점에 시작되는지를 정하는 부분입니다.

| Advice| 설명|
|----------|---------------------------------|
| `@Before`| 대상 메서드의 수행 전|
| `@After`| 대상 메서드의 수행 후|
| `@After-returning`| 대상 메서드의 정상적인 수행 후|
| `@After-throwing`| 예외 발생 후|
| `@Around`| 대상 메서드의 수행 전/후|


### Join Point

>메소드의 실행이나 예외 처리와 같은 프로그램의 실행 중의 포인트<br>
>Spring AOP 에서는 항상 `메소드 실행`을 의미함

### Weaving

>Aspect를 실제로 적용하는 과정

- **Runtime Weaving(RTW)**
- Load Time Weaving(LTW)
- Compile Time Weaving(CTW)

Spring AOP는 RTW를 사용하며, 프록시 패턴을 활용하여 위빙을 수행합니다.

# AOP 간단 적용

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUeNDALYw4P84YOud7i-IADM2J1GTEdBOT-7wdiUgOvfxjo-JAAMuH6SOreB4HrCK_jtkV_fzX9tW93OEh_9OBgJ18yXXeSj60QJnUERqDXfZoWwvdUuj6XfMewoR_yvLkH9Sjg9OA=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)

위 서비스에서 AOP를 사용하여 Logging을 하려면 어떻게 해야 할까요?

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUfg2OHNfAi21uFDgvatDgqoVxJgjJ2-NWWkzGYEIXtrSY4701n_6v__-0lRtchNzeMbm4XSYnV80DD0AYzcbrlS7qchDD_aDGUcyahgWC1bniRSwf-7pgDrKk_dk7bhuYiyH8HVfw=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)

우선 Spring AOP를 사용하기 위해서는 의존성을 추가해주어야 합니다.

!![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdxbu8ffKIfds6lkgMYEewxLhq_iAKJGt-FP2xAJk5b63X6oOSv2bEco3FxRlgNGNxKM3aohlLpkWAlSG1z7Nz-w_MLhz2rU6Xf8HO3JfBQColl-O_z-QljvXGb7U_Pc-aDv70PGA=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)

이후 해당 관심사를 모아둔 Aspect클래스를 작성해줍니다.<br>
@Aspect 어노테이션을 달아주고 빈에 등록하기 위해 @Component 어노테이션을 달아주었습니다.<br>
이후 execution 표현식을 사용하여 패키지,클래스,메소드까지 지정해주었습니다.<br>
그리고 Advice를 지정해주고 해당 지점에 맞게 로그를 적어주었습니다.<br>


|- ![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUfHlnBZapYTBucHuOHUDZcn4OCoBKpk7D5eD--jkGzxm-L_luHh5N6Cx-leRE-Mv-Z7JlCKwKlU60Gi_Tc375DxrfBgBhg2aAml64DsjZr4tcQCEoPHd-POBpCUUqo5Epo1YTDoNA=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu) - | - ![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUeR6KwF23pMtvkNznqUaKzT7f5GVIL8apUdvou2Z4hRpPTzFw5U0dV6m6a-9Ly8aMaAC4OoPMgit47rT3_3RPL5LzCZtuNePiMW6YhaQlvmVcba81n5vZufBjabMkQyNyFauYl5=s2048?key=xiE42Nayb1kM7oYsNzhPJTvu)  -|
| - | - |

위 과정을 거치고나면 기존과 똑같이 동작하지만 `핵심 요소`와 `부가 요소`는 분리가 됩니다.


# AOP 적용 추천

1. 로깅
2. 보안 검사
3. 모니터링
4. 예외 처리
5. 캐싱
6. 데이터 검증
7. 트랜잭션


