---
layout: post  
title: "동시성 문제를 겪으며 해결한 경험"
author: "양상원"
categories: "기술세미나"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Java", "동시성", "synchronized", "트랜잭션"]
---

안녕하세요. kernel360 크루 양상원입니다.   
파이널 프로젝트를 진행하면서 겪었던 동시성 문제에 대해 해결했던 경험에 대하여 이야기를 진행하고자 합니다.   

## 동시성을 겪었던 상황
프론트에서 태그 이름을 한글로 입력하는 과정에서 0.03초 안에 생성 요청이 2번이 발생하였습니다.
```java
private void validateDuplicateTagName(Long userId, String name) {
    if (tagDataHandler.checkDuplicateTagName(userId, name)) {
        throw ApiTagException.TAG_ALREADY_EXIST();
    }
}
```
위와 같은 로직으로 태그가 중복되지 않도록 하였습니다.   
같은 태그 있는 경우, 예외를 발생시키기 때문에 요청이 여러 번 오더라도 동시성 문제가 발생하지 않을 것이라 예상하였습니다.   
하지만, 요청이 2번 발생했을 때 해당 로직에 검증이 되지 않고 2개의 태그 생성 요청 모두 DB에 저장되었습니다.   

![image1](https://github.com/user-attachments/assets/e3fbd33c-7cac-4582-ae2e-d161b6683bde)
DB에 1개의 태그만 저장되는 것이 올바른데, 2개가 저장되는 문제가 발생하게 된 것입니다.   
그렇다면 동시성 문제를 해결할 수 있는 방안에는 무엇이 있을까요?   
간단하고, 쉬운 방법인 synchronized를 이용하여 문제를 해결하고자 하였습니다.   
synchronized를 사용하면서 겪었던 문제와 해결된 상황을 알아보도록 하겠습니다.
- - -
## synchronized + @Transactional
```java
@Test
@DisplayName("태그 저장 동시성 테스트")
void createTagConcurrencyTest() throws InterruptedException {
    // given
    int threadCount = 20;
    ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
    CountDownLatch countDownLatch = new CountDownLatch(threadCount);
  
    AtomicInteger successCount = new AtomicInteger();
    AtomicInteger failCount = new AtomicInteger();
  
    Long userId = user.getId();
  
    // when
    for (int i = 0; i < threadCount; i++) {
      executorService.submit(() -> {
        try {
          TagCommand.Create command = new TagCommand.Create(userId, "태그12341", 2);
          tagService.saveTag(command);
          successCount.incrementAndGet(); // 성공 카운트
        } catch (Exception e) {
          log.info(e.getMessage());
          failCount.incrementAndGet(); // 실패 카운트
        } finally {
          countDownLatch.countDown();
        }
      });
    }
  
    countDownLatch.await(); // 모든 스레드가 완료될 때까지 대기
    executorService.shutdown();
  
    // then
    log.info("success : {} ", successCount.get());
    log.info("fail : {} ", failCount.get());
  
    assertThat(successCount.get()).isEqualTo(1);
    assertThat(failCount.get()).isEqualTo(threadCount - 1);
}
```
이전에 발생했던 요청과 같은 방식은 아니지만, 멀티 스레드 환경에서 동시성 문제가 발생하는 것을 예시로 가져왔습니다.   
하나의 유저가 20개의 요청을 동시에 보냈을 때 태그 중복 검증이 제대로 이루어지는지 확인하고자 하였습니다.
```java
@Transactional
public TagResult saveTag(TagCommand.Create command) {
    validateDuplicateTagName(command.userId(), command.name());
    return tagMapper.toResult(tagDataHandler.saveTag(command.userId(), command));
}
```
synchronized를 붙이지 않은 기존 코드입니다.   

```java
@Transactional
public synchronized TagResult saveTag(TagCommand.Create command) {
    validateDuplicateTagName(command.userId(), command.name());
    return tagMapper.toResult(tagDataHandler.saveTag(command.userId(), command));
}
```
synchronized를 사용한 코드입니다.   
기존과 다르게 synchronized를 사용하였는데도 태그 중복에 대한 검증이 이루어지지 않는 문제가 발생합니다.   
   
왜??? 이런 문제가 발생했을까요??   
synchronized를 @Transactional과 함께 사용했기 때문입니다.   
그렇다면, synchronized를 @Transactional과 함께 사용해서 생기는 문제에 대하여 알아보도록 하겠습니다.   

우선은 트랜잭션만 사용했을 때, 왜 검증이 제대로 되지 않는지에 대해 알아보도록 하겠습니다.   
저희 프로젝트에서는 MySQL을 사용하고, 실제 로컬 DB와 연결하여 동시성 테스트를 진행하였습니다.      
MySQL의 기본 isolation level은 Repeatable Read이므로 동일 트랜잭션 내에서 일관성이 보장됩니다.   
![image2](https://github.com/user-attachments/assets/05418448-dba9-41d0-be54-29da490a796e)
위 그림은 동일 트랜잭션 내에 있는 멀티 스레드 환경에서 왜 동시성 문제가 발생한 것인지 대하여 추측하며 작성했던 그림입니다.   
스레드 1이 중복 태그가 있는지 확인합니다. 그 후 스레드 7번도 중복 태그가 있는지 확인합니다.   
스레드 1은 중복 태그가 없으므로 저장하고 커밋, 스레드 7도 동일하게 태그 저장을 하고 커밋을 합니다.   
스레드 7은 스레드 1이 커밋되지 않은 상태에서 데이터를 조회하기 때문에 DB에 데이터가 없어서 중복이 제대로 검증되지 않은 것입니다.   

그렇다면, 이를 방지하기 위해서 스레드 1의 작업이 모두 끝나고 스레드 7의 작업이 진행되도록 해야 합니다.   
이 방식대로 하기 위해 synchronized를 사용하게 되었습니다.   

synchronized는 기본적으로 멀티 스레드 환경에서 여러 스레드가 하나의 공유 자원에 동시에 접근하지 못하도록 해줍니다.   
그렇기 때문에 동시성 문제를 해결할 수 있을 것이라 생각했습니다.   

하지만, @Transactional과 synchronized를 함께 사용하여 동시성 문제가 해결되지 않았던 것을 봤습니다.   
본격적으로 왜 이런 문제가 발생했는지 알아보기 위해 트랜잭션 로그를 찍어서 확인해보았습니다.   
```java
// --------- 트랜잭션은 쓰레드 1과 2가 동시에 시작 --------------
01:42:49.759 [thread-2] Exposing JPA transaction as JDBC
01:42:49.759 [thread-1] Exposing JPA transaction as JDBC

// synchronized 로 인해 쓰레드 1은 대기
Hibernate: select l1_0.id,l1_0.description,l1_0.image_url,l1_0.invalidated_at_at,l1_0.title,l1_0.url from link l1_0 where l1_0.url=?
01:42:49.889 [thread-2] Found thread-bound EntityManager for JPA transaction
01:42:49.889 [thread-2] Participating in existing transaction

Hibernate: insert into link (description,image_url,invalidated_at_at,title,url) values (?,?,?,?,?)
01:42:49.924 [thread-2] Initiating transaction commit

// ****** 커밋 처리를 시작한다는 로그 출력
01:42:49.925 [thread-2] Committing JPA transaction on EntityManager
// ****** 0.003초 뒤에 바로 synchronized 로 멈춰 있던 쓰레드 1이 시작
Hibernate: select l1_0.id,l1_0.description,l1_0.image_url,l1_0.invalidated_at_at,l1_0.title,l1_0.url from link l1_0 where l1_0.url=?
01:42:49.928 [thread-1] Found thread-bound EntityManager for JPA transaction
01:42:49.928 [thread-1] Participating in existing transaction

Hibernate: insert into link (description,image_url,invalidated_at_at,title,url) values (?,?,?,?,?)
01:42:49.934 [thread-1] Initiating transaction commit

// 커밋 처리를 시작한다는 로그
01:42:49.934 [thread-1] Committing JPA transaction on EntityManager
```
트랜잭션이 시작 될 때 스레드 1, 2가 동시에 시작되었습니다.   
그 후 synchronized로 인해 스레드 1이 대기 상태로 들어가게 되었습니다.   
스레드 2는 트랜잭션에 참여하고, insert문을 실행하였습니다.   
그 후 commit을 하는 과정에서 스레드 1이 트랜잭션에 참여하게 되었습니다.   

이 과정에서 문제가 발생하게 되는 것입니다.   
커밋이 완료된 이후에 다른 스레드가 트랜잭션에 참여해야 합니다.  
스레드 2가 커밋이 되는 과정에서 아직 DB에 데이터가 반영되지 않은 상태에 스레드 1이 참여하게 되었습니다.   
스레드 1은 DB에 데이터가 없는 것으로 판단하여 중복되는 태그가 없다고 판단하여 스레드 1, 2에 태그 생성이 모두 성공하게 되는 것입니다.   

정확히는 트랜잭션 범위 내에 synchronized가 위치하게 되어 동시성 제어가 제대로 이루어지지 않는다고 이해하면 될 것 같습니다.   
synchronized 범위만 있다면, 하나의 스레드가 끝날 때까지 다른 스레드는 대기하게 할 수 있습니다.   

위에서 로그를 보며 살펴봤기 때문에 synchronized만 사용하면 될 것으로 예상이 됩니다.   
그러면 synchronized만 사용해서 동시성 제어를 해보도록 하겠습니다.   
```java
public TagResult saveTag(TagCommand.Create command) {
    validateDuplicateTagName(command.userId(), command.name());
    return tagMapper.toResult(tagDataHandler.saveTag(command.userId(), command));
}
```
![image3](https://github.com/user-attachments/assets/353787f0-ea1b-4fdd-882e-73120dcfbf50)
synchronized만 사용하여 동시성 제어가 제대로 된 것을 확인 할 수 있습니다. 태그가 1개만 생성되었습니다.   
하지만, synchronized 사용하는 것을 추천하지 않는데요. 그 이유에 대해서 다음 장에서 더 자세히 살펴보도록 하겠습니다.
- - -
## synchronized 문제점
1. 하나의 프로세스 안에서만 보장
2. 서버가 1대일 때는 괜찮지만, 여러 대일 경우 보장해주지 못함.
3. 각 프로세스 안에서만 보장되며, 각 프로세스에서 여러 스레드가 동시에 접근할 경우 Race Condition
4. 실제 운영 중인 서비스의 경우 대부분 2대 이상의 서버를 사용함.

저희 프로젝트에서는 위와 같은 문제들 때문에 DB unique constraint를 사용하여 해결하였습니다.   
물론 락을 이용한 방법도 있지만 완벽하게 DB에서 막는 것이 좋다고 생각하여 unique를 걸게 되었습니다.   

- - -
## Unique Constraint
Table에 unique constraint를 추가하였습니다.   
user_id과 name 두 컬럼을 묶어 하나의 인덱스로 걸었습니다.   
```java
@Table(
  name = "tag",
  uniqueConstraints = {
    @UniqueConstraint(
      name = "UC_TAG_NAME_PER_USER",
      columnNames = {"user_id", "name"}
    )
  }
)
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Tag {
```
unique index를 사용했을 때 장점, 단점에 대해 알아보겠습니다.   
- 장점 : 사용자별 태그 이름 중복 방지가 가능하다.
- 단점 : DB와 관련된 예외 처리 비용이 들 것으로 예상된다. (DataIntegrityViolationException)

현재 synchronized, unique constraint를 알아보았습니다.   
또 다른 동시성 제어 방법에 대하여 간략하게 설명하고 글을 마무리 짓도록 하겠습니다.   

## 다른 동시성 문제 해결 방법
### 1. 낙관적 락
낙관적 락의 경우 실제 DB에 락을 거는 것이 아닌 버전을 이용하여 정합성을 맞추는 방법입니다.   
먼저 데이터를 읽은 후 update를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하여 업데이트를 하는 방식입니다.
### 2. 비관적 락
비관적 락의 경우 실제 DB 데이터에 락을 걸어서 정합성을 맞추는 방식입니다.   
주로 select for update문을 이용하여 락을 걸게 됩니다.   
다른 트랜잭션에서는 Lock이 해제되기 전까지 데이터를 가져갈 수 없고, 데드락이 걸릴 수 있어서 사용하는데 주의해야 합니다.   
기본적으로 타임 아웃 설정이 되어 있지 않기 때문에 타임 아웃 설정을 하지 않으면 무한 대기 상태인 데드락에 걸리게 됩니다.   
비관적 락을 사용할 때는 꼭 타임 아웃을 설정해주어야 합니다.
### 3. 네임드 락
네임드 락의 경우 이름을 가진 metadata lock입니다.   
이름(문자열)을 키로 가지고, 해당 키를 가지고 락을 거는 방식입니다.   
즉 문자열을 가지고 락을 거는 방식이라고 생각하시면 됩니다.   
이름을 가진 lock을 획득한 후 해제될 때까지 다른 트랜잭션은 이 락을 획득할 수 없습니다.   
주의 할 점으로는 트랜잭션이 종료될 때 락이 자동으로 해제되지 않아서 직접 수동으로 락을 해제해주어야만 합니다.   
락을 제대로 해제해주지 못하면, 큰 문제가 발생하기 때문에 주의해서 사용해야 합니다.   
### 4. Redis
Redis는 기본적으로 싱글 스레드 방식이기 때문에 동시성 문제를 쉽게 제어할 수 있습니다.   
내부에서 여러 개념들이 있는데, 그러한 개념들을 활용해서 동시성 문제를 더 쉽게 제어할 수 있는 것으로 알고 있습니다.   
실무에서 동시성 제어하는데 주로 사용하는 방식이고, 서버가 여러 대일 때 많이 사용한다고 합니다.   
Redis를 이용하여 분산 락을 구현하여 동시성 문제를 해결하는 것으로 알고 있습니다.    
더 자세한 내용은 찾아보시면 좋을 것 같습니다.    

## 정리
1. synchronized와 @Transactional은 같이 사용하면 안된다.
2. synchronized보다 다른 방식으로 동시성 문제를 해결해야 한다.
3. Lock 또는 DB constraint를 고려해봐야 한다.
4. 서버가 여러 대인 경우 Redis 사용을 고려해보는 것이 좋다.

실제 프로젝트를 하며 겪었던 내용에 대해서 주로 작성하였습니다.   
Lock에 대해서 깊게 다루지 않는 글이기 때문에 궁금하신 분은 이러한 개념들이 있구나하고 찾아봐주시면 감사하겠습니다.   
긴 글 읽어주셔서 감사합니다...!
