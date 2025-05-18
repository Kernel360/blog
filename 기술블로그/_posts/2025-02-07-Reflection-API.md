---
layout: post  
title: "JPA 엔티티 설계에서 기본 생성자가 필요한 이유 : Reflection API와 접근제어자의 역할"
author: "김대현"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["JAVA", "JPA", "ORM", "Reflection", "Hibernate", "Proxy", "LazyLoading"]
---



`Entity`설계 시 멘토님으로부터 "현업에서는 `Entity`에`Setter`나 `AllArgsConstructor`를 사용하지 않고 기본생성자(`NoArgsConstructor`)만 작성 후 정적 팩토리 메서드를 사용해 `Entity`객체를 생성한다"라는 피드백을 받았습니다.

그래서 "팩토리 메서드 내에 기본 생성자를 사용하도록 구현할 테니 기본 생성자를 객체 외부에서 사용하지 못하도록 해야겠지?" 라는 생각으로(~~뇌를 비우고~~) accessLevel을 **Private**로 설정했는데 `InstantiationException`이 발생했습니다.

`Reflection API`와 JPA의 `Entity`관리에 대한 이해 부족으로 생긴 문제였고 정확히 이해하기 위해 학습을 진행했습니다.

---
# 1. Reflection API
## 1-1. Reflection API?
JAVA의 `Reflection API`는 런타임에 클래스의 정보를 탐색하고 필드, 메서드, 생성자 등에 접근할 수 있는 기능을 제공합니다.
컴파일 시점에 클래스의 구체적인 내용을 알지 못해도 런타임 시점에 클래스의 구조를 파악하고 해당 클래스의 객체를 생성하거나 메서드를 호출할 수 있습니다.

#### ⚙️ 이게 어떻게 가능한가?
> 
자바로 작성된 프로그램은 먼저 **Java 컴파일러(javac)**에 의해 `.java` 소스 코드가 **바이트코드(`.class` 파일)**로 변환됩니다. 프로그램이 실행될 때, **JVM**은 이 `.class` 파일을 읽고, **클래스 로더(ClassLoader)**를 통해 메모리에 로드합니다. 이때 클래스의 메타데이터(필드, 메서드, 생성자 정보 등)가 **JVM의 메소드 영역(Method Area)**에 저장됩니다.(Java 8 이전엔 Method 영역이 맞으나 Java 8 부터는 **Metaspace**에 저장됨. Heap 메모리가 아닌 Native 메모리를 사용함.)
>
`Reflection API`는 이 메소드 영역에 저장된 클래스의 메타데이터에 접근하여, 런타임 시점에 클래스의 구조를 탐색하거나 객체를 생성할 수 있습니다.(**동적으로 접근/변경이 가능하다는 뜻**) 이를 통해 클래스 이름만으로도 필드, 메서드, 생성자와 같은 대부분의 정보를 가져올 수 있게됩니다.

Spring에서는 DI, Proxy등에서 사용됩니다.
위와 같은 아주 강력한 기능들을 제공하며 이러한 장점들로 보통 프레임워크, 라이브러리 개발에서 사용합니다.

## 1-2. 그럼 AccessLevel.PRIVATE이 문제될게 없지 않나??
#### ❌ 자바의 보안 모델로 인해 접근이 불가능하다.
>
위에서 서술했듯 `Reflection API`는 런타임에 클래스의 정보를 탐색하고 접근하는 기능을 제공합니다.
하지만 Java는 기본적으로 **캡슐화(encapsulation)**를 지원하기 때문에, **클래스의 private 멤버는 Reflection으로도 바로 접근할 수 없도록 제한**되어 있습니다.

```Java
Class<?> clazz = User.class;

// private 생성자를 가져옴
Constructor<?> constructor = clazz.getDeclaredConstructor();
System.out.println(constructor); // 정상적으로 출력됨
```
이처럼 클래스의 private 생성자나 필드 정보는 가져올 수 있다.

```Java
Constructor<User> constructor = User.class.getDeclaredConstructor();
constructor.newInstance(); // IllegalAccessException
```
하지만 이를 **호출하거나 값을 수정**하려고 할 때는 접근제한(Access Control)이 적용됩니다.

## 1-3. JPA와 Reflection API
`Reflection API`는 JPA에서 객체 생성, 필드 값 매핑, Lazy Loading, 콜백 메서드 호출 등 다양한 작업에 사용됩니다. 이를 통해 Setter 메서드 없이도 private 필드에 직접 접근하거나, Proxy 객체를 생성할 수 있습니다.

#### 1) 엔티티 클래스 접근
JPA는 엔티티 클래스의 필드, 메서드, 생성자, 어노테이션 정보를 동적으로 탐색하여 엔티티 클래스를 분석한다.
```Java
Class<?> clazz = Class.forName("com.example.User");
```

