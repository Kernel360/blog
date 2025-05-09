---
layout: post  
title: "스프링 이벤트를 활용한 비동기 처리 방법 (ex. 프로젝트 생성 횟수 통계)"
author: "정서연"
categories: "기술블로그"
banner:
  image: 썸네일로 넣고 싶은 이미지 링크
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`Spring`, `event`, `Java`, `Async`]
---

# SODA 프로젝트: 스프링 이벤트와 비동기 처리로 프로젝트 생성 통계 구현하기

SODA 프로젝트를 진행하면서, 사용자들이 생성한 프로젝트의 추이를 그래프로 보여주는 기능을 위한 API 개발이 필요했습니다. 이 기능을 구현하기 위해 두 가지 접근 방식을 고민했습니다.

1.  **실시간 DB 전체 조회:** 요청 시마다 실제 프로젝트 DB를 전체 조회하여 생성일 기준으로 카운트.
2.  **스프링 배치 활용:** 일괄 처리를 통해 통계용 테이블을 별도로 생성하고, 하루 한 번씩 카운트 저장.

첫 번째 방법은 데이터가 많아질수록 심각한 성능 저하를 유발할 것이 뻔했기에 고려 대상에서 제외했습니다. 처음에는 두 번째 방법, 즉 스프링 배치를 활용하는 것이 적합하다고 판단했습니다. 프로젝트 규모가 커지면 하루에 생성되는 프로젝트 수가 많아질 것이고, 매번 통계 테이블에 접근하는 것보다 정해진 시간에 한 번에 처리하는 것이 효율적이라고 생각했기 때문입니다.

주말 동안 스프링 배치를 적용하고 테스트하며 정상 작동을 확인했지만, 팀 회의를 통해 생각이 바뀌었습니다.

> "아무리 규모가 커져도 프로젝트가 하루에 수백, 수천 개씩 생성될까? 현재 예상 규모에서 스프링 배치는 오버 엔지니어링이 아닐까?"

스프링 배치를 적용할 만한 명확한 이유를 찾기 어려웠고, 다른 대안을 모색한 결과 **스프링 이벤트(Spring Event)를 활용한 비동기 처리** 방식을 적용하기로 결정했습니다.

## 비동기 처리를 선택한 이유

프로젝트 생성 통계를 업데이트하는 로직은 다음과 같습니다.

1.  프로젝트 생성 API 요청.
2.  프로젝트 정보 DB 저장 (핵심 로직).
3.  프로젝트 생성 성공 이벤트 발행.
4.  (별도 스레드) 이벤트 리스너가 통계 테이블에서 해당 날짜 데이터 조회.
5.  (별도 스레드) 해당 날짜 데이터가 없으면 새로 생성 후 `count + 1`, 있으면 기존 `count + 1`.

이렇게 구성하면, 프로젝트 생성 API는 이벤트만 발행하고 즉시 사용자에게 응답을 반환할 수 있습니다. 통계 업데이트는 백그라운드에서 비동기적으로 처리되므로, API의 응답 속도에 전혀 영향을 주지 않을 거라고 판단했습니다.

## 스프링 이벤트의 기본 구조와 작동 원리

스프링 이벤트는 옵저버 패턴(Observer Pattern)의 구현체로, 특정 동작(이벤트)이 발생했을 때, 이벤트를 구독하고 있는 다른 객체(리스너)에게 이를 알리고 관련 작업을 수행하도록 하는 메커니즘입니다.

*   **`ApplicationEvent`**
    *   이벤트 자체를 나타내는 클래스
    *   개발자는 이를 상속받아 커스텀 이벤트를 정의합
*   **`ApplicationEventPublisher`**
    *   이벤트를 발행(publish)하는 역할을 하는 인터페이스
*   **`@EventListener`**
    *   특정 이벤트를 구독하고 처리할 메서드에 사용하는 어노테이션

이 구조는 이벤트의 발생과 처리를 명확하게 분리하여 코드의 가독성과 유지보수성을 향상시킵니다.

## 스프링 이벤트를 활용한 비동기 프로그래밍의 장점

기본적으로 스프링 이벤트는 **동기적(Synchronous)**으로 동작합니다. 즉, 이벤트를 발행한 스레드 내에서 이벤트 리스너의 로직까지 모두 처리된 후에야 다음 로직으로 진행됩니다. 이는 핵심 로직과 부가 로직(이벤트 처리)이 섞여 전체 프로세스 시간을 늘릴 수 있습니다.

하지만 이벤트 처리 과정을 `@Async` 어노테이션을 통해 **비동기적(Asynchronous)**으로 만들면 다음과 같은 장점을 얻을 수 있습니다.

