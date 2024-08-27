---
layout: post
title: 정적 팩토리 메서드 패턴
author: 김택준
categories: 기술세미나
banner:
image: assets/images/post/2023-11-05.webp
background: “#000”
height: “100vh”
min_height: “38vh”
heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [정적 팩토리 메서드 패턴, static, factory method pattern]
---

# Static Factory Method Pattern

## 정적 팩토리 메서드 패턴이란?

지금까지 객체를 인스턴스화 할 때 직접적으로 생성자를 호출하고 생성했는데, 이와는 반대로 객체 생성 역할을 하는 메서드를 통해 간접적으로 객체 생성을 유도하는 것이 정적 팩토리 메서드 패턴이다.<br>
조슈아 블로크의 이펙티브 자바에서 소개하는 아이템 중에서 첫 번째로 조언하는 것이 ‘생성자 대신 정적 팩토리 메서드를 고려하라’ 인데, 왜 이렇게 한단계 더 거쳐서 정적 팩토리 메서드를 통해 객체를 생성하라는 것인가에 대해 그 장점과 단점을 알아보자.

## 정적 팩토리 메서드의 장점

### 1. 생성 목적에 따라 네이밍이 가능하다

정적 팩토리 메서드는 메서드 이름으로 객체 생성의 목적을 명확히 할 수 있다.<br>
이는 다양한 생성자를 오버로딩하여 사용하는 것보다 코드의 가독성을 높여준다.

적절한 메서드 네이밍 예시:
```java
public class User {
    private String name;
    private int age;

    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static User createAdult(String name, int age) {
        return new User(name, age); // Assume adult age validation
    }

    public static User createMinor(String name, int age) {
        return new User(name, age); // Assume minor age validation
    }
}
```
위 예제에서 createAdult와 createMinor 메서드는 각각의 객체 생성 목적을 명확히 한다.

부적절한 메서드 네이밍 예시:
```java
public class User {
    private String name;
    private int age;

    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static User create(String name, int age) {
        return new User(name, age);
    }

    public static User initialize(String name, int age) {
        return new User(name, age);
    }
}
```
create와 initialize는 메서드의 목적을 명확히 드러내지 않으며, 객체의 용도나 특성을 이해하기 어렵다.

### 네이밍 컨벤션 참고 =><br>
https://docs.oracle.com/javase%2Ftutorial%2F/datetime/overview/naming.html
<img width="1444" alt="스크린샷 2024-08-27 17 47 52" src="https://github.com/user-attachments/assets/008b75a1-0506-4393-9a6a-1262f57c8ccd">

### 2. 인자에 따라 다른 객체를 반환하도록 분기할 수 있다

정적 팩토리 메서드는 매개변수에 따라 다양한 객체를 반환할 수 있어 유연한 객체 생성을 지원한다.

예시 코드:
```java
public class User {
    private String name;
    private int age;

    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static User createUser(String type, String name, int age) {
        if ("admin".equalsIgnoreCase(type)) {
            return new AdminUser(name, age);
        } else if ("regular".equalsIgnoreCase(type)) {
            return new RegularUser(name, age);
        } else {
            throw new IllegalArgumentException("Unknown user type");
        }
    }
}

class AdminUser extends User {
    public AdminUser(String name, int age) {
        super(name, age);
    }
}

class RegularUser extends User {
    public RegularUser(String name, int age) {
        super(name, age);
    }
}
```
위 코드에서 createUser 메서드는 type 매개변수에 따라 AdminUser 또는 RegularUser 인스턴스를 반환한다.

### 3. 객체 생성을 캡슐화할 수 있다

정적 팩토리 메서드는 객체 생성의 내부 구현을 캡슐화할 수 있다.<br>
생성자가 private으로 설정되면 외부에서 직접 객체를 생성할 수 없으며, 정보 은닉성과 의존성 제거의 장점을 제공한다.

예시 코드:
```java
public class DatabaseConnection {
    private String url;

    private DatabaseConnection(String url) {
        this.url = url;
    }

    public static DatabaseConnection createConnection(String url) {
        // Validate URL or other logic
        return new DatabaseConnection(url);
    }
}
```
DatabaseConnection 클래스는 private 생성자를 사용하고 createConnection 메서드를 통해 객체를 생성한다.<br>
아래는 mermaid를 이용한 구조 이미지이다. 참고하면 좋을듯<br>
![스크린샷 2024-08-22 18 37 56](https://github.com/user-attachments/assets/a5ac9083-16f4-4b63-af84-960c87b90685)

### 4. 불필요한 객체 생성을 막을 수 있다

정적 팩토리 메서드는 객체를 재사용할 수 있도록 하여 불필요한 객체 생성을 방지한다.

예시 코드:
```java
public class BooleanValue {
    private boolean value;

    private BooleanValue(boolean value) {
        this.value = value;
    }

    public static BooleanValue of(boolean value) {
        if (value) {
            return TRUE_INSTANCE;
        } else {
            return FALSE_INSTANCE;
        }
    }

    private static final BooleanValue TRUE_INSTANCE = new BooleanValue(true);
    private static final BooleanValue FALSE_INSTANCE = new BooleanValue(false);
}
```
위 코드에서 BooleanValue.of 메서드는 true와 false에 대해 미리 생성된 인스턴스를 반환하여 객체 생성을 최적화한다.

## 정적 팩토리 메서드 패턴의 단점

### 1. 상속을 이용한 확장이 불가능하다

정적 팩토리 메서드를 사용하면 생성자가 private으로 설정되어 상속을 통한 확장이 불가능하다.<br> 이는 객체의 상속보다는 합성을 유도하거나 불변 객체로 만들 때 유용하다.

예시 코드:
```java
public class User {
    private String name;
    private int age;

    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static User create(String name, int age) {
        return new User(name, age);
    }
}

// 상속을 시도하는 코드
public class PremiumUser extends User {
    // Compilation error: User() has private access in User
}
```
User 클래스의 생성자가 private이므로 PremiumUser 클래스를 상속할 수 없다.

### 2. API 문서의 불편함

정적 팩토리 메서드는 생성자에 비해 API 문서화가 어려울 수 있다.<br> 
메서드의 목적과 사용법을 명확히 설명하는 것이 필요하다.

예시:
```java
public class Logger {
    private String level;

    private Logger(String level) {
        this.level = level;
    }

    /**
     * Creates a logger instance with the specified logging level.
     * @param level The logging level.
     * @return A new Logger instance.
     */
    public static Logger getLogger(String level) {
        return new Logger(level);
    }
}
```
getLogger 메서드의 목적과 사용법을 명확히 문서화하여 API 사용자에게 제공해야 한다.
