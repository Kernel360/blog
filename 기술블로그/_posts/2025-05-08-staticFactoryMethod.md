# 🧭 Java에서 객체를 생성할 때, 왜 정적 팩토리 메서드를 선택할까?

Java에서는 객체를 생성하는 방식으로 `@Setter`, `@Builder`, 그리고 정적 팩토리 메서드(Static Factory Method) 등이 존재합니다. 많은 개발자들이 Lombok의 `@Setter`나 `@Builder`를 선호하지만, 저는 개인적으로 **정적 팩토리 메서드**를 더 선호합니다.

단순한 스타일의 문제가 아니라, **객체의 안정성, 일관성, 유지보수성**을 고려한 실무적인 선택입니다. 이 글에서는 그 이유를 정리하고, 각 방식의 장단점을 비교해보겠습니다.

---

## 📚 목차

1. [Entity에서 Setter, Builder를 지양해야 하는 이유](#1-entity에서-setter-builder를-지양해야-하는-이유)  
   1-1. 도메인 객체의 불변성과 일관성 무너짐  
   1-2. 도메인 규칙 위반 가능  
   1-3. 영속성 컨텍스트와 충돌 가능성

2. [DTO에서 Setter, Builder를 지양해야 하는 이유](#2-dto에서-setter-builder를-지양해야-하는-이유)  
   2-1. DTO는 값 전달 역할만 수행  
   2-2. 불변 객체로 만들면 사이드 이펙트 방지

3. [Setter와 Builder가 가진 구조적 문제](#3-setter와-builder가-가진-구조적-문제)  
   3-1. 깡통 객체 생성 가능  
   3-2. 객체 일관성 무너짐  
   3-3. 필수 필드 강제 불가

4. [정적 팩토리 메서드의 강점](#4-정적-팩토리-메서드의-강점)  
   4-1. 의도를 드러내는 이름  
   4-2. 불변성 보장  
   4-3. 유효성 검사 및 전처리 용이  
   4-4. 다양한 생성 시나리오 지원

5. [실용 예시](#5-실용-예시)

6. [마무리](#6-마무리)

---

## 1. **🚫 Entity에서 Setter, Builder를 지양해야 하는 이유**

### 1-1. **도메인 객체의 불변성과 일관성 무너짐**

- `@Setter`를 사용하면 객체의 내부 상태가 **언제든 외부에 의해 변경**될 수 있습니다.
- 이는 객체의 **일관성**과 **캡슐화 원칙**을 깨뜨려, 도메인 규칙이 흐려집니다.

### 1-2. **도메인 규칙 위반 가능**

- 예: `setPrice(-100)`처럼 잘못된 상태로도 객체가 구성될 수 있음
- 비즈니스 로직이 의도한 제약 조건을 **무력화**할 수 있음

### 1-3. **영속성 컨텍스트와 충돌 가능성**

- JPA는 리플렉션으로 필드를 관리하기 때문에 Builder가 초기화 로직을 어지럽히면 **영속성 컨텍스트 동작에 예기치 않은 영향**을 줄 수 있음

---

## 2. **🚫 DTO에서 Setter, Builder를 지양해야 하는 이유**

### 2-1. **DTO는 값 전달 역할만 수행해야 함**

- DTO는 **순수한 데이터 구조체**로, 상태 변경이 불필요합니다.
- 값이 불변이면 로직이 명확해지고 **테스트가 쉬워집니다.**

### 2-2. **불변 객체로 만들면 사이드 이펙트 방지**

- API 요청이나 응답 객체가 외부에서 바뀌면 **보안과 유지보수 측면에서 리스크**가 큽니다.
- `@Setter`는 어디서든 필드를 바꿀 수 있으므로 추적이 어려워짐

---

## 3. **⚠️ Setter와 Builder가 가진 구조적 문제**

### 3-1. ** 깡통 객체 생성 가능**

`@Setter`, `@Builder`를 사용할 경우, 객체가 **완전한 상태인지 여부를 컴파일 시점에 확인할 수 없습니다.**

```java
User user = new User();
user.setEmail("test@example.com"); // id, name은 세팅되지 않음
```

혹은 Builder를 사용할 경우:

```java
User user = User.builder()
    .email("hello@example.com")
    .build(); // 필수값 누락에도 컴파일 성공
```

→ 이런 객체는 실제 서비스 흐름에서 **NullPointerException, 데이터 무결성 오류**를 일으킬 가능성이 높습니다.

### 3-2. ** 객체 일관성 무너짐**

Setter는 객체를 생성하고 나서 필드를 하나씩 설정하게 되므로, **일시적으로 불완전한 상태**가 발생합니다.  
→ 특히 멀티스레드 환경이나 테스트 코드에서 **예상치 못한 사이드 이펙트**가 발생할 수 있습니다.

### 3-3. ** 필수 필드 강제 불가**

Builder는 유연한 API를 제공하지만, **필수값 누락에 대한 컴파일 타임 검증이 어렵습니다.**

```java
User user = User.builder()
    .email("hello@example.com")
    .birthDate(LocalDate.of(1990, 1, 1))
    .build(); // id, name이 빠졌지만 컴파일 성공
```

→ `id`나 `name`이 필수 필드라면 이는 논리적인 오류를 일으킬 수 있습니다. 그러나 컴파일러는 이를 **잡아주지 못하고**, 오류는 **런타임까지 연기됩니다.**

---

## 4. ** 정적 팩토리 메서드의 강점**

### 4-1. ** 의도를 드러내는 이름**

```java
LocalDate.of(2025, 5, 8); // 명확하고 가독성 좋음
User.create("id123", "Kim", "kim@example.com", birthDate); // 어떤 객체를 만들고 싶은지 한눈에 보임
```

### 4-2. \* 불변성 보장\*\*

- 필드를 `final`로 선언
- 생성자는 `private`으로 감춤
- setter는 아예 제공하지 않음

→ 생성 이후 상태가 바뀌지 않으며, **스레드 안전성 확보**, **디버깅 간편**, **예상 가능한 동작**이라는 큰 이점을 가집니다.

### 4-3. ** 유효성 검사 및 전처리 용이**

→ 생성 시점에 유효성 검사, 전처리, 데이터 정제를 하나의 진입점에서 수행할 수 있습니다.

### 4-4. ** 다양한 생성 시나리오 지원**

```java
User.create(...)                     // 기본 생성
User.withoutBirthDate(...)           // 선택적 필드 제외
User.from(otherUser, "new@email")   // 복사 기반 생성
```

---

## 5. ** 실용 예시**

```java
public class User {
    private final String id;
    private final String name;
    private final String email;
    private final LocalDate birthDate;

    private User(String id, String name, String email, LocalDate birthDate) {
        this.id = Objects.requireNonNull(id, "id must not be null");
        this.name = Objects.requireNonNull(name, "name must not be null");
        this.email = Objects.requireNonNull(email, "email must not be null");
        this.birthDate = birthDate;
    }

    public static User create(String id, String name, String email, LocalDate birthDate) {
        if (!isValidEmail(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
        return new User(id, name, email, birthDate);
    }

    public static User withoutBirthDate(String id, String name, String email) {
        return create(id, name, email, null);
    }

    public static User from(User user, String newEmail) {
        return create(user.id, user.name, newEmail, user.birthDate);
    }

    private static boolean isValidEmail(String email) {
        return email != null && email.contains("@");
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public LocalDate getBirthDate() { return birthDate; }
}
```

---

## 6. ** 마무리**

정적 팩토리 메서드는 단순한 객체 생성 방식을 넘어, **안전하고 명확한 객체 설계의 기본**입니다.  
편리함만 추구하다 보면 시스템은 점점 **불안정하고 복잡한 구조**로 흘러갑니다.

`@Setter`, `@Builder`는 분명 생산성 측면에서 도움이 되는 도구이지만, **장기적으로 유지보수하고 협업하는 코드**를 만든다면, 정적 팩토리 메서드는 그 어떤 패턴보다 강력한 무기가 될 수 있습니다.
