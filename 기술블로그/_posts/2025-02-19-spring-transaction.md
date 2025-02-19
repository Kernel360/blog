---
layout: post  
title: "Spring Transaction"
author: "윤지윤"
categories: "기술블로그"
banner:
image: 
background: "#000"
height: "100vh"
min_height: "38vh"
heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [`transaction`, `@Transactional`, `propagation`]
---

# Spring Transaction

## `@Transactional(propagation = Propagation.REQUIRES_NEW)`를 써보며

이번에 Spring에서 제공하는 `@Transactional`로 트랜잭션을 선언하고, 분리도 해보았다. 의도대로 동작하지 않는 바람에 오히려 많은 것을 배울 수 있었다. 이 글은 아래와 같은 흐름으로 진행된다.

- 트랜잭션이란
- `@Transactional`이란
- `@Transactional`의 옵션 `propagation`이란
- 오잉 왜 DB가 비어있지? (적용과 문제 해결 과정)
- 번외: 다음부터는 비동기 처리

---

## 트랜잭션

트랜잭션은 ‘작업의 한 단위’라고 하는데, 쉽게 말해서 **DB 읽기와 쓰기 여러 개를 논리적으로 묶어놓은 것**을 말한다.  
**모두 반영하거나 혹은 모두 반영하지 않거나 두 결과만 있다.** (all or nothing)

트랜잭션 범위는 **커넥션(Connection) 기준**이다.  
JDBC API를 사용했다면 잘 알겠지만, 나처럼 `@Transactional`만 아무 생각 없이 사용했다면 모를 수 있다.  
아래는 앞으로도 계속해서 등장할 우리 **모니카 프로젝트 시동 OFF 로직**이다.  
JDBC API를 사용하는 것으로 바꿔봤다. 모든 메서드가 유일한 `Connection`을 파라미터로 전달받아 사용하고 있다.  
그리고 처음과 끝에는 **트랜잭션 시작과 끝을 명시**한다.

```java
public class VehicleController {

    private final DataSource dataSource;

    public BaseResponse keyOff(final KeyOffRequest request) {
        try (Connection conn = dataSource.getConnection()) {
            conn.setAutoCommit(false); // 트랜잭션 시작
            
            // ✅ 차량 정보 조회
            VehicleInformation vehicleInformation = getVehicleInformation(conn, ...);

            // ✅ 최근 차량 이벤트 조회
            Optional<VehicleEvent> vehicleEvent = getRecentVehicleEvent(conn, ...);
            boolean isAlreadyOff = vehicleEvent.map(VehicleEvent::isTypeOff).orElse(false);

            if (isAlreadyOff) {
                return BaseResponse.fail(ErrorCode.WRONG_APPROACH);
            }

            // ✅ 차량 상태 업데이트
            updateVehicleStatus(conn, ...);

            // ✅ 총 거리 업데이트 및 조회
            Long updatedTotalDistance = updateTotalDistance(conn, ...);

            // ✅ 주행 이력 저장
            saveDrivingHistory(conn, ...);

            // ✅ 차량 이벤트 저장
            saveVehicleEvent(conn, ...);

            // ✅ 알람 저장 및 전송
            saveAlarmIfNecessary(conn, ...)
                .ifPresent(alarmId -> sendAlarm(conn, ...));

            conn.commit(); // 트랜잭션 커밋
            return BaseResponse.success();

        } catch (Exception e) {
            return BaseResponse.fail(ErrorCode.INTERNAL_ERROR);
        }
    }

    // private 메서드들...
}
```
keyOff 시퀀스 다이어그램

<img src="https://github.com/user-attachments/assets/d0722d90-571c-433d-aed5-125f7144df62" width="800" height="700"/>

### @Transactional

다양한 방법이 있지만 우리가 가장 잘 아는 @Transactional은 Spring의 선언적 트랜잭션이다. 그냥 선언만 하면 된다! 
트랜잭셔널 어노테이션이 붙어있는 메서드를 실행하게 되면 이렇게 동작한다. 

`@Transactional ➡️ (AOP Proxy) ➡️ TransactionInterceptor ➡️ TransactionManager가 트랜잭션 시작 ➡️ 실제 비즈니스 로직 실행 ➡️ TransactionManager가 commit or rollback ➡️TransactionManager가 트랜잭션 종료` 

일반적으로(리액티브 환경이 아니라면)  `TransactionManager` 는 같은 스레드 내의 모든 데이터 접근 코드에 트랜잭션을 적용시킨다. 스레드를 분리하면 분리된 스레드 내의 데이터 접근 코드들에는 트랜잭션이 적용되지 않는다는 의미이기도 하다.

### Propagation

propagation은 트랜잭션 전파 방식을 의미하고, 개발자가 필요에 따라 @Transactional 의 propagation 옵션을 설정할 수 있다. 
앞서 말한 것처럼 트랜잭션은 메서드를 타고 타며 전파된다. 
기존 커넥션을 계속 유지하는 것이다. default 값인 REQUIRED인 경우 OFF 트랜잭션 흐름은 아래 그림과 같다.

