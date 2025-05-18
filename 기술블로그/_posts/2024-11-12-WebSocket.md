---
layout: post  
title: WebSocket
author: 임건우
banner: 
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [WebSocket, 기술세미나]
---




# WebSocket

프로젝트에서 채팅을 구현할 때 WebSocket을 사용했다. 물론 HTTP를 사용해서 채팅을 구현 할 수는 있는데 왜 WebSocket을 사용해야 했었고 실시간 통신에는 WebSocket이 더 효율적인 선택인지 설명할거다. 

🔥 WebSocket: 기존의 단방향 HTTP 프로토콜과 호환되어 양방향 통신을 제공하기 위해 개발된 프로토콜

- 특징
  - 일반 Socket 통신과 달리 HTTP 80 Port를 사용해서 방화벽에 제약이 없다
  - 접속까지는 HTTP 프로토콜을 이용하고, 그 이후에는 자체적인 WebSocket 프로토콜로 통신
  - HTTP 통신의 특징인 (연결 -> 연결 해제) 때문에 효율이 매우 낮아서 2014년 HTML5에 웹 소켓을 포함하게 됨. WebSocket은 클라이언트가 접속 요청을 하고 웹 서버가 응답한 후 연결을 끊는게 아니라 Connection을 유지하고 클라이언트의 요청 없이도 데이터 전송가능한 프로토콜임


## HTTP vs WebSocket

### HTTP

- 클라이언트가 요청을 보내면 서버가 응답하는 단방향 통신
- 비연결성 (응답 받으면 연결 종료), 무상태성 (서버가 클라이언트의 상태를 가지고 있지 않음)
- 매 요청마다 헤더 정보 포함

### WebSocket

- 초기 연결 설정 후 양쪽에서 요청 보낼 수 있음, 양방향 통신
- 한 번 연결되면, 한쪽에서 닫을 때까지 유지, 실시간성
- 초기 연결 후에는 최소한의 오버헤드로 데이터를 전송

> HTTP 만으로도 원하는 정보를 송수신 할 수 있다. 하지만 실시간성을 보장해야하고, 변경 사항의 빈도가 잦고, 대기 시간이 짧은 경우에는 WebSocket을 권장하고. 그 반대로 변경 사항의 빈도가 자주 일어나지 않고, 데이터 크기가 작은 경우 `Ajax, Streaming, Long Polling` 기술이 더 효과적이다.


<img src="https://github.com/Kernel360/blog-image/blob/main/2024/1112/websocket_lifecycle.png?raw=true" alt="WebSocket Lifecycle" width="500">

출처: https://velog.io/@mw310/Stomp-WebSocket-개념-정리ver-Spring

WebSocket Life Cycle을 쉽게 잘 설명해주는 사진이여서 첨부를 함.



## STOMP (Simple Text Oriented Messaging Protocol)

🔥 STOMP: 간단한 텍스트기반 메시지 프로토콜이다. 클라이언트와 서버가 전송할 메시지의 유형, 형식, 내용들을 정의

WebSocket을 통해 메시지를 전달할 때 사용되는 Protocol이며 SpringBoot에서 적용하기 가장 쉬운 방법이자 메시지 브로커를 활용해서 Pub-Sub(발행 - 구독) 방식으로 클라이언트와 서버가 쉽게 메시지를 주고 받을 수 있도록 도와준다.

<img src="https://github.com/Kernel360/blog-image/blob/main/2024/1112/pub%3Asub3.png?raw=true" alt="WebSocket Lifecycle" width="500">

출처: https://brunch.co.kr/@springboot/695

**그림을 통해서 쉽게 Pub-Sub이 뭔지 알아보자**

> 사용자 1 & 사용자 2 = no01 구독중
> 
> 사용자 3 = no02 구독중

1. 발행자 메시지의 타겟을 no01로 설정해서 메시지를 보냄
2. 발행자의 메시지 발행 url은 /pub/hello 이고, 서버가 메시지를 받는다
3. 서버에서는 발행자의 메시지를 확인 후 no01 채털을 구독하는 모든 사용자 (클라이언트)에게 메시지를 보냄
4. 구독 URL 이 다른 사용자 3은 메시지를 받지 못함

블로그에서 정말 간단하게 설명해준 글이 있는데, 개인 우편함을 예를 들어서 설명을 해주었다. 각 구독자는 각각의 큐를 갖고 있고, 그 큐를 각각의 개인 우편함이라고 생각하면 된다. no01은 조선일보 신문, no02는 한겨례 신문이라고 하면 사용자 1,2는 조선일보 신문만 받아야하고, no02는 한겨례 신문에서만 받아야한다. 결국엔, 각자 구독중인 곳에서만 메시지를 받아야 하고, 각자 우편함이 따로 존재할것이다.