1.  **성능 향상**: 이벤트 처리 로직이 주 애플리케이션 흐름을 차단하지 않고 별도의 스레드에서 실행되므로, API 응답 속도 등 주요 기능의 성능이 향상됩니다.
2.  **관심사의 분리 및 확장성**: 프로젝트 생성 로직과 통계 업데이트 로직이 분리됩니다. 나중에 "프로젝트 생성 시 알림 발송"과 같은 추가 기능이 필요하다면, 새로운 이벤트 리스너만 추가하면 되므로 시스템 확장이 용이합니다.

### 이벤트 발행과 스레드 동작 예시

1.  **프로젝트 생성**: 사용자가 프로젝트 생성을 요청합니다.
2.  **이벤트 발행**: 프로젝트가 성공적으로 생성되면, `ProjectCreatedEvent`가 발행됩니다.
3.  **비동기 처리**: `@Async`가 적용된 이벤트 리스너가 다른 스레드에서 활성화되어, 통계 테이블의 생성 횟수를 증가시키는 메서드를 실행합니다.
4.  **즉시 응답**: 메인 스레드는 통계 업데이트 완료를 기다리지 않고, 프로젝트 생성 성공 응답을 즉시 사용자에게 반환합니다.

## 실제 구현 코드 예제

### 1. 이벤트 클래스 생성 (`ProjectCreatedEvent`)

"프로젝트가 생성되었다"는 사실(이벤트)을 다른 컴포넌트(여기서는 `ProjectEventListener`)에 알리기 위한 객체입니다.

```java
import lombok.Getter;
import java.time.LocalDate;

@Getter
public class ProjectCreatedEvent {
    private final LocalDate creationDate;

    public ProjectCreatedEvent(LocalDate creationDate) {
        this.creationDate = creationDate;
    }
}
```

### 2. 이벤트 리스너 생성 (`ProjectEventListener`)

`ProjectCreatedEvent`를 비동기적으로 수신하고 처리하는 리스너입니다.

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.event.EventListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Slf4j
@Component
@RequiredArgsConstructor
public class ProjectEventListener {

    private final ProjectStatsUpdateService statsUpdateService;

    @EventListener
    @Async // 이 메서드를 비동기로 실행하도록 지정
    public void handleProjectCreatedEvent(ProjectCreatedEvent event) {
        log.info("ProjectCreatedEvent 수신: Date = {}", event.getCreationDate());
        try {
            statsUpdateService.incrementProjectCount(event.getCreationDate());
            log.info("비동기 통계 업데이트 로직 호출 완료: DATE = {}", event.getCreationDate());
        } catch (Exception e) {
            log.error("비동기 통계 업데이트 처리 중 최종 오류 발생: Date = {}, Error={}", event.getCreationDate(), e.getMessage(), e);
        }
    }
}
```
*   `@EventListener`: 이 메서드가 `ProjectCreatedEvent` 타입의 이벤트를 수신함을 스프링에 알립니다.
*   `@Async`: 이 메서드의 실행을 별도의 스레드 풀에서 비동기적으로 처리하도록 합니다. 따라서 프로젝트 생성 요청을 처리하는 주 스레드는 이 리스너의 작업 완료를 기다리지 않습니다.

### 3. 통계 업데이트 서비스 로직 (`ProjectStatsUpdateService`)

실제로 통계 데이터를 증가시키는 비즈니스 로직입니다.

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDate;

@Slf4j
@Service
@RequiredArgsConstructor
public class ProjectStatsUpdateService {
    private final ProjectDailyStatsRepository statsRepository; // JPA Repository

    @Transactional(propagation = Propagation.REQUIRES_NEW) // 새로운 트랜잭션에서 실행
    public void incrementProjectCount(LocalDate date) {
        log.info("통계 업데이트 시작: Date = {}", date);
        try {
            ProjectDailyStats stats = statsRepository.findByStatDate(date)
                    .orElseGet(() -> {
                        log.info("해당 날짜({})의 통계 없음. 새로 생성 시작", date);
                        return ProjectDailyStats.builder()
                                .statDate(date)
                                .creationCount(0L) // 초기값 0
                                .build();
                    });

            stats.incrementCount(); // 카운트 증가
            statsRepository.save(stats); // DB 저장
            log.info("통계 업데이트 완료 : Date={}, New Count={}", date, stats.getCreationCount());
        } catch (Exception e) {
            log.error("통계 업데이트 중 오류 발생: Date={}, Error={}", date, e.getMessage(), e);
            throw e;
        }
    }
}
```
*   `@Transactional(propagation = Propagation.REQUIRES_NEW)`
    *   새로운 트랜잭션에서 실행되도록 하는 메서드
    *   이벤트 발행 로직의 트랜잭션과 분리되어 통계 업데이트의 성공/실패가 주 로직에 영향을 주지 않도록 하기 위함
    *   동시성 문제나 DB 오류 발생 시 롤백 처리를 독립적으로 수행

