---
layout: post
title: 메시지 지향 미들웨어
author: 고병룡
categories: 기술세미나
banner:
  image: https://docs.oracle.com/cd/E19340-01/820-6424/images/to_MOM.gif
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [메시지 지향 미들웨어, 메시지 브로커, 메시지 큐, Kafka, RabbitMQ, ActiveMQ, 기술세미나]
---

## 메시지 지향 미들웨어란 ?

_**메시지 지향 미들웨어(Message-Oriented Middleware)**_ 는 시스템과 시스템간의 데이터 통신을 위한 소프트웨어,

즉 메시지라는 정보의 단위를 주고 받을 수 있게 하는 중간의 매개체 시스템입니다.

여기서 메시지 지향 미들웨어는 메시지 브로커와 같은 것들을 포함하는 매우 큰 단위로

메시지 브로커로 잘 알려진 ``Apache Kafka``, ``RabbitMQ``, ``ActiveMQ``를 가지고 있습니다.

위에서 소개한 메시지 브로커들을 이해하려면 해당 브로커들에서 사용 중인 메시지 모델에 대한 이해가 필요합니다.

## 메시지 모델

메시지 모델에는 크게 두 가지로 나눌 수 있습니다. ``Message Queue  (메시지  큐)``와 ``Pub / Sub  (게시 / 구독)`` 입니다.

두 모델 다 어플리케이션 사이에 존재하여 각각의 방법으로 메시지를 전달합니다.

### Message Queue (메시지 큐)

