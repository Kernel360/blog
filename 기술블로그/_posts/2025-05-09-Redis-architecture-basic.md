---
layout: post  
title: "Redis 아키텍처 기초"
author: "신윤상"
categories: "기술블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Redis"]
---

# Redis 개념과 고가용성 아키텍처 정리

## Redis란?

**Redis(Remote Dictionary Server)**는 이름 그대로 원격에서 사용할 수 있는 사전(Dictionary) 형태의 자료구조 서버입니다.  
Key-Value 데이터 구조를 기반으로 다양한 형태의 자료구조를 지원하며, 인메모리 기반의 NoSQL 데이터 저장소입니다.

### 주요 특징

- **메모리 기반 저장**으로 매우 빠른 데이터 접근이 가능합니다.
- **다양한 자료구조 지원**: `String`, `List`, `Set`, `Sorted Set`, `Hash`, `Bitmap`, `HyperLogLog`, `Geospatial` 등
- **다양한 활용 사례**:
  - 캐시(Cache)
  - 세션 저장소(Session Store)
  - 실시간 순위표(Leaderboard)
  - pub/sub 메시징 시스템
  - 분산 락 구현 등

---

## Redis의 캐시 활용

캐시는 쓰기보다 읽기 요청이 많은 데이터에 적합한 저장 전략입니다.  
Redis는 메모리 기반이기 때문에 응답 속도가 빠르며, 자주 조회되는 데이터를 미리 적재해두고 사용할 수 있습니다.

- **장점**:
  - 빠른 데이터 응답
  - 시스템 부하 감소
  - 백엔드 DB의 요청 수 절감

---

## Redis의 동작 방식

Redis는 싱글 스레드 기반으로 작동하며, 이벤트 루프(Event Loop)를 통해 클라이언트의 요청을 하나씩 처리합니다.

- 명령어는 **이벤트 큐**에 적재되며, 싱글 스레드로 순차 처리됩니다.
- **멀티 스레드 환경에서 발생할 수 있는 데드락, 컨텍스트 스위칭 비용**을 피할 수 있는 구조입니다.
- **단점**: 대용량 데이터를 처리할 때 단일 스레드로 인해 병목이 생길 수 있습니다.

> Redis는 싱글 스레드 기반이지만, 일부 최신 기능에서 백그라운드 I/O 작업(예: RDB 저장, AOF Rewrite)은 멀티 스레드로 처리됩니다.

---

## Redis의 영속성(Persistence)

메모리에 데이터를 저장하지만, 영속성을 보장하기 위해 디스크 저장도 지원합니다.  
Redis는 다음과 같은 방식으로 영속성을 구현합니다:

### RDB (Redis Database)

- 메모리 전체 데이터를 일정 주기로 스냅샷으로 저장
- 스냅샷 파일 하나로 빠르게 백업 및 복구 가능
- **단점**: 마지막 저장 시점 이후의 데이터는 유실될 수 있음

### AOF (Append Only File)

- 데이터 변경 명령을 모두 기록
- 가장 최근의 데이터까지 복원 가능
- **단점**: 파일 크기 증가, 복원 속도 느림

> 실무에서는 RDB + AOF 방식을 함께 사용해 **백업 속도 + 데이터 무결성**을 함께 확보하는 것이 일반적입니다.

---

## Redis 장애 대응 전략

단일 Redis 인스턴스로는 장애 발생 시 서비스 전체가 중단될 수 있으므로, 일반적으로 마스터-레플리카 구조를 사용합니다.

### 레플리케이션 구조

- **Master 노드**는 쓰기 작업을 처리하며,
- **Replica 노드(Slave)**는 데이터를 동기화 받아 읽기 작업을 분산하거나 장애 대응 용도로 활용됩니다.

---

## Redis Sentinel 아키텍처

![Sentinel Architecture](https://github.com/Kernel360/blog-image/blob/main/2025/0509/sentinel.png?raw=true)

Redis Sentinel은 Redis 클러스터의 고가용성을 확보하기 위한 모니터링 및 장애 조치 시스템입니다.

### 구성 요소

- Redis Master (1개)
- Redis Slaves (여러 개)
- Sentinel 서버 (3개 이상 권장)

### 기능

- **Monitoring**: 마스터 및 슬레이브의 상태를 감시합니다.
- **Automatic Failover**: 마스터 장애 발생 시 슬레이브 중 하나를 자동 승격합니다.
- **Notification**: Pub/Sub을 통해 장애 발생 정보를 클라이언트에 전달합니다.

### 동작 방식

- Sentinel 간 다수결 투표로 마스터 장애를 판단
- 새 마스터를 선출하고 나머지 슬레이브는 해당 마스터를 기준으로 재구성됩니다

---

## Redis Cluster 아키텍처

![PHANTOM READ](https://github.com/Kernel360/blog-image/blob/main/2025/0509/cluster.png?raw=true)

Redis Cluster는 **수평 확장성**과 **내장형 장애 복구 기능**을 제공하는 아키텍처입니다.

### 주요 기능

- **샤딩(Sharding)**: 키 해시값을 기준으로 데이터를 분산 저장합니다.
- **고가용성 유지**: 마스터 장애 시 레플리카가 자동 승격됩니다.
- **Sentinel 불필요**: 클러스터 자체적으로 상태를 관리합니다.

### 동작 방식

- 각 키는 CRC-16 해시 → 해시 슬롯(0~16383)으로 매핑됩니다.
- 슬롯을 여러 마스터 노드에 나눠서 분산 저장합니다.
- 각 마스터는 하나 이상의 레플리카 노드를 가집니다.
- 노드들은 **Gossip Protocol**을 사용해 서로의 상태를 지속적으로 감시합니다.

---

## Redis 아키텍처 비교 요약

| 항목            | Sentinel 아키텍처                              | Cluster 아키텍처                             |
|-----------------|------------------------------------------------|---------------------------------------------|
| 장애 복구       | Sentinel 프로세스가 담당                      | 클러스터 자체적으로 gossip 기반 수행         |
| 샤딩            | 미지원 (수동 분산 처리)                       | 지원 (자동 분산)                            |
| 데이터 일관성   | Master-Slave 복제                              | Master-Replica 구성                         |
| 구성 복잡도     | 낮음                                           | 높음                                        |
| 사용 목적       | 고가용성 (HA), 장애 복구용                      | 분산 처리, 고가용성, 대규모 시스템 대응       |

---

## 결론

Redis는 단순한 인메모리 캐시를 넘어, 고성능 분산 환경에서도 견고한 아키텍처를 구성할 수 있는 강력한 시스템입니다.  
특히, Sentinel과 Cluster 아키텍처를 통해 장애 대응과 수평 확장이 가능하여 **대규모 트래픽 처리** 및 **고가용성 서비스**에 매우 적합합니다.
