---
layout: post
title: Spring Security Authentication 구조 훑어보기
author: 안소현
banner:
  image: https://github.com/Kernel360/blog-image/blob/main/2023/1120/11.png?raw=true
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [spring, spring security]
---

안녕하세요. 스프링 시큐리티의 전체적인 인증 구조에 대해서 발표할 안소현입니다. 

스프링 시큐리티, 무엇일까요?

스프링 시큐리티는 스프링 기반의 애플리케이션 보안을 담당하는 스프링 하위의 프레임워크입니다. 

우리가 시큐리티 로직을 짤 때 흔히 인증과 인가으로 나눌 수 있죠. 

인증(authentication)은 사용자의 신원을 입증하는 과정으로 사용자가 사이트에 로그인을 할 때 누구인지 확인하는 과정이고 인가는 사이트의 특정 부분에 접근할 수 있는지에 대한 권한을 확인하는 작업이죠!

앞으로 발표할 내용은 spring docs에 나와있는 내용을 기반으로 작성되어 있습니다. 

스프링 시큐리티 독스에는 정말 많은 내용이 담겨있는데 우리는 오늘 이 중에서

![1](https://github.com/Kernel360/blog-image/blob/main/2023/1120/1.png?raw=true)

서블릿 애플리케이션에서의 스프링 시큐리티를 알아볼겁니다. 


## Delegating Filter Proxy

클라이언트에서 요청이 오면 WAS의 필터체인을 타고 이 체인 안에서 Spring Security 관련 Filter Chain을 거치게 되는데요. 

![2](https://github.com/Kernel360/blog-image/blob/main/2023/1120/2.png?raw=true)

그림을 보시면 SecurityFilterChain이 WASFilterChain에 등록되어 있다기보다는 DelegatingFilterProxy의 한 부분으로 들어간 것 같은 모습을 볼 수 있습니다. 

왜 그럴까요?


Spring Security는 Spring 컨테이너에서 생성된 필터를 사용하여 스프링 컨테이너의 빈으로 등록됩니다. 

하지만 정작 클라이언트의 요청은 서블릿 필터를 기반으로 한 필터 체인을 타게 되죠.  

이때 서블릿 필터는 스프링에서 정의된 빈을 주입받아 사용할 수 없기 때문에 우리가 만든 security 필터를 사용할 수 없게 되죠! 

그래서 서블릿 컨테이너에서 관리되는 프록시용 필터인 DelegatingFilterProxy를 사용하여 WASFilterChain에 우리가 만든 security filterChain을 연결시켜주는 겁니다. 

조금 어렵죠…?ㅋㅋ

![12](https://github.com/Kernel360/blog-image/blob/main/2023/1120/12.png?raw=true)

즉, 정리하자면 간단히 말해서 스프링 빈과 서블릿 컨테이너의 생명주기를 연결하기 위해 DelegatingFilterProxy라는 Filter 구현체를 사용한다고 생각하시면 됩니다.  



## spring security authentication 구조

그럼 이제 spring security authentication 구조로 들어가볼까요?

![3](https://github.com/Kernel360/blog-image/blob/main/2023/1120/3.png?raw=true)

spring security 구조 라고 검색하면 흔히 볼 수 있는 사진, 저도 가져왔습니다. ㅋㅋ

이 구조는 가장 basic인 유저의 id, password를 받는 formLogin의 상황입니다. 

![13](https://github.com/Kernel360/blog-image/blob/main/2023/1120/13.png?raw=true)

먼저 사용자가 http request로 로그인 정보와 함께 인증 요청을 합니다. 이때 Authentication Filter가 요청을 가로채고, 가로챈 정보를 기반으로 UsernamePasswordAuthentication Token을 생성합니다.

UsernamePasswordAuthenticationToken, 이 아이가 결국 구현하고자 하는 것은 Authentication 인데요.

![4](https://github.com/Kernel360/blog-image/blob/main/2023/1120/4.png?raw=true)
![5](https://github.com/Kernel360/blog-image/blob/main/2023/1120/5.png?raw=true)

![6](https://github.com/Kernel360/blog-image/blob/main/2023/1120/6.png?raw=true)

이 Authentication은 추후 우리가 하나의 request 처리 흐름동안 들고다니면서 security 로직에 사용할 security context에 담기는 내용으로 

우리가 흔히 Userdetails에 두었던 principal, 주로 비밀번호를 가져오는 credentials, 권한 정보인 authorities 정보를 담고 있습니다. 


다음으로 AuthenticationManager의 구현체인 ProviderManager에게 생성한 UsernamePasswordToken 객체를 전달합니다. 

![14](https://github.com/Kernel360/blog-image/blob/main/2023/1120/14.png?raw=true)

앞서 말했듯 **ProviderManager는 AuthenticationManager의 가장 일반적인 구현체이고  ProviderManager는 ‘AuthenticationProvider 목록’을 가지고 있습니다.** 

이 목록을 조회하면서 해당 Authentication의 인증을 지원하는 Authentication Provider를 찾아서 인증 역할을 위임합니다. 

![7](https://github.com/Kernel360/blog-image/blob/main/2023/1120/7.png?raw=true)

다음으로 유저의 정보를 인증하는 과정인데요!

**각 AuthenticationProvider는 인증 성공, 실패, 결정할 수 없음을 나타낼 수 있고, 나머지 AuthenticationProvider가 결정을 할 수 있도록 전달합니다. 인증을 결정할 수 있는 프로바이더는 자신의 authenticate메서드를 통해서** 입력으로 들어온 authentication을 자신만의 방식으로 검증하여 성공할 시 인증 성공된 토큰을 리턴하고  실패할 시에는  AuthenticationException을 던집니다.

authentication을 검증할 수 있는 자신만의 방식에는 DB 비교도 있고, 시그니처 비교도 있고,, 여러 가지 방법들이 있겠죠??

![10](https://github.com/Kernel360/blog-image/blob/main/2023/1120/10.png?raw=true)

이제 마지막 코스입니다! 이렇게 생성된 Authentication 객체는 다시 최초의 AuthenticationFilter에 반환되고 이 Authenticaton 객체를 SecurityContext에 저장하면서 인증이 끝나게 됩니다. 

여기까지 spring security의 기본적인 구조였습니다.



