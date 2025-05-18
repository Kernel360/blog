---
layout: post  
title: "springBean"
author: "차의진"
categories: "기술 블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["layeredArchitecture"]
---

# 스프링과 싱글톤 패턴

## 목차

1. [들어가며](#들어가며)
2. [싱글톤 패턴이란?](#싱글톤-패턴이란)
3. [스프링과 싱글톤](#스프링과-싱글톤)
4. [스프링이 싱글톤을 관리하는 방식](#스프링이-싱글톤을-관리하는-방식)
5. [싱글톤 패턴의 단점과 스프링의 해결책](#싱글톤-패턴의-단점과-스프링의-해결책)
6. [주의해야 할 점](#주의해야-할-점)
7. [마치며](#마치며)

## 1. 들어가며

스프링 프레임워크의 핵심 기능 중 하나는 애플리케이션 컨텍스트를 통한 빈(Bean) 관리입니다. 스프링은 기본적으로 모든 빈을 싱글톤(Singleton)으로 관리합니다. 이게 무슨 의미일까요? 그리고 왜 스프링은 빈을 싱글톤으로 관리할까요?

---

## 2. 싱글톤 패턴이란?

먼저 싱글톤 패턴이 무엇인지 알아보겠습니다. 싱글톤 패턴은 소프트웨어 디자인 패턴 중 하나로, **어떤 클래스의 인스턴스가 오직 하나만 생성되는 것을 보장**하는 패턴입니다. 즉, 애플리케이션이 시작될 때 해당 클래스의 인스턴스를 딱 하나만 만들고, 이후에는 그 인스턴스를 계속 재사용하는 것입니다.

### 전통적인 싱글톤 패턴 구현 예시

```java
public class ClassicSingleton {
    // 정적 필드에 인스턴스 보관
    private static ClassicSingleton instance;

    // private 생성자로 외부에서 인스턴스 생성 방지
    private ClassicSingleton() {}

    // 인스턴스 접근 메서드
    public static synchronized ClassicSingleton getInstance() {
        if (instance == null) {
            instance = new ClassicSingleton();
        }
        return instance;
    }

    // 비즈니스 로직 메서드
    public void doSomething() {
        System.out.println("싱글톤 객체 사용 중");
    }
}
```

이 패턴은 다음과 같은 특징을 가집니다:

- private 생성자로 외부에서 new로 인스턴스 생성을 막습니다.
- static 메서드를 통해 오직 하나의 인스턴스만 반환합니다.
- 처음 호출될 때만 인스턴스를 생성하고, 이후 호출에서는 생성된 인스턴스를 반환합니다.

---

## 3. 스프링과 싱글톤

### 스프링이 싱글톤을 사용하는 이유

스프링이 빈을 싱글톤으로 관리하는 이유는 무엇일까요? 가장 큰 이유는 **성능** 때문입니다.

1. **메모리 사용량 감소**: 애플리케이션에서 자주 사용되는 객체를 매번 새로 생성하지 않고 하나의 인스턴스를 재사용함으로써 메모리 사용량을 크게 줄일 수 있습니다.

2. **서버 부하 감소**: 웹 애플리케이션은 수많은 사용자의 요청을 처리해야 합니다. 모든 요청마다 새로운 객체를 생성한다면 서버에 큰 부하가 걸릴 수 있습니다.

3. **요청 처리 속도 향상**: 객체 생성과 GC(가비지 컬렉션)는 비용이 큰 작업입니다. 싱글톤 패턴을 사용하면 이러한 비용을 줄여 요청 처리 속도를 향상시킬 수 있습니다.

### 웹 애플리케이션과 싱글톤의 관계

웹 애플리케이션은 보통 다음과 같은 특성을 가집니다:

1. **다수의 사용자**: 수십, 수백, 수천 명의 사용자가 동시에 접속할 수 있습니다.
2. **다수의 요청**: 각 사용자는 여러 개의 HTTP 요청을 보낼 수 있습니다.

만약 사용자의 요청마다 애플리케이션 객체(서비스, 리포지토리 등)를 새로 생성한다면 어떻게 될까요?

```java
// 싱글톤을 사용하지 않는 경우
@RestController
public class UserController {
    // 요청마다 새 객체 생성
    private UserService userService = new UserService();

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}
```

이 경우, 사용자 요청이 100번 들어오면 UserService 객체도 100개가 생성됩니다. 만약 UserService가 내부적으로 다른 객체들을 생성한다면, 메모리 사용량은 기하급수적으로 증가할 수 있습니다.

---

## 4. 스프링이 싱글톤을 관리하는 방식

스프링은 빈을 싱글톤으로 관리하기 위해 **싱글톤 레지스트리(Singleton Registry)**라는 기능을 제공합니다. 이는 스프링 컨테이너가 싱글톤 객체를 생성하고, 관리하고, 제공하는 기능을 담당합니다.

### 스프링 빈의 등록과 사용

```java
// @Component 어노테이션으로 빈 등록
@Component
public class UserService {
    public User getUser(Long id) {
        // 사용자 조회 로직
        return new User(id, "User" + id);
    }
}

// 컨트롤러에서 빈 주입 받기
@RestController
public class UserController {
    // 싱글톤 빈 주입
    private final UserService userService;

    // 생성자 주입
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}
```

위 예제에서 UserService는 스프링 컨테이너에 의해 단 한 번만 생성되고, 여러 UserController 인스턴스에서 공유됩니다.

### 스프링과 전통적인 싱글톤 패턴의 차이

스프링의 싱글톤 관리 방식은 전통적인 싱글톤 패턴과 몇 가지 중요한 차이점이 있습니다:

1. **생성 방식**: 전통적인 싱글톤은 클래스 로더당 하나의 인스턴스를 보장하지만, 스프링은 컨테이너당 하나의 인스턴스를 보장합니다.

2. **의존성 관리**: 스프링은 의존성 주입(DI)을 통해 싱글톤 객체 간의 관계를 관리합니다.

3. **테스트 용이성**: 스프링의 싱글톤은 DI를 통해 쉽게 모의 객체(mock)로 대체할 수 있어 테스트가 용이합니다.

---

## 5. 싱글톤 패턴의 단점과 스프링의 해결책

전통적인 싱글톤 패턴에는 몇 가지 단점이 있습니다:

1. **private 생성자로 인한 상속 불가**: 싱글톤 클래스를 상속할 수 없습니다.
2. **테스트 어려움**: 싱글톤 인스턴스를 모의 객체로 대체하기 어렵습니다.
3. **상태 공유 위험**: 싱글톤 객체는 상태를 가질 경우 여러 스레드에서 동시에 접근하면 문제가 발생할 수 있습니다.
4. **의존성 숨김**: 싱글톤 패턴은 getInstance()를 통해 의존성을 숨기므로, 의존성 관계가 명확하지 않습니다.

스프링은 이러한 단점을 다음과 같이 해결합니다:

1. **상속 가능**: 스프링 빈은 일반 클래스로 작성되므로 상속이 가능합니다.
2. **테스트 용이성**: DI를 통해 쉽게 모의 객체로 대체할 수 있습니다.
3. **상태 관리**: 스프링은 프로토타입 스코프 등 다양한 빈 스코프를 제공하여 상태 공유 문제를 해결할 수 있습니다.
4. **명시적 의존성**: 생성자 주입 등을 통해 의존성을 명시적으로 선언합니다.

```java
// 상속과 DI가 가능한 스프링 빈
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    // 명시적 의존성 주입
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public User getUser(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}
```

---

## 6. 주의해야 할 점

스프링의 싱글톤 빈을 사용할 때 주의해야 할 점이 있습니다.

### 1. 상태 관리에 주의

싱글톤 빈은 여러 스레드에서 동시에 접근할 수 있으므로, 상태를 가지면 안 됩니다(무상태, Stateless).

```java
// 잘못된 예: 싱글톤 빈이 상태를 가짐
@Service
public class BadUserService {
    // 인스턴스 변수(상태)를 가짐 - 위험!
    private User currentUser;

    public void setCurrentUser(User user) {
        this.currentUser = user; // 여러 스레드에서 동시 접근 시 문제 발생
    }

    public User getCurrentUser() {
        return currentUser;
    }
}

// 올바른 예: 상태를 메서드 파라미터나 지역 변수로 관리
@Service
public class GoodUserService {
    private final UserRepository userRepository;

    public GoodUserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUser(Long id) {
        // 메서드 파라미터로 상태 관리
        return userRepository.findById(id).orElseThrow();
    }

    public void processUser(Long id) {
        // 지역 변수로 상태 관리
        User user = userRepository.findById(id).orElseThrow();
        // 처리 로직
    }
}
```

### 2. 초기화 순서에 주의

스프링은 빈 간의 의존 관계에 따라 초기화 순서를 결정합니다. 순환 참조가 발생하지 않도록 설계해야 합니다.

```java
// 순환 참조 문제 예시
@Service
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Service
public class ServiceB {
    private final ServiceA serviceA; // 순환 참조 발생!

    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}
```

### 3. 프록시 패턴 이해하기

스프링은 AOP(관점 지향 프로그래밍)를 위해 빈을 프록시로 감쌀 수 있습니다. 이 프록시의 동작을 이해하는 것이 중요합니다.

```java
@Service
@Transactional
public class UserService {
    // 이 메서드는 프록시에 의해 트랜잭션으로 감싸짐
    public void createUser(User user) {
        // 사용자 생성 로직
    }

    // 주의: 같은 클래스 내의 메서드 호출은 프록시를 거치지 않음
    public void registerUser(User user) {
        // 이 호출은 트랜잭션 없이 직접 호출됨!
        createUser(user);
    }
}
```

---

## 마치며

스프링 프레임워크가 빈을 싱글톤으로 관리하는 이유는 웹 애플리케이션의 성능과 리소스 관리 효율성 때문입니다. 싱글톤 패턴의 단점을 보완하면서도 장점은 그대로 살리는 스프링의 접근 방식은 대규모 엔터프라이즈 애플리케이션 개발에 적합합니다.

하지만 싱글톤 빈을 사용할 때는 상태 관리에 주의해야 하며, 필요에 따라 다른 스코프의 빈을 활용할 줄 알아야 합니다. 스프링의 DI와 싱글톤 레지스트리를 적절히 활용하면 유지보수하기 쉽고 확장성 있는 애플리케이션을 개발할 수 있습니다.

객체지향 설계의 기본 원칙(SOLID)을 지키면서 스프링의 싱글톤 관리 기능을 활용한다면, 높은 품질의 소프트웨어를 개발할 수 있을 것입니다.
