---
layout: post  
title: "람다를 사용하는 이유"
author: "남유람"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["java", "lambda", "method_reference"]
---

# 람다에 대하여

생성일: 2025년 2월 5일 오후 2:12

## **📌 사전 지식**

---

### **🔹 내부 클래스**

- **클래스 내부에 선언된 클래스**
- 내부 클래스의 유형1️⃣ **인스턴스 내부 클래스**2️⃣ **정적 내부 클래스**3️⃣ **지역 내부 클래스**

```java
class Outer {               // 외부(Top-level) 클래스
    class Inner {           // 내부(Non-static) 클래스
        static class SIn {} // 정적(Static) 내부 클래스
    }

    static class StaticInner {}  // 정적(Static) 내부 클래스

    public void abc() {
        class Local {} // 지역(Local) 내부 클래스
    }
}
```

---

### **🔹 익명 내부 클래스**

- **클래스 이름 없이 선언하고 바로 생성하여 사용할 수 있는 내부 클래스**
- **인터페이스 또는 추상 클래스를 단 하나만 바로 생성 가능**

---

## **🚀 람다식**

### **🔹 함수형 프로그래밍과 람다식**

- 자바는 **객체 기반 프로그래밍** 방식
- 어떤 기능이 필요하면 **클래스를 먼저 만들고**, 그 안에 기능을 구현한 **메서드를 만든 후 호출해야 함**
- 즉, **클래스가 없다면 메서드를 사용할 수 없음**
- 반면 **함수형 프로그래밍**은 **함수의 구현과 호출만으로 프로그램을 만들 수 있는 방식**
- 자바에서는 **람다식(lambda expression)**을 통해 함수형 프로그래밍을 지원

---

### **🔹 람다식 문법**

- 람다식은 **이름 없는 익명 함수**를 만드는 문법
- 기존 메서드와 달리 **메서드 이름과 반환형**을 없애고 `→` 사용
- 매개변수 자료형, 괄호, `return` 생략 가능

### **📌 기존 방식 (메서드)**

```java
int add(int x, int y) {
    return x + y;
}
```

### **📌 람다식**

```java
(x, y) -> { return x + y; }
(x, y) -> x + y  // return 생략 가능
```

---

### **🔹 함수형 인터페이스**

- 람다식을 사용하려면 **함수형 인터페이스**가 필요함1️⃣ 먼저 **인터페이스 생성**2️⃣ 인터페이스에 **람다식으로 구현할 메서드 선언**
- 함수형 인터페이스에는 **메서드를 하나만 선언해야 함**
- `@FunctionalInterface` 어노테이션을 사용하여 두 개 이상 선언하지 않도록 예방

---

## **💡 객체 지향 프로그래밍 방식과 람다식 비교**

### **🟢 공통 - `StringConcat` 인터페이스 만들기**

```java
public interface StringConcat {
    public void makeString(String s1, String s2);
}
```

---

### **1️⃣ 클래스에서 인터페이스 구현**

```java
public class StringConcatImpl implements StringConcat {
    @Override
    public void makeString(String s1, String s2) {
        System.out.println(s1 + "," + s2);
    }
}
```

```java
public class TestStringConcat {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "World";

        StringConcatImpl concat1 = new StringConcatImpl();
        concat1.makeString(s1, s2);
    }
}
```

---

### **2️⃣ 람다식으로 인터페이스 구현**

```java
ublic class TestStringConcat {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "World";

        StringConcat concat2 = (s, v) -> System.out.println(s + "," + v);
        concat2.makeString(s1, s2);
    }
}
```

✅ **람다식을 사용하면 코드가 더 간결해짐**

---

### **3️⃣ 익명 객체를 생성하는 람다식**

```java
public class TestStringConcat {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "World";

        // 익명 객체로 구현
        StringConcat concat3 = new StringConcat() {
            @Override
            public void makeString(String s1, String s2) {
                System.out.println(s1 + "," + s2);
            }
        };
        concat3.makeString(s1, s2);
    }
}
```

---

## **🛠 함수를 변수처럼 사용하는 람다식**

람다식을 사용하면 **함수를 변수처럼 사용할 수 있음**

### **1️⃣ 인터페이스형 변수에 람다식 대입**

```java
PrintString lambdaStr = s -> System.out.println(s);
lambdaStr.showString("hello lamda_1");
```

---

### **2️⃣ 매개변수로 전달되는 람다식**

```java
public interface PrintString {
    void showString(String s);
}

public class TestLambda {
    public static void main(String[] args) {
        PrintString lambdaStr = s -> System.out.println(s);
        lambdaStr.showString("Hello Lambda_1");
        showMyString(lambdaStr);
    }

    public static void showMyString(PrintString p) {
        p.showString("Hello Lambda_2");
    }
}
```

---

### **3️⃣ 반환값으로 쓰이는 람다식**

```java
package ch13.lambda;

public class TestLambda {
    public static void main(String[] args) {
        PrintString reStr = returnString(); // 변수로 반환받기
        reStr.showString("hello "); // 메서드 호출
    }

    public static PrintString returnString() {
        return s -> System.out.println(s + "world");
    }
}
```

---

## **❓ 익명 클래스 말고 람다를 사용하는 이유**

✅ **람다식을 사용하면 코드가 더 짧아지고 가독성이 향상됨**

### **📌 익명 클래스 사용 코드**

```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

### **📌 람다식 사용 코드**

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

---

## **⚠ 람다 주의 사항 및 제약 사항**

1️⃣ 람다는 **메서드나 클래스와 달리 이름이 없고 문서화가 불가능함**

2️⃣ **열거 타입(enum) 생성자 안의 람다는 인스턴스 멤버에 접근할 수 없음**

3️⃣ 람다는 **함수형 인터페이스에서만 사용 가능**

4️⃣ 람다는 **자기 참조**가 안된다

5️⃣ **람다는 직렬화 형태가 구현별로 다를 수 있으므로 직렬화가 필요한 경우 사용 지양**

---

---

### **📌 마무리**

- 내부 클래스와 람다식의 차이점을 이해하고,
- 코드의 **가독성과 유지보수성**을 높일 수 있도록 적절한 방법을 선택하여 사용하자!
