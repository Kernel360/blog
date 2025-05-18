---
layout: post
title: "Observer 패턴을 활용한 알림 시스템 리팩토링 경험기"
author: "박병찬"
categories: "기술블로그"
banner:
  image: "assets/images/observer-pattern-banner.jpg"
  background: "#333"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Java", "Spring", "Design Patterns", "Observer"]
---

# Observer 패턴을 활용한 알림 시스템 리팩토링 경험기

## 목차

1. [서론: 기존 알림 시스템과 문제점](#서론-기존-알림-시스템과-문제점)
2. [Observer 패턴이란?](#observer-패턴이란)
3. [기존 코드: 알림 시스템 구조](#기존-코드-알림-시스템-구조)
4. [리팩토링 후 변화](#리팩토링-후-변화)
5. [결론: 리팩토링의 효과](#결론-리팩토링의-효과)

---

## 서론: 기존 알림 시스템과 문제점

이번 글에서는 기존의 알림 시스템을 어떻게 **Observer 패턴**을 활용하여 리팩토링했는지 그 과정을 설명하려 합니다. 기존 시스템에서의 문제점을 해결하고, 유지보수성을 높이기 위한 리팩토링 경험을 다룹니다.

기존 시스템은 알림을 여러 방식(이메일, 실시간 알림 등)으로 처리하는 로직이 따로 구현되어 있었습니다. 이로 인해 **중복 코드**, **의존성 증가**, **확장성 부족** 등 여러 문제가 발생했죠.

사실, 저는 **Observer 패턴**이라는 디자인 패턴을 **디렉터님 특강**을 통해 처음 알게 되었습니다.  
그 후 이 패턴에 대해 궁금해져서 좀 더 찾아보고 공부하던 중,  
알림 시스템과 같은 **이벤트 기반 시스템**에 매우 적합하다는 것을 알게 되었습니다.  
이 패턴을 알림 기능에 적용하면, 코드가 더 **유연하고**, **확장성이 뛰어나며**, **유지보수하기 쉬운** 구조로 바뀔 수 있다는 생각이 들었습니다.  
이후, 실제로 제 프로젝트에 이 패턴을 적용해 보았고, 그 결과를 여러분과 공유하려 합니다.

기존의 알림 시스템은 이메일, 실시간 알림 등 여러 종류의 알림 방식에 대해 각각의 로직을 따로 구현했기 때문에 코드가 많이 중복되고, 알림 방식을 추가할 때마다 다른 부분에 영향을 미쳤습니다. 이는 코드 유지보수에 큰 어려움을 주었고, 시스템 확장에 제약을 주는 요소였습니다.

그렇다면 **Observer 패턴**을 적용하면 기존의 문제들을 어떻게 해결할 수 있는지, 그리고 실제 리팩토링을 통해 어떤 변화를 가져왔는지 함께 살펴보겠습니다.

### 기존 코드

기존의 알림 시스템은 다음과 같은 방식으로 구성되어 있었습니다:

#### `InquiryService`

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class InquiryServiceImpl implements InquiryService {
    private final CustomerExecutor customerExecutor;
    private final InquiryTemplateReader inquiryTemplateReader;
    private final InquiryExecutor inquiryExecutor;
    private final InquiryReader inquiryReader;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    @Override
    public CreateInquiryResDto createInquiry(CreateInquiryReqDto reqDto) {
        // templateToken 으로 InquiryTemplateShareToken 엔티티 조회
        InquiryTemplate template = inquiryTemplateReader.findByToken(reqDto.getTemplateToken());
        // 문의 남긴 고객이 DB에 존재하는지 확인
        Customer customer = customerExecutor.findOrCreateCustomer(reqDto.toCustomerReqDto(), template.getAgent());
        CreateInquiryCommand command = CreateInquiryCommand.builder()
            .template(template)
            .customer(customer)
            .answers(reqDto.getAnswers())
            .build();
        Inquiry inquiry = inquiryExecutor.executeInquiryCreation(command);

        // 알림 이벤트 발행
        eventPublisher.publishEvent(
            InquiryCreatedEvent.builder()
                .receiverId(template.getAgent().getId())
                .inquiryId(inquiry.getId())
                .content("새로운 문의가 도착했습니다.")
                .build()
        );

        return CreateInquiryResDto.builder()
            .inquiryId(inquiry.getId())
            .build();
    }
}
```

#### 이메일 서비스와 실시간 알림 서비스

이전 알림 시스템에서는 이메일 알림과 실시간 알림을 각각 `EmailService`와 `SseEmitterService`에서 처리했습니다. 이로 인해 **중복된 알림 로직**, **유지보수 어려움** 등이 발생했습니다.

```java
// EmailService
@Service
@RequiredArgsConstructor
public class EmailService {
	private final JavaMailSender mailSender;

	@Value("${spring.mail.username}")
	private String fromEmail;

	public void send(Notification notification) {
		SimpleMailMessage message = new SimpleMailMessage();
		message.setFrom(fromEmail);
		message.setTo(notification.getReceiver().getEmail());
		message.setSubject("새로운 알림");
		message.setText(notification.getContent());

		try {
			mailSender.send(message);
		} catch (MailException e) {
			throw new BusinessException(ErrorCode.EMAIL_SEND_FAILED);
		}
	}
}

// SseEmitterService
@Slf4j
@Service
@RequiredArgsConstructor
public class SseEmitterService {
	private final EmitterRepository emitterRepository;
	private final Map<Long, Queue<Notification>> notificationQueue = new ConcurrentHashMap<>();

	private static final Long DEFAULT_TIMEOUT = 60L * 1000 * 60; // 1시간

	public void send(Notification notification) {
		Long receiverId = notification.getReceiver().getId();
		List<SseEmitter> emitters = emitterRepository.findAll(receiverId);

		if (!emitters.isEmpty()) {
			List<SseEmitter> deadEmitters = new LinkedList<>();

			for (SseEmitter emitter : emitters) {
				try {
					emitter.send(NotificationDto.from(notification));
				} catch (IOException e) {
					String message = e.getMessage();
					if (message != null && message.contains("Broken pipe")) {
						log.warn("Broken pipe detected. Removing emitter: receiverId={}", receiverId);
					} else {
						log.warn("Emitter send failed: receiverId={}, reason={}", receiverId, e.getMessage());
					}
					emitter.completeWithError(e);
					deadEmitters.add(emitter);
				}
			}

			for (SseEmitter dead : deadEmitters) {
				emitterRepository.delete(receiverId, dead);
			}
		} else {
			enqueueNotification(receiverId, notification);
		}
	}
}
```

## Observer 패턴이란?

**Observer 패턴**은 한 객체의 상태 변화가 있을 때, 그 객체에 의존하는 다른 객체들에게 자동으로 통지하는 디자인 패턴입니다. 이 패턴은 주로 **이벤트 기반 시스템**, **알림 시스템**에서 유용하게 사용됩니다. **Subject(주체)**와 **Observer(관찰자)**라는 두 가지 주요 컴포넌트로 구성됩니다.

- **Subject**: 상태 변화가 발생하는 객체
- **Observer**: 상태 변화를 감지하고 반응하는 객체

## 리팩토링 후 변화

### 1. 이벤트 기반으로 알림 처리

Observer 패턴을 적용하여 **이벤트**를 발행하고 이를 처리하는 방식으로 시스템을 리팩토링했습니다. 이제 알림 시스템은 **Observer(알림 서비스)**가 **Subject(이벤트)**를 구독하여 알림을 처리하는 구조로 변경되었습니다.

### 2. 코드 리팩토링

```java
// InquiryCreatedEvent
@Getter
@AllArgsConstructor
@Builder
public class InquiryCreatedEvent {
    private final Long receiverId;
    private final Long inquiryId;
    private final String content;
}

// NotificationListener
@Component
@RequiredArgsConstructor
public class InquiryNotificationListener {
    private final NotificationService notificationService;

    @EventListener
    public void handleInquiryCreatedEvent(InquiryCreatedEvent event) {
        notificationService.sendInquiryNotification(event);
    }
}
```

이렇게 `InquiryCreatedEvent`를 발행하고 `NotificationListener`에서 이를 수신하여 알림을 처리하는 구조로 변경하였습니다. 이제 `NotificationService`는 여러 알림 처리 로직을 책임지며, **이메일 알림**과 **실시간 알림**을 동시에 처리할 수 있습니다.

```java
// NotificationService
@Slf4j
@Service
@RequiredArgsConstructor
public class NotificationService {
    private final NotificationStore notificationStore;
    private final NotificationReader notificationReader;
    private final NotificationExecutor notificationExecutor;

    public void sendInquiryNotification(InquiryCreatedEvent event) {
        Agent receiver = notificationReader.findReceiverById(event.getReceiverId());

        Notification notification = Notification.builder()
            .receiver(receiver)
            .url("/inquiries/" + event.getInquiryId() + "/answers")
            .content(event.getContent())
            .type(NotificationType.INQUIRY_CREATED)
            .isRead(false)
            .build();

        Notification saved = notificationStore.create(notification);

        notificationExecutor.send(saved);
    }
}
```

### 3. Observer 패턴 적용

리팩토링 후 `NotificationExecutor`는 **Observer** 역할을 하며, 이메일 알림과 실시간 알

림을 담당하는 **Observer**들이 **Event**를 통해 트리거됩니다.

## 🧹 결론: 리팩토링의 효과

기존 알림 시스템은 각 기능에서 직접 알림을 보내도록 구성되어 있어, 알림 방식을 추가하거나 수정할 때마다 여러 부분을 함께 수정해야 했습니다. 그만큼 실수나 누락이 발생할 가능성도 높았고, 이메일과 실시간 알림 로직도 중복된 형태로 작성되어 있어 유지보수가 쉽지 않았습니다.

이번에 Observer 패턴 기반으로 알림 구조를 리팩토링하면서 다음과 같은 변화가 있었습니다:

### ✅ 1. 알림 방식 확장이 쉬워졌습니다

이제는 새로운 전송 로직만 구현하고 Observer로 등록하면 됩니다.  
예를 들어 **문자 발송(SMS)** 기능이 추가될 경우, 관련 클래스를 정의하고 구독만 연결하면 자연스럽게 동작하게 됩니다.

### ✅ 2. 중복 코드가 줄어들었습니다

공통 알림 처리 흐름을 한 곳에서 관리하게 되면서, 각 기능마다 유사한 코드를 반복 작성할 필요가 없어졌습니다.  
덕분에 유지보수도 쉬워졌고, 코드의 의도도 더 명확하게 드러납니다.

### ✅ 3. 다양한 이벤트에 대한 알림 발행이 쉬워졌습니다

초기에는 `InquiryCreatedEvent`에만 알림을 붙였지만, 이후 다른 이벤트에 대해서도 쉽게 확장할 수 있게 되었습니다.  
예를 들어:

- 🎂 생일 대상 고객 문자 발송 이벤트 (`BirthdayCustomerNotifiedEvent`)
- 🏠 계약 만료 대상 매물 집주인 문자 발송 이벤트 (`LeaseExpirationAlertEvent`)

이처럼 다양한 이벤트에 대한 알림도 동일한 방식으로 처리할 수 있어, 시스템 전체의 확장성과 일관성이 좋아졌습니다.

### ✅ 4. 책임이 명확히 분리되었습니다

알림 처리 책임이 도메인 로직과 분리되어 구조가 더욱 명확해졌습니다.  
읽는 사람 입장에서 각 클래스의 역할이 분명히 드러나, 가독성과 유지보수성 모두 향상되었습니다.

### 💡 정리하자면...

이번 리팩토링은 단순한 구조 개선을 넘어, **향후 알림 기능 확장에 들어가는 비용과 리스크를 줄이고 전체 시스템의 유연성을 높이는 데 큰 도움이 되었습니다.**