## SpringBoot에서 WebSocket 사용하는 방법

spring guide: https://spring.io/guides/gs/messaging-stomp-websocket

위 사이트에서 WebSocket의 설명을 읽고 테스트해도 좋다. 정말 쉽게 설명해줬고 깃 clone만 하면 직접 실행해서 동작을 확인할 수 있다. 아래에는 위에서 배운 내용들을 짧게 설명할거다.


- `*코드 구현*`

	### WebSocketConfig

    ```java
	@Configuration
	@EnableWebSocketMessageBroker
	public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

		// 연결 후 message를 보내고 받기 위해 사용하는 메서드
		@Override
		public void configureMessageBroker(MessageBrokerRegistry config) {
			// 먼저 구독을 한 후 메시지를 보내야 한다
			// 1. 구독(subscribe) 2. 발행 (publish)
			config.enableSimpleBroker("/sub");

			// 사용자가 서버에 메시지를 전송할 때
			// 어디로 도착해야할 지 prefix로 정의하고 있음
			// 2개 이상 설정 가능
			config.setApplicationDestinationPrefixes("/pub");
		}

		@Override
		public void registerStompEndpoints(StompEndpointRegistry registry) {
			// js 에서 웹소켓 접속 시
			// var socket = new SocketJS("/omocha-websocket")
			// 일반적인 브라우저에서 websocket 문제가 발생할 수 있어 -> 호환성이 있는 SockJS를 사용
			registry.addEndpoint("/websocket").setAllowedOrigins("*")/*.withSockJS()*/;
		}
	}
	```

먼저 Config 파일을 설정한다. Config 파일에는 WebSocket의 EndPoint, 구독 경로, 발행 경로를 등록해야한다.
중요한 Annotation과 메서드들을 간략하게 설명을 하겠다.

- `@EnableWebSocketMessageBroker`
  - 메시지 브로커가 지원하는 WebSocket 메시지 처리를 활성화
- `enableSimpleBroker()`
  - **/sub** 경로로 클라이언트들에게 Simple Memory-Based Message Broker을 사용해서 메시지를 전송한다
- `setApplicationDestinationPrefixes()`
  - **/pub** 경로로 메시지가 발행되면 서버가 먼저 메시지를 수신한 후 구독한 클라이언트들에게 메시지를 전달한다  
- `addEndPoint()`
  - **/websocket** 이라는 endpoint로 websocket을 연결한다


***만약 WebSocket 연결을 테스트하고 싶으면 어떻게 하나?***

WebSocket을 연결하고 Test를 하려면 프론트코드를 직접 JavaScript로 작성해서 연결을 해보는게 가장 빠르다. 옛날에는 Apic이나 Postman에서도 연결을 테스트 할 수 있는 환경이 제공되었지만, 지금은 중단된 상태인 것 같다.


### Controller

```java
@Controller
public class MessageController {

	// /message/connect
	// 메시지가 오는게 아닌 접속 했을 때 메시지를 예시로 보내주는 것
	@MessageMapping("/pub")
	@SendTo("/sub/greetings")
	public Greeting hello(HelloMessage message) throws Exception {
		Thread.sleep(1000); // simulated delay
		return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
	}

}
```

이제 메시지를 어떻게 받고 전송하는지 설명할거다. 

- `@MessageMapping()`
  - @MessageMapping annotation은 /pub 경로로 메시지가 전송되면, hello() 메서드가 실행된다
  - Spring에서 @RequestMapping이랑 매우 유사하다
- `@SendTo`
  - /pub 경로로 메시지가 전송되고 메서드가 실행되면 /sub/greetings 경로로 구독자들에게 메시지가 Broadcast 된다 (모든 구독자들에게 메시지를 전달)
  - 전송되는 메시지는 위에서 만들어진 Greeting 객체가 전송이 될 것이다

## 마무리

WebSocket을 통해서 채팅 메시지를 구현하는 방법은 매우 쉽다. 다만 현재 테스트하기 위해서 프론트엔드 코드를 작성해야 하는것은 백엔드 개발자들에게 조금 어려움이 있지만 프론트 코드 작성하는 방법이 인터넷에 많기도 하고 나름 구현하기 간단해서 pub/sub의 간단한 원리만 알고있으면 도전 해볼만하다. 본인이 프로젝트에서 채팅이나 챗봇을 구현하고 싶을 때 가장 좋은 방법은 WebSocket을 사용하는 방법이며 아주 간단하니 겁먹지말고 모두 시도해보길 바란다.




## 참고

https://velog.io/@mw310/Stomp-WebSocket-개념-정리ver-Spring

https://velog.io/@mw310/Stomp-WebSocket-개념-정리ver-Spring

https://brunch.co.kr/@springboot/695

