---
layout: post  
title: "열거 타입을 사용해야 하는 이유"
author: "최윤서"
banner:
  image: "https://github.com/Kernel360/blog-image/blob/main/2025/0202/why-use-enum.jpg?raw=true"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["enum", "열거타입", "정수 상수", "상수"]
---

# **상수를 정의할 때 열거 타입(`enum`)을 사용해야 하는 이유**

개발할 때 특정한 값을 상수로 정의하는 경우가 많습니다. 과거에는 `int` 타입의 상수를 나열하는 방식(정수 열거 패턴)이 일반적이었지만, 오늘날에는 **열거 타입(`enum`)이 더 많이 사용됩니다**

그렇다면 단순히 "많은 개발자가 사용하기 때문"이 아니라, **왜 `enum`을 사용해야 하는지** 정확히 이해하는 것이 중요합니다

본 글에서는 **정수 열거 패턴의 단점과 `enum`이 제공하는 이점**을 비교하며, **열거 타입을 사용하는 이유**를 공유하고자 합니다



## **정수 열거 패턴의 문제점**

열거 타입이 등장하기 전에는 정수(int) 상수를 한 묶음으로 선언해 사용했습니다.

```java

public static final int IVE_RAY = 0;
public static final int IVE_WONYOUNG = 1;
public static final int IVE_RIZ = 2;

public static final int AESPA_CARINA = 0;
public static final int AESPA_WINTER = 1;

```

이러한 방식을 **정수 열거 패턴**이라고 하는데, 여러 단점이 있습니다

1. **타입 안전성을 보장할 수 없다**
    
    → 다른 그룹의 상수가 동일한 정수 값을 가질 경우, 구분이 어렵다
    
2. **의미가 명확하지 않다**
    
    → 숫자로만 표현되므로, 코드만 봐서는 어떤 값인지 직관적으로 이해하기 어렵다
    
3. **잘못된 비교를 해도 컴파일 오류가 발생하지 않는다**
    
    → 서로 관련 없는 정수 값도 비교할 수 있어, 오류가 발생할 가능성이 크다
    
4. **프로그램이 쉽게 깨질 수 있다**
    - 상수 값을 변경하면, 이를 사용하는 모든 코드를 다시 컴파일해야 한다
    - 다시 컴파일하지 않으면, 프로그램이 예기치 않게 동작할 가능성이 있다
5. **출력하기 어렵다**
    - 정수 값만 출력되므로, 별도의 매핑 테이블이 필요하다
    - 문자열 값으로 정의하면 오타로 인한 오류가 발생할 가능성이 크다

이러한 단점을 해결하고, 더 나은 기능을 제공하는 것이 바로 **열거 타입(enum type)** 입니다.

```java
public enum IVE { RAY, WONYOUNG, RIZ }
public enum AESPA { CARINA, WINTER }
```

> 🤔 enum이란?
> - 열거 타입 자체가 클래스이며, 각 상수는 **해당 클래스의 인스턴스(정적 상수 객체)** 임
> - 사용자가 인스턴스를 직접 생성하거나 확장할 수 없어, **각 상수는 단 하나의 인스턴스임이 보장함**

그럼 이제, **열거 타입이 정수 열거 패턴의 단점을 어떻게 해결하는지** 하나씩 살펴보겠습니다.

---

## 1. 타입 안전성을 보장한다

### ❌ 정수 열거 패턴

```java
public static final int IVE_RAY = 0;
public static final int IVE_WONYOUNG = 1;
public static final int AESPA_CARINA = 0;

public class GroupChecker {
    public static void checkMember(int member) {
        if (member == IVE_RAY) {
            System.out.println("Ray is in IVE");
        } else if (member == AESPA_CARINA) {
            System.out.println("Carina is in AESPA");
        }
    }
}
```

위 코드에서 `IVE_RAY`와 `AESPA_CARINA`의 값이 동일한 `0`이므로, `checkMember(0);`을 호출하면 **어떤 그룹인지 정확하게 판별할 수 없습니다**

### ✅ 열거 타입 사용

```java
public enum IVE { RAY, WONYOUNG }
public enum AESPA { CARINA, WINTER }

public class GroupChecker {
    public static void checkMember(IVE member) {
        if (member == IVE.RAY) {
            System.out.println("Ray is in IVE");
        }
    }
}
```

`IVE.RAY`와 `AESPA.CARINA`는 **서로 다른 타입이므로** `checkMember(AESPA.CARINA);`을 호출하면 **컴파일 오류가 발생합니다**

즉, `enum`을 사용하면 **잘못된 비교를 원천 차단**할 수 있습니다.

## 2. 의미가 명확하고, 출력도 직관적이다

정수 열거 패턴을 사용할 경우, **숫자 값 자체가 의미를 전달하지 못하는 문제**가 있습니다

