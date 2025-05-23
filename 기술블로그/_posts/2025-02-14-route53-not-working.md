---
layout: post
title: "Route53-not-working 해결기"
author: ""
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["Route53"]
---

[카프카1]

![image.png](attachment:630bda56-dd00-4f71-9331-8bc271fe5bab:image.png)

[카프카2]

![image.png](attachment:67a6c7a3-fcb2-47d0-b8fb-6cf4f80c9399:image.png)

[카프카3]

![image.png](attachment:7161bc3c-cba2-4575-a184-34164833c8b2:image.png)

[카프카3]에서 기존 토픽을 상세조회하려고 하는데 위와 같이 timeout이 떴다.

해결을 위해 시도한 방법

1. 위 에러와 동일한 블로그를 찾았다.

    https://wildeveloperetrain.tistory.com/219 블로그를 통해 카프카의 [server.properties](http://server.properties) 정보에서 listeners 정보를 업데이트 해주었다.

    ![image.png](attachment:86ec4f24-bf1a-4a29-afd7-ff1eca45aede:image.png)

    → 그래도 안됨.

2. 주키퍼가 내가 설정한 토픽을 잘 바라보고 있는지 의심이 들었다.

    https://developnote-blog.tistory.com/172 블로그대로

    `./zookeeper-shell.sh 172.31.8.236:2181` 를 통해 직접 주키퍼에 접근했다.

    ![image.png](attachment:bcfdf794-34e5-42ec-97fb-4d53d6e98d8c:image.png)

    주키퍼가 내가 만든 토픽들을 제대로 보고 있었다.

    → 결론적으로 이 부분이 아니었음.

3. 각 서버간의 주키퍼와 카프카가 잘 켜져있는지 확인하기 위해, 9092, 2181 포트가 켜져있는지 확인했다.

    아래와 같이 모두 잘 켜져있었음. → 결론적으로 이 부분이 아니었음.

    [카프카1]

    ![image.png](attachment:9fa9a4f0-e286-4ff4-b3bf-ebb16fd19495:image.png)

    [카프카2]

    ![image.png](attachment:3a1dc6be-e09d-4a02-a3cf-b54e7b17413a:image.png)

    [카프카3]

    ![image.png](attachment:e2be5178-e5fd-424c-afcf-650c706872af:image.png)

4. 토픽을 상세조회할때, 카프카를 통해 토픽 상세조회, 주키퍼를 통해 토픽 상세조회를 했는데 카프카를 통해 토픽을 상세조회할때, timeout이 났다. → 왜?

    `~/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic cycleInfo-json-topic --describe`

    ![image.png](attachment:7d9061c9-dcb3-4bbf-acc9-0463c17ca2c9:image.png)

    `~/kafka/bin/kafka-topics.sh --zookeeper 172.31.11.207:2181, 172.31.3.191:2181, 172.31.8.236:2181 --topic cycleInfo-json-topic --describe`

    ![image.png](attachment:81e8aefe-77d1-4e1c-9d48-74861dfe50ba:image.png)

    껐다 켜보자.

    근데 termius에서 해당 EC2로 접속이 안되더라.

    설마 route53이 문제인가?

    [dig 도메인명 명령어]를 통해 내 자신, 상대방의 카프카 등의 도메인에 맞는 ip인지 확인했다.

    !! 카프카3이 도메인에 맞는 ip가 아니었다.

    기존에 route53에 ec2의 public ip을 넣어주었음에도 불구하고 제대로 동작을 안했던 것이다.

    ![image.png](attachment:cd631130-6117-4c84-abcb-c65ddafbdc5d:image.png)

    처음으로 돌아가자. 난 애초에 토픽을 상세조회하려고하는데 time out이 났고 이로 인해 제대로 동작이 안되었다.

    토픽을 상세조회할때 사용한 명령어가 어떤식으로 동작하는지 확인하기 위해, cat kafka-topics.sh를 입력했다.

    [kafka-topics.sh](http://kafka-topics.sh)는 아래처럼 kafka.admin.topiccommand "$@” 로 되어있었다.

    ![image.png](attachment:516f246e-3db5-4805-b194-b743e93b1e40:image.png)

    [내부 코드](https://github.com/a0x8o/kafka/blob/master/core/src/main/scala/kafka/admin/TopicCommand.scala#L299)를 확인하니, 살아있는 노드들을 확인할때 (liveBrokers) 도메인에 맞는 ip로 접근해야하는데 도메인에 맞지 않는 ip로 접근하다보니, timeout이 발생했던 것이다.

    ![image.png](attachment:6affcf79-838f-4b39-a2ca-b1ccdafa5117:image.png)

    궁극적으로 route53이 제대로 DNS질의를 하지 못한것이었고, 난 /etc/hosts 파일에 도메인에 맞는 ip를 명시적으로 넣어줌으로써 해결할 수 있었다.