<img src="https://github.com/user-attachments/assets/34623501-60cd-4ba0-8e73-e5c3f1de8d20" width="680" height="800"/>

### Propagation.REQUIRED_NEW

오늘 내가 소개 할 옵션 값은 REQUIRED_NEW이다. 이 방식은 기존 커넥션과 별도의 커넥션을 새로 생성해서 데이터에 접근하기 때문에 기존 커넥션에 영향을 주지 않는다. 반대로 기존 커넥션도 별도의 커넥션에 영향을 주지 않는다.

- 기존 트랜잭션을 잠시 중단(Suspend)
- 새로운 커넥션을 생성하고 새로운 트랜잭션을 시작
- 새로운 트랜잭션에서 해당 메서드 실행
- 실행이 끝나면 해당 트랜잭션을 커밋 또는 롤백
- 기존 트랜잭션을 다시 활성화(Resume)

REQUIRED_NEW의 경우 OFF 트랜잭션 흐름은 아래와 같다.

<img src="https://github.com/user-attachments/assets/7b8dd951-3f2a-46bd-bcec-e00c1a084121" width="680" height="800"/>

### 오잉 왜 DB가 비어있지?

이제 트랜잭션을 잘(?) 알게 되었으니 코드에도 적용해보기로 한다.

OFF 로직을 살펴보니, 차량 정보와 최근 이벤트를 조회해서 현재 상태를 확인하고 여러 테이블을 업데이트 하는 것은 하나의 작업 단위로 묶어야겠다는 생각이 든다. 차량 정보 테이블과 차량 이벤트(ON/OFF) 테이블, 운행 내역 테이블은 데이터 일관성이 요구되기 때문이다.

그런데 API를 호출하는 알림 전송은 다르다. 네크워크 I/O 특성상 실패 확률이 비교적 높기 때문에, 같은 트랜잭션으로 묶여있으면 독이 될거라고 판단했다. 나의 알림 전송은 우리 시스템 내 다른 API 서버에게 ‘주행거리 보니 지금쯤 점검할 때 됐다 ~ 사용자에게 알림 보내!’ 이기 때문에 안정성이 크게 요구되지 않는다. 
다음 OFF 때 재시도하면 그만이다. 이걸 실패했다고 해서 데이터 변경 쿼리들을 모두 롤백시키면, 중요한 데이터가 유실되어버린다. 그래서 keyOff 트랜잭션과 sendAlarm 트랜잭션읇 분리하기로 했다.

```java
@Transactional
public BaseResponse keyOff(
	@Valid @RequestBody final KeyOffRequest request
) {
    // ✅ 차량 정보 조회
    // ✅ 최근 차량 이벤트 조회
    // ✅ 차량 상태 업데이트
    // ✅ 총 거리 업데이트 및 조회
    // ✅ 주행 이력 저장
    // ✅ 차량 이벤트 저장

    // ✅ 알람 저장 및 전송
    Optional<Long> alarmId = alarmService.saveAlarmIfNecessary(vehicleInformation.getId(), updatedTotalDistance);
    alarmService.sendAlarm(alarmId);
    
    return BaseResponse.success();
}
```

```java
public Optional<Long> saveAlarmIfNecessary(Long vehicleId, Long totalDistance) {
    Optional<Alarm> alarm = alarmRepository.findRecentOneByVehicleId(vehicleId);
    int targetDistance = alarm.map(Alarm::getDrivingDistance).orElse(0);
    
    if (checkBiggerThanIntervalDistance(totalDistance, targetDistance)) {
      if (alarm.isEmpty() || alarm.get().getStatus().equals(AlarmStatus.COMPLETED)) {
        return Optional.ofNullable(alarmRepository.save(vehicleId));
      }
    }
    return Optional.empty();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void sendAlarm(Optional<Long> alarmId) {
    alarmSender.sendAlarm(AlarmSend.builder().alarmId(alarmId).build());
}
```

그러나 문제 1에 직면 ! ! sendAlarm이 자꾸 실패한다. 
알고보니, sendAlarm 트랜잭션이 진행 될 동안 saveAlarmIfNecessary에 전파된 기존 keyOff 트랜잭션이 커밋되지 않고 잠시 멈춰있었기 때문에 알림 요청을 받은 다른 API 서버가 DB에서 해당 alarm 데이터를 못찾는 것이었다.

saveAlarmIfNecessary의 트랜잭션이 끝나야(커밋되어야) 다른 API 서버가 데이터를 찾을 수 있기 때문에  saveAlarmIfNecessary에 flush를 통해 DB에 직접 반영하는 등의 처리가 필요하다. 근데 우리 뭘 알고있지? 
트랜잭션을 새로 분리하면 해당 메서드가 끝날 때 분리된 트랜잭션이 커밋되거나 롤백된다는 것을 알고있다. 
saveAlarmIfNecessary에도 @Transactional(propagation = Propagation.REQUIRES_NEW)을 적용하기로 한다.