![message_queue](https://github.com/Kernel360/blog-image/blob/main/2023/1108/message_queue.png?raw=true)

메시지 큐에서는  메시지 생산자가 메시지를 만들어서 메시지 큐에 보내면 큐에 메시지를 저장하게 됩니다.

이때 만들어진 메시지는 메시지 큐에 담겨있다가 먼저 들어온 메시지가 먼저 전달되는 선입선출의 형태로 메시지 소비자에게 전달되게 됩니다.

이때 소비자는 큐에 연결되어서 메시지가 들어와 있으면 해당 메시지를 확인하고 처리하는 기능을 가지고 있습니다.

메시지 큐에는 2개 이상의 소비자가 연결되어 있을 수 있지만 여전히 메시지는 한번에 하나씩만 전달되는 구조입니다.

따라서 메시지 큐는 다음과 같은 장점을 갖고 있습니다.

* 소비자가 메시지를 한번에 처리하지 않아 비동기 통신이 가능합니다
* 소비자와 생산자가 직접적으로 메시지를 교환하지 않기 때문에 결합도가 낮아서 따라서 시스템 확장할 때도 유리합니다.
* 소비자가 메시지를 바로 처리할 수 없어도 메시지 큐에 보관되기 때문에 시스템에 유연성이 더해집니다.

물론 단점도 존재합니다.

* 초기 설정이 복잡할 수 있고 / 유지 비용이 많이 듭니다. 
* 하나의 메시지 큐 서버를 공유하고 있어서 서버에 장애가 생기면 시스템 전체에 영향을 미치는 단일 장애 지점이 됩니다.  
* 큐에 과도하게 많은 양의 메시지가 들어오거나 과부하가 걸리면 성능에 제한이 걸립니다.

### Pub / Sub (게시 / 구독)
![pub/sub](https://github.com/Kernel360/blog-image/blob/main/2023/1108/pub-sub.png?raw=true)

게시 구독 모델에서는 ``토픽``이라는 단위를 사용합니다. 

여기서 토픽이란 데이터가 개시되는 채널 또는 주제를 의미합니다. 

토픽 게시자는 데이터를 생성하고 메시지를 특정 토픽에 올리는 역할을 하고 토픽 구독자는 특정 토픽을 구독하고, 해당 주제로 게시된 메시지를 수신하고 처리할 수 있습니다. 

구독자는 한번에 여러 개의 토픽을 구독할 수 있고 한 메시지를 여러 구독자가 수신할 수 있습니다.

따라서 게시 / 구독 모델은 다음과 같은 장점을 갖고 있습니다.

* 토픽 기반 메시지를 통해서 구독자가 원하는 주제에 대한 메시지만 받을 수 있고 다수의 수신자가 한 메시지를  처리할 수 있어서 확장성이 뛰어납니다. 
* 소비자는 동시에 여러 토픽에서 메세지를 병렬적으로 받아올 수 있기 때문에 메시지를 수신하는 동안 다른 작업을 수행할 수 있습니다.

단점은 다음과 같습니다.

* 메시지를 토픽 기반으로 수신하기 때문에 메시지 처리 순서가 예측 불가능 하고 또한 메시지가 중복될 수도 있습니다. 
* 구독자는 원치 않은 메시지를 수신할 수도 있기 때문에 보안 문제가 생길 수 있습니다. 
* 메시지 큐와 마찬가지로 관계도를 이어주는데 복잡할 수 있고 유지하는데 상당한 비용이 듭니다.

## 메시지 지향 미들웨어의 종류

* ``Apache ActiveMQ``
* ``RabbitMQ``
* ``Apache Kafka``

### [Apache ActiveMQ](https://activemq.apache.org/components/artemis/)

![apache_activemq](https://github.com/Kernel360/blog-image/blob/main/2023/1108/activemq.png?raw=true)


Apache ActiveMQ는 오픈 소스이자 멀티프로토콜인 메시지 지향 미들웨어(Message Oriented Middleware, MOM)로서, Java Message Service (JMS) 스펙을 구현한 프로젝트 중 하나입니다. 

JavaScript, C, C++, Python, .NET 등 다양한 언어로 된 클라이언트를 지원하고 AMQP, STOMP, MQTT 등의 다양한 프로토콜을 지원합니다.

메시지 큐 및 게시 / 구독 패턴을 지원하며 메시지를 영속적으로 저장하고 전송 보증을 통해 손실 없이 전달할 수 있습니다.

또한 클러스터링을 통해서 확장성을 제공하지만 대용량 데이터 스트림 처리에는 부족할 수 있습니다.

현재 제공하는 버전은 좀 더 전통적인 버전인 ApacheMQ Classic과 ActiveMQ Artemis가 있으며 Classic 버전 만큼의 기능성을 갖게 된다면 Artemis가 ActiveMQ Classic을 대체할 예정입니다.

### [RabbitMQ](https://www.rabbitmq.com/)
![rabbitmq](https://github.com/Kernel360/blog-image/blob/main/2023/1108/rabbit_mq.png?raw=true)

RabbitMQ는 Erlang으로 개발된 오픈 소스 지향 미들웨어로서 AMQP를 구현하여 다양한 어플리케이션 간의 비동기적인 통신을 지원합니다. 

메시지 큐 기반의 메시지 지향 미들웨어로서 메시지를 큐에 저장하고 소비자가 해당 큐에서 메시지를 소모합니다.

소비자 어플리케이션은 조금 더 수동적인 역할을 하며 브로커가 메시지를 푸시할 때까지 기다립니다.

우선 순위 대기열을 사용하여 특정 우선순위가 반영된 메시지를 먼저 보낼 수 있고 그 이외의 경우에는 소비자는 항상 들어온 순서대로 메시지를 받습니다.

메시지는 소모되면 삭제되며 따로 저장되지는 않습니다.

복잡한 메시지 라우팅이 가능하고 ActiveMQ와 마찬가지로 MQTT와 STOMP와 같은 프로토콜을 지원합니다. 

다양한 프로그래밍 언어를 지원하기 때문에 범용성이 좋습니다.

하지만 메시지 큐에 대용량의 메시지가 들어온다면 속도가 느려질 수 있으며 안전하게 교환할 수 없을 수도 있습니다.

### [Apache Kafka](https://kafka.apache.org/)

![apache_kafka](https://github.com/Kernel360/blog-image/blob/main/2023/1108/apache_kafka.png?raw=true)

Kafka는 게시 / 구독 모델을 기반으로 한 분산형 이벤트 스트리밍 플랫폼입니다.  

여기서 이벤트란 실시간으로 데이터에 변경이 일어나는 것을 의미합니다. 

Kafka는 초기 설계부터 실시간으로 생기는 대용량 데이터에 대해서 로그를 남기고 임의의 타이밍에 데이터를 읽을 수 있도록 했고, 메세지에 영속성을 부여하여 로그를 남길 수 있도록 설계되었습니다.

Kafka는 처음에 링크드인이라는 구인구직과 소셜네트워킹을 합쳐놓은 회사에서 시작되었습니다. 

개발될 당시에 링크드인은 사업이 확장되면서 웹사이트 모니터링을 위한 이벤트 로그 시스템을 필요로 했습니다.  

더 구체적으로는 메시지 데이터를 쌓아두고 주기적으로 batch 작업을 통해서 데이터를 처리하는 시스템으로 보내고 동시에 로그도 쌓아서 모니터링을 할 수 있는 미들웨어를 필요로 했습니다. 

이 때, Kafka를 대신해서 선택할 수 있는 것들에 다른 메시지 지향 미들웨어인 조금 더 전통적인 메시지 브로커, ActiveMQ, RabbitMQ와 같은 것들이 있었는데 이 미들웨어들은 앞선 소개에서 설명했듯이 대용량 데이터 스트림 처리에는 부족할 수 있기 때문에 이는 링크드인의 요구사항과는 멀었습니다. 

이에 따라서 Kafka 개발이 시작되었고 처음 의도했던 기획대로 이벤트 중심의 분산형 스트리밍 플랫폼이 필요한 많은 서비스에서 사용되고 있습니다.

Kafka를 사용하는 기업들은 [다음](https://kafka.apache.org/powered-by)에서 확인하실 수 있습니다. 

카프카는 클러스터에 여러 개의 브로커를 가지고 있는 형태로 되어 있고 해당 브로커 내부에는 토픽을 나누어둔 토픽 파티션이 존재합니다. 

토픽 파티션은 여러 개의 레플리카로 존재하여 여러 개의 브로커에 나뉘어져 메시지를 존재합니다.

브로커에 장애가 생긴다면 해당 브로커에 있는 토픽 파티션들을 다른 브로커로 이어줘야 하며 구독자(소비자)가 메시지를 계속 이어 받을 수 있게 관리가 필요합니다.

## 참고 자료
[Kafka와 RabbitMQ의 차이점은 무엇인가요?](https://aws.amazon.com/ko/compare/the-difference-between-rabbitmq-and-kafka/)

[Apache Kafka란 무엇입니까?](https://aws.amazon.com/ko/what-is/apache-kafka/)

[Apache ActiveMQ](https://activemq.apache.org/)

[메시지 큐란?](https://www.ibm.com/kr-ko/topics/message-queues)

[Pub/Sub이란 무엇인가요?](https://cloud.google.com/pubsub/docs/overview?hl=ko)