또한, 상수를 출력할 때 숫자만 보이므로 **이 값이 무엇을 의미하는지 알기 어렵습니다**

### ❌ 정수 열거 패턴

```java
public static final int IVE_RAY = 0;
public static final int IVE_WONYOUNG = 1;

public static void printMember(int member) {
    System.out.println("Member: " + member);
}

printMember(IVE_RAY); // 출력 -> Member: 0
```

위 코드에서 출력 결과인 `0`은 어떤 **멤버를 의미하는지 직관적으로 알기 어렵습니다**

개발자는 코드와 주석을 찾아봐야 하고, 유지보수도 번거롭습니다

이때, 출력 시 사람이 이해할 수 있는 형태로 변환하려면 아래와 같이 별도의 매핑 테이블이 필요합니다


```java
public static String getMemberName(int member) {
    switch (member) {
        case 0: return "RAY";
        case 1: return "WONYOUNG";
        default: return "UNKNOWN";
    }
}

System.out.println("Member: " + getMemberName(IVE_RAY)); // 출력 -> Member: RAY
```
하지만 이런 방식은 **추가적인 코드 작성이 필요하고, 실수할 가능성도 높아집니다**

### ✅ 열거 타입 사용

```java
public enum IVE { RAY, WONYOUNG }

System.out.println("Member: " + IVE.RAY); // 출력 -> Member: RAY
```
`enum`을 사용하면 **값 자체가 의미를 전달**하므로, 별도로 숫자를 변환하지 않아도 직관적인 출력을 얻을 수 있습니다

또한, `toString()` 메서드가 자동으로 적용되기 때문에 **의미 있는 문자열이 출력**됩니다

필요하다면 `name()`을 활용해 보다 명확하게 출력할 수도 있습니다

```java
System.out.println("Member: " + IVE.RAY.name()); // 출력 -> Member: RAY
```

한국어 이름을 추가하고 싶을 수 있습니다. 이땐 필드를 추가해 더 유용한 정보를 담을 수도 있습니다

```java
public enum IVE {
    RAY("레이"),
    WONYOUNG("원영");

    private final String koreanName;

    IVE(String koreanName) {
        this.koreanName = koreanName;
    }

    public String getKoreanName() {
        return koreanName;
    }
}

System.out.println("Member: " + IVE.RAY.getKoreanName()); // 출력 -> Member: 레이
```

## 3. 상수를 추가하거나 변경해도 안전하다

### ❌ 정수 열거 패턴

```java
public static final int IVE_WONYOUNG = 1;
```

위 코드에서 `IVE_WONYOUNG` 값을 `3`으로 변경한다면 어떻게 될까요?

이 값을 사용하는 클라이언트 코드가 **다시 컴파일 되지 않을 경우** 여전히 `1`을 참조하여 오류가 발생할 수 있습니다

### ✅ 열거 타입 사용

```java
public enum IVE { RAY, WONYOUNG, RIZ }
IVE member = IVE.WONYOUNG;
```

`enum`은 내부적으로 어떤 값이든 변경해도 클라이언트 코드에는 영향을 주지 않습니다

따라서 **새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일할 필요가 없습니다**


## 4. 상수를 제거하면 참조하는 코드에서 컴파일 오류 발생

### ❌ 정수 열거 패턴

정수 열거 패턴에서는 **기존의 상수를 제거해도, 이를 참조하는 코드에서 컴파일 오류가 발생하지 않습니다**

이는 **잘못된 값이 계속 사용될 가능성을 높이고, 오류를 찾기 어렵게 만듭니다**

```java
public class Constants {
    public static final int IVE_RAY = 0;
    public static final int IVE_WONYOUNG = 1;
    public static final int IVE_RIZ = 2;
}

// 나중에 WONYOUNG 제거
public class Constants {
    public static final int IVE_RAY = 0;
    public static final int IVE_RIZ = 2;
}

public class Main {
    public static void main(String[] args) {
        int member = Constants.IVE_WONYOUNG; // 컴파일 오류 발생하지 않음
        System.out.println("Member: " + member);
    }
}
```

위 코드에서 `IVE_WONYOUNG`을 제거했지만, **참조하는 코드에서 여전히 컴파일 오류가 발생하지 않습니다**

하지만 이 코드가 실행되면 `IVE_WONYOUNG`이 정의되지 않았으므로, **런타임 오류나 예상치 못한 동작이 발생할 수 있습니다**

### ✅ 열거 타입 사용

```java
public enum IVE { RAY, WONYOUNG, RIZ }

// 이후 WONYOUNG을 제거
public enum IVE { RAY, RIZ }

public class Main {
    public static void main(String[] args) {
        IVE member = IVE.WONYOUNG; // 컴파일 오류 발생!
    }
}
```

`enum`에서는 `IVE.WONYOUNG`이 제거되면, 이를 참조하는 코드에서 **즉시 컴파일 오류가 발생**합니다

