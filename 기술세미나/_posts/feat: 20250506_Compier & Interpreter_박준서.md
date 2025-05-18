---
layout: post
title: "COMPILER and INTERPRETER"
author: "박준서"
categories: "기술세마나"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["JAVA"]
---

## 주제: COMPILER and INTERPRETER

# 컴파일러와 인터프리터

컴파일러와 인터프리터는 프로그래밍 언어를 실행 가능한 형태로 변환하는 핵심 도구입니다. 오늘은 이 두 가지 실행 방식의 차이점과 특히 자바 인터프리터의 구현 방식이 어떻게 진화해왔는지 살펴보겠습니다.

## 컴파일러 vs 인터프리터: 기본 개념

**컴파일러**는 소스 코드 전체를 한 번에 분석하여 기계어나 중간 코드로 변환합니다. 이렇게 생성된 실행 파일은 이후에 별도의 번역 과정 없이 실행될 수 있습니다. C, C++, Rust와 같은 언어들이 전통적인 컴파일 방식을 사용합니다.

**인터프리터**는 소스 코드를 한 줄씩 읽어 즉시 실행합니다. 별도의 실행 파일을 생성하지 않고, 코드를 분석하면서 바로 실행합니다. Python, JavaScript, Ruby와 같은 언어들이 주로 인터프리터 방식으로 실행됩니다.

## 자바의 특별한 위치

자바는 이 두 가지 방식을 혼합한 형태를 취합니다:
1. 자바 컴파일러(javac)가 소스 코드(.java)를 바이트코드(.class)로 컴파일
2. 자바 가상 머신(JVM)이 이 바이트코드를 인터프리트하거나 JIT(Just-In-Time) 컴파일을 통해 실행

## 자바 인터프리터의 내부 구현: Big While Loop와 Switch 문

초기 자바 인터프리터는 '큰 while 루프와 switch 문' 패턴으로 구현되었습니다. 이 패턴은 다음과 같이 작동합니다:

```java
void interpret() {
    while (hasMoreBytecode()) {
        byte opcode = fetchNextOpcode();
        switch (opcode) {
            case IADD:
                // 정수 덧셈 실행
                break;
            case ISUB:
                // 정수 뺄셈 실행
                break;
            case INVOKEVIRTUAL:
                // 메소드 호출 실행
                break;
            // 수백 개의 다른 opcode 케이스들...
        }
    }
}
```

이 방식은 개념적으로 이해하기 쉽지만, 몇 가지 성능 문제가 있습니다:
- switch 문 내부에서 분기 예측 실패가 자주 발생
- 각 opcode 실행 사이에 추가적인 디스패치 오버헤드 발생
- 최신 CPU의 파이프라인과 캐시 최적화를 효과적으로 활용하지 못함

## 현대적 접근: 디스패치 테이블(Dispatch Table)

이러한 성능 문제를 해결하기 위해, 현대의 JVM 구현(예: HotSpot)은 디스패치 테이블을 사용합니다. 디스패치 테이블은 각 바이트코드 opcode에 해당하는 함수 포인터의 배열입니다:

```c
typedef void (*BytecodeHandler)(JVM*);

BytecodeHandler dispatchTable[256] = {
    handleNOP,
    handleACONST_NULL,
    handleICONST_M1,
    // 나머지 opcode들에 대한 핸들러...
};

void interpret(JVM* jvm) {
    while (jvm->hasMoreBytecode()) {
        uint8_t opcode = jvm->fetchNextOpcode();
        dispatchTable[opcode](jvm);
    }
}
```

이 접근 방식의 장점은:
- 분기 예측 실패 감소 (switch 문의 복잡한 분기 로직 제거)
- 간접 함수 호출 통해 더 효율적인 CPU 파이프라이닝
- 핫스팟 최적화에 더 친화적인 구조
- 코드 모듈화와 확장성 개선

## 더 발전된 기법들

최신 JVM은 디스패치 테이블 외에도 다양한 최적화 기법을 사용합니다:

**JIT 컴파일**: 핫스팟 코드를 식별하여 네이티브 코드로 컴파일함으로써 인터프리터 오버헤드 제거

## 결론

자바 인터프리터의 구현 방식은 단순한 switch 기반 접근법에서 고도로 최적화된 디스패치 테이블 메커니즘으로 진화해왔습니다. 이러한 발전은 JVM의 성능을 크게 향상시켰으며, 자바가 고성능 애플리케이션에서도 효과적으로 사용될 수 있게 했습니다.

컴파일러와 인터프리터의 이해는 프로그래밍 언어의 동작 방식을 깊이 이해하는 데 필수적이며, 자바의 사례는 성능 최적화를 위한 다양한 접근 방식을 보여주는 좋은 예입니다.