### 4. 프로젝트 생성 로직에 이벤트 발행 추가 (`ProjectService`)

프로젝트가 성공적으로 생성된 후 이벤트를 발행합니다.

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDate;

@Slf4j
@Service
@RequiredArgsConstructor
public class ProjectService {
    // ... (기존 의존성 및 필드)
    private final ApplicationEventPublisher eventPublisher; // 이벤트 발행기

    @Transactional // 프로젝트 생성은 하나의 트랜잭션으로 처리
    public ProjectCreateResponse createProject(String userRole, ProjectCreateRequest request) {
        // ... (프로젝트 생성 및 저장 로직)
        Project project = new Project(/* ... */);
        projectRepository.save(project);
        // ... (기존 로직)

        // 이벤트 발행
        try {
            LocalDate creationDate = project.getCreatedAt().toLocalDate(); // 예시: project 엔티티에서 생성일자 가져오기
            eventPublisher.publishEvent(new ProjectCreatedEvent(creationDate));
            log.info("ProjectCreatedEvent 발행: Date={}", creationDate);
        } catch (Exception e) {
            // 이벤트 발행 실패는 주 로직 실패로 이어지지 않도록 처리 (로깅 또는 별도 처리)
            log.error("ProjectCreatedEvent 발행 실패: Project ID={}, Error={}", project.getId(), e.getMessage(), e);
        }

        log.info("프로젝트 생성 완료: 프로젝트 ID = {}", project.getId());
        return createProjectCreateResponse(project); // 응답 생성
    }

    private ProjectCreateResponse createProjectCreateResponse(Project project) {
        // ... 응답 객체 생성 로직
        return new ProjectCreateResponse(/* ... */);
    }
}
```
*   `ApplicationEventPublisher`: 주입받아 `publishEvent()` 메서드로 이벤트를 발행합니다.

### 5. 통계 조회 서비스 로직 (`ProjectStatsService` - 간략)

통계 데이터를 조회하는 서비스 로직은 기존과 유사하게 유지하되, `ProjectDailyStatsRepository`를 사용합니다.

```java
// ProjectStatsService.java (일부)
public ProjectStatsResponse getProjectCreationTrend(Long userId, String userRole, ProjectStatsCondition statsRequest) {
    // ... (권한 및 날짜 유효성 검사)

    // 집계 데이터 조회
    List<Tuple> statsData = projectDailyStatsRepository.findProjectCreationStats(
        statsRequest.getStartDate(), 
        statsRequest.getEndDate(), 
        statsRequest.getTimeUnit()
    );
    log.debug("프로젝트 생성 통계 데이터 조회 완료: {}개의 데이터", statsData.size());

    // 조회 결과를 Map으로 변환하고, 누락된 날짜는 0으로 채우는 로직 (generateFullTrendData)
    Map<String, Long> statsMap = // ...
    List<ProjectStatsResponse.DataPoint> trendData = generateFullTrendData(
        statsRequest.getStartDate(), 
        statsRequest.getEndDate(), 
        statsRequest.getTimeUnit(), 
        statsMap
    );
    
    return ProjectStatsResponse.from(statsRequest, trendData);
}
```
`generateFullTrendData` 메서드는 조회 기간 내 모든 날짜/주/월에 대해 데이터가 없는 경우 0으로 채워주는 역할을 합니다. 이는 그래프 시각화 시 데이터가 누락되지 않도록 하기 위함입니다.

### 6. 비동기 기능 활성화 (`@EnableAsync`)

스프링 부트 애플리케이션의 메인 클래스나 별도의 `@Configuration` 클래스에 `@EnableAsync` 어노테이션을 추가하여 비동기 처리를 활성화합니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync // 비동기 기능 활성화
public class SodaApplication {
    public static void main(String[] args) {
        SpringApplication.run(SodaApplication.class, args);
    }
}
```

## 자세한 코드 및 참고 자료

*   **실제 통계 관련 예시** 
    *   [\[Feature\] 프로젝트 생성 통계 구현 by seoyeon-jung · Pull Request #257 · Kernel360/KDEV4-SODA-BE](https://github.com/Kernel360/KDEV4-SODA-BE/pull/257)
*   **참고 자료:**
    *   [f-lab: 스프링 이벤트를 활용한 비동기 처리 방법](https://f-lab.kr/insight/spring-event-asynchronous-processing)
    *   [ \[Spring\] 스프링에서 이벤트의 발행과 구독 방법과 주의사항, 이벤트 사용의 장/단점과 사용 예시](https://mangkyu.tistory.com/292)