#### 2) 객체 생성
`Reflection ApI`는 **기본 생성자**를 사용해 엔티티 객체를 생성합니다.
JPA도 `Reflection API`를 활용하여 엔티티 객체를 생성하므로 **엔티티 클래스에는 반드시 기본 생성자가 필요합니다.**
```Java
Constructor<?> constructor = clazz.getDeclaredConstructor();
constructor.setAccessible(true); // private 생성자에도 접근 가능
Object entity = constructor.newInstance();
```

#### 3) 필드 값 설정
객체 생성 후 DB의 값과 엔티티 객체를 매핑하기 위해 `Refection API`를 사용합니다.
```Java
Field field = clazz.getDeclaredField("name"); //클래스명으로 정보 가져오기
field.setAccessible(true); // private 필드에도 접근 허용
field.set(entity, "bik_kyun");
```
이 과정에서 `Setter` 메서드를 호출하지 않고 필드에 직접 값이 설정됩니다.

#### 4) 메서드 호출
JPA는 특정 상황에서 엔티티 클래스의 메서드를 호출하기 위해 `Reflection API`를 사용합니다.
ex) 콜백 메서드(`@PrePersist`, `@PostLoad`)
```Java
Method prePersistMethod = clazz.getDeclaredMethod("prePersist");
prePersistMethod.setAccessible(true);
prePersistMethod.invoke(entity);
```

#### 5) Lazy Loading과 Proxy 객체
>**(Hibernate)Proxy 객체** : 실제 엔티티 클래스를 **상속**받은 객체.

JPA는 `Lazy Loading`을 구현하기 위해 Proxy 객체를 동적으로 생성합니다.
이때 `Reflection API`를 사용하여 Proxy 객체를 생성하며, 즉시 초기화하지 않고 DB에도 접근하지 않습니다.
실제 데이터가 필요한 시점(호출 시점)이 되면 Proxy 객체는 해당 호출을 가로채 DB 쿼리를 실행하며 필요한 데이터를 로드합니다.
로드가 완료되면 Proxy 객체는 실제 엔티티의 데이터로 초기화된 상태가 됩니다.(**호출 시점 전까지는 Proxy 객체로 유지된다.**)
```Java
//Hibernate 내부
User proxyUser = (User) Proxy.newProxyInstance(
    User.class.getClassLoader(),
    new Class[]{User.class},
    (proxy, method, args) -> {
        System.out.println("프록시 메서드 호출: " + method.getName());
        return null; // 실제 데이터는 Lazy 로딩 시 가져옴
    }
);
```

# 2. 결론
>`@NoArgsConstructor(access = AccessLevel.PROTECTED)`

1. JPA는 `Reflection API`를 이용해 엔티티 객체를 생성하고 데이터베이스 값을 매핑합니다. 이 과정에서 **기본 생성자가 필수적**입니다.

2. Lazy Loading시 생성되는 Proxy 객체는 엔티티(부모클래스)의 자식클래스이며, **이를 생성할 때 기본 생성자가 호출됩니다**.

3. 기본 생성자가 `private`으로 선언되어 있으면 해당 엔티티를 상속한 Proxy 객체를 만들 수 없게됩니다.
또한, 상속받은 클래스에서 부모 객체의 생성자(`super()`)를 호출할 수 없습니다.
>✅ **참고** - 부모클래스와 자식클래스의 생성자 호출
부모 클래스로부터 상속받은 메소드 및 필드는 부모 클래스에 정의된 것이고 부모 클래스의 것이다. 
따라서 부모 클래스의 생성자가 호출되어야 자식 클래스에서 사용이 가능하다.
>
부모 클래스의 생성자(`super()`)는 자식 클래스의 생성자로 인스턴스를 생성할 때 자동으로 호출된다.
순서 : 부모클래스 호출 -> 자식클래스 생성자 호출

4. 따라서, 기본 생성자는 JPA 엔티티가 동작하는 데 있어 반드시 필요하며, **실제 엔티티(부모클래스)** 또한 `public` 또는 `protected`인 기본 생성자가 존재해야 **Proxy 객체(자식클래스)** 도 기본 생성자를 사용할 수 있기 때문에 실제 엔티티에도 **기본 생성자가 필요**한 것입니다.

5. `public`은 객체 외부에서 기본 생성자 접근이 가능하게 되므로 `public`과 `private`의 타협점인 `protected`를 사용하여 **불필요한 객체 생성을 막습니다.**

>✅ **참고**
IntelliJ를 사용하면 `public`이나 `protected`로 선언된 기본 생성자가 없는 클래스에 `Class 'XXX' should have [public, protected] no-args constructor`라는 경고를 볼 수 있지만, 기본 생성자의 접근 제어자에 관련된 예외는 런타임 예외이기 때문에 즉시 로딩을 사용하거나 프록시를 사용할 일이 없다면 관련 예외가 발생하지 않는다.