개발자는 **잘못된 참조를 바로 확인하고 수정할 수 있으며**, 예상치 못한 런타임 오류를 방지할 수 있습니다


## 5. 객체처럼 활용할 수 있으며, 복잡한 개념도 표현할 수 있다

일반적으로 `enum`은 단순한 상수를 정의하는 데 사용되지만, 사실 **필드와 메서드를 가질 수 있는 강력한 클래스**입니다.

이를 활용하면, 단순한 값만 저장하는 것이 아니라 **더 복잡한 개념도 표현할 수 있습니다.**


예를 들어, 각 그룹의 이름뿐만 아니라 **데뷔 연도와 대표곡까지 포함하는 `enum`** 을 만들어 보겠습니다

```java
public enum Group {
    IVE("아이브", 2021, "ELEVEN"),
    AESPA("에스파", 2020, "Black Mamba");

    private final String koreanName;
    private final int debutYear;
    private final String hitSong;

    Group(String koreanName, int debutYear, String hitSong) {
        this.koreanName = koreanName;
        this.debutYear = debutYear;
        this.hitSong = hitSong;
    }

    public String getKoreanName() {
        return koreanName;
    }

    public int getDebutYear() {
        return debutYear;
    }

    public String getHitSong() {
        return hitSong;
    }

    public String getInfo() {
        return String.format("%s(%s) - 데뷔: %d년, 대표곡: %s", this.name(), koreanName, debutYear, hitSong);
    }
}
```

```java
System.out.println(Group.IVE.getInfo());
// 출력 -> IVE(아이브) - 데뷔: 2021년, 대표곡: ELEVEN
```

이렇게 enum은 **관련 정보와 함께 객체처럼 관리할 수 있고,** 

필요한 정보(대표곡, 데뷔 연도 등)를 **메서드로 바로 가져올 수 있어 가독성이 높일 수 있습니다**


또한, `enum`에서는 **각 열거 상수마다 다른 동작을 하도록 추상 메서드를 정의할 수도 있습니다**

그룹별로 서로 다른 콘셉트를 설명하는 메서드를 추가해 보겠습니다

```java
public enum Group {
    IVE("아이브") {
        @Override
        public String getConcept() {
            return "세련되고 고급스러운 컨셉";
        }
    },
    AESPA("에스파") {
        @Override
        public String getConcept() {
            return "메타버스와 AI 세계관을 활용한 컨셉";
        }
    };

    private final String koreanName;

    Group(String koreanName) {
        this.koreanName = koreanName;
    }

    public String getKoreanName() {
        return koreanName;
    }

    // 추상 메서드 선언 (각 열거값이 개별적으로 구현해야 함)
    public abstract String getConcept();
}
```

```java
System.out.println(Group.IVE.getKoreanName() + "의 컨셉: " + Group.IVE.getConcept());
// 출력 -> 아이브의 컨셉: 세련되고 고급스러운 컨셉

System.out.println(Group.AESPA.getKoreanName() + "의 컨셉: " + Group.AESPA.getConcept());
// 출력 -> 에스파의 컨셉: 메타버스와 AI 세계관을 활용한 컨셉
```

이렇게 추상메서드를 정의하면 각 상수마다 **고유한 동작을 정의할 수 있어 유연성이 높아집니다**

---

## 결론

|  | 정수 열거 패턴 | 열거 타입 (`enum`) |
| --- | --- | --- |
| 타입 안전성 | 없음 | 있음 (다른 타입과 비교 불가능) |
| 가독성 | 숫자로 표현됨 | 의미 있는 이름 사용 |
| 의미 있는 문자열로 출력 | 별도 매핑 필요 | `toString()` 자동 지원 |
| 변경 시 재컴파일 필요성 | 있음 | 없음 |
| 상수 제거 후 컴파일 오류 여부 | 발생하지 않음 | 발생 (잘못된 참조 즉시 감지 가능) |
| 확장성 | 없음 | 필드, 메서드 추가 가능 |


정수 상수(int 값)를 사용하던 시절에는 코드가 복잡해지고 실수할 여지가 많았습니다. 값이 헷갈리거나, 잘못된 비교가 이루어지더라도 컴파일러가 잡아주지 못하는 경우가 많았죠

하지만 enum을 사용하면 타입 안정성을 확보할 수 있고, 가독성이 좋아지며, 유지보수도 훨씬 편해집니다
또한, 단순한 상수가 아니라 필드와 메서드를 가질 수 있는 객체처럼 활용할 수 있기 때문에 확장성도 뛰어납니다
> "이 코드가 어떤 의미를 갖는지 고민할 필요 없이, 읽기만 해도 이해할 수 있다!"

enum은 단순한 코드 스타일의 차이가 아니라, 더 좋은 코드를 만들기 위한 선택입니다
앞으로 상수를 정의할 때는 enum을 적극 활용해 보세요!