```java
⬇️ 추가!
@Transactional(propagation = Propagation.REQUIRES_NEW)
public Optional<Long> saveAlarmIfNecessary(Long vehicleId, Long totalDistance) {
    if (checkBiggerThanIntervalDistance(totalDistance, targetDistance)) {
      if (. . .) {
        return Optional.ofNullable(alarmRepository.save(vehicleId));
      }
    }
    return Optional.empty();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void sendAlarm(Optional<Long> alarmId) {
    alarmSender.sendAlarm(AlarmSend.builder().alarmId(alarmId).build());
}
```

그러나 문제 2에 직면. . saveAlarmIfNecessary에서 alarm 데이터 조회시 SQLException이 발생했는데(이것은 휴먼 에러) 기존 keyOff 트랜잭션까지 롤백되어버린거다! 난 트랜잭션을 분리했는데 왜?

### @Transactional의 롤백

Spring에서 롤백을 트리거하는 방법 중 권장되는 것은 예외를 던지는 것이다. 
공식 문서 ‘The recommended way to indicate to the Spring Framework’s transaction infrastructure that a transaction’s work is to be rolled back is to throw an `Exception` from code that is currently executing in the context of a transaction.’ 참고. 
트랜잭션 매니저는 런타임 예외를 감지하고 해당 트랜잭션의 롤백을 결정한다.

saveAlarmIfNecessary에 SQLException을 잡는 곳이 없어서 keyOff 트랜잭션까지 **예외가 전파**됐고, keyOff 트랜잭션이 ‘어? 예외! 너 나가’ 한 것이었다. 
그래서 saveAlarmIfNecessary와 sendAlarm 트랜잭션에서 각자의 예외를 처리하도록 코드를 수정했다!

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public Optional<Long> saveAlarmIfNecessary(Long vehicleId, Long totalDistance) {
    try {
        if (checkBiggerThanIntervalDistance(totalDistance, targetDistance)) {
          if (. . .) {
            return Optional.ofNullable(alarmRepository.save(vehicleId));
          }
        }
        return Optional.empty();
    } catch (Exception e) { ⬅️추가!
        log.error("Alarm 쿼리 예외 발생", e);
    }
    return Optional.empty();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void sendAlarm(Long alarmId) {
    try {
      alarmSender.sendAlarm(AlarmSend.builder().alarmId(alarmId).build());
    } catch (Exception e) { ⬅️추가!
      log.info("something wrong: {}", e.getMessage());
    }
}
```
### 번외: 더 잘할 수 있을 것 같은데

알림 전송 로직을 위해 스레드를 분리하면 더 잘 처리할 수 있을 것 같다. 사실 우리가 잘 쓰는 많은 서비스들은 알림 전송을 비동기 처리하기도 한다. 알림 전송은 마지막에 결국 네트워크 I/O가 발생하는데, 네트워크를 절대 신뢰하지 마라는 말이 있듯 리스크가 큰 작업이기 때문이다.

1. 서버를 떠나. . 산넘고. . 바다 건너. . 저쪽 서버 / 사용자에게 잘 도착하리라는 보장이 없음
2. 해당 트랜잭션 대기 시간이 증가할 수 있음. (요청이 많다면 더더욱)
3. 잠시 멈춘 상태인 기존 트랜잭션의 처리 또한 지연됨.

이걸 감내할만큼 중요도가 높은 알림이라면 동기적으로 처리 해야지. 하지만 앱 푸시 알림처럼 나의 알림 또한 그 중요도가 낮기 때문에 비동기 처리하면 좋겠다. 커넥션이 분리되고, 스레드까지 분리되면 keyOff 전체 처리 속도가 단축될 것이라고 판단된다.

자바의 ApplicationEventPublisher을 이용해서 서버 내부에서 로직간의 결합도를 낮출 수 있으며, 혹은 외부 이벤트 브로커를 활용할 수도 있다. 지금은 주행거리 기준 점검 알림만 있지만, 에어컨 필터 등의 점검 기준이 더 늘어나면 메세지 큐로 내부 API 서버에 도달하는 수많은 알림 요청을 효과적으로 조절할 수 있다.

간단하게 ApplicationEventPublisher를 적용한 코드다

```java
private final ApplicationEventPublisher eventPublisher; ⬅️ 추가!

@Transactional(propagation = Propagation.REQUIRES_NEW)
public Optional<Long> saveAlarmIfNecessary(Long vehicleId, Long totalDistance) {
    try {
        if (checkBiggerThanIntervalDistance(totalDistance, targetDistance)) {
            if (. . .) {
                Long alarmId = alarmRepository.save(vehicleId);
                eventPublisher.publishEvent(new AlarmCreatedEvent(alarmId)); ⬅️ 추가!
                return Optional.of(alarmId);
            }
        }
        return Optional.empty();
    } catch (Exception e) {
        log.error("Alarm 쿼리 예외 발생", e);
    }
    return Optional.empty();
}
```

```java
@Async
@EventListener
public void handleAlarmEvent(AlarmCreatedEvent event) {
    alarmService.sendAlarm(event.getAlarmId());
}
```
sendAlarm을 비동기로 뺐을 때 트랜잭션 다이어그램
<img src="https://github.com/user-attachments/assets/33289b42-37cd-4399-9251-c236d12ebdae" width="1000" height="800"/>


