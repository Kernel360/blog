```
---
layout: post  
title: "List 중간 요소를 List 맨 뒤에 추가할 때 ArrayList 와 LinkedList 의 성능 비교"
author: "박준홍"
categories: "백엔드 기술블로그"
banner:
  background: "#2a9d8f"2
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Java", "ArrayList", "LinkedList"]
---
```

# List 중간 요소를 List 맨 뒤에 추가할 때 ArrayList 와 LinkedList 의 성능 비교

스프링 서버를 개발하는 중에 배열을 순회하면서 배열의 중간에 있는 요소 중에 조건에 따라서 그 요소를 맨 뒤로 보내야 하는 로직을 작성해야 할 필요가 있었다.

나는 이 구조를 개발하기 위해서 List 를 사용하기로 하였고 그 중에서 ArrayList 와 LinkedList 를 선택해야 하는 상황이었다.

이 선택에 대한 테스트가 필요하여 해당 테스트를 진행한다.



테스트는 총 2가지를 진행할 것이다.

1. 요소를 순차적으로 뒤에 추가할 경우
2. 배열의 특정 중간 요소를 삭제하는 경우
3. 배열의 특정 중간 요소를 빼서 배열의 맨 뒤에 추가하는 경우



주로 작동할 동작은 배열의 중간 요소를(첫 번째 인덱스 에서 마지막 인덱스 사이의 요소) 마지막 인덱스의 다음으로 붙이는 상황이다.



이 3가지의 방법을 횟수 별로 비교하여 성능을 비교하고자 한다.

1. 10,000 건
2. 100,000 건
3. 1,000,000 건
4. 50,000,000 건
5. 70,000,000 건 (100,000,000 건은 컴퓨터 성능 이슈로 인해서  테스트 하지 못하였다.)

그리고 중복된 케이스가 위치한 곳에 따라서 성능이 달라질 것이기 때문에 3가지의 경우로 테스트를 나눠주었다.



테스트 환경

- OS : Apple M1 Pro
- 메모리 : 16GB
- JDK : OpenJDK 64-Bit Server VM Corretto-17.0.12.7.1 (build 17.0.12+7-LTS, mixed mode, sharing)



## 테스트 케이스 1. 요소를 순차적으로 뒤에 추가할 경우

### 코드

```java
public class LastAdd {

    public static void main(String[] args) {
        doTest(10_000);
        doTest(100_000);
        doTest(1_000_000);
        doTest(10_000_000);
        doTest(50_000_000);
        doTest(70_000_000);
    }

    public static void doTest(int repeatCount) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();

        System.out.println("========== " + repeatCount + "회 반복 테스트 시작 ==========");

        // ArrayList 추가 테스트
        testArrayList(arrayList, repeatCount);

        // LinkedList 추가 테스트
        testLinkedList(linkedList, repeatCount);

        System.out.println("========== " + repeatCount + "회 반복 테스트 끝 ==========");
    }

    public static void testArrayList(ArrayList<Integer> arrayList, int repeatCount) {
        // ArrayList 추가 테스트
        long start = System.currentTimeMillis();
        addArrayList(arrayList, repeatCount);
        long end = System.currentTimeMillis();
        System.out.println("ArrayList 추가 시간: " + (end - start) + "ms");
    }

    public static void testLinkedList(LinkedList<Integer> linkedList, int repeatCount) {
        // LinkedList 추가 테스트
        long start = System.currentTimeMillis();
        addLinkedList(linkedList, repeatCount);
        long end = System.currentTimeMillis();
        System.out.println("LinkedList 추가 시간: " + (end - start) + "ms");
    }

    public static boolean addArrayList(ArrayList<Integer> list, int size) {
        for (int i = 0; i < size; i++) {
            list.add(i);
        }
        return true;
    }

    public static boolean addLinkedList(LinkedList<Integer> list, int size) {
        for (int i = 0; i < size; i++) {
            list.add(i);
        }
        return true;
    }
}
```



### 실행 횟수 별 비교

| 반복 횟수  | ArrayList 추가 시간 | LinkedList 추가 시간 |
| ---------- | ------------------- | -------------------- |
| 10,000     | 0ms                 | 0ms                  |
| 100,000    | 2ms                 | 4ms                  |
| 1,000,000  | 15ms                | 105ms                |
| 10,000,000 | 203ms               | 993ms                |
| 50,000,000 | 938ms               | 5064ms               |
| 70,000,000 | 1635ms              | 11209ms              |



### 결론

순차적으로 배열의 맨 뒤에 추가할 경우 ArrayList 가 훨씬 빠르다.

(50,000,000 건 일 경우 5배 빠르다. 70,000,000 건 일 경우 6배 빠르다.)



## 테스트 케이스 2. 배열의 특정 중간 요소를 삭제하는 경우

### 코드

```java
public class RandomDelete {

    public static void main(String[] args) {
        doTest(10_000);
        doTest(100_000);
        doTest(1_000_000);
        doTest(10_000_000);
//        doTest(50_000_000);
//        doTest(70_000_000);
    }

    private static void doTest(int repeatCount) {
        List<Integer> arrayList = makeList(new ArrayList<>(), repeatCount);
        List<Integer> linkedList = makeList(new LinkedList<>(), repeatCount);

        System.out.println("========== " + repeatCount + "회 반복 테스트 시작 ==========");

        // ArrayList 추가 테스트
        testRandomDelete(arrayList);

        // LinkedList 추가 테스트
        testRandomDelete(linkedList);

        System.out.println("========== " + repeatCount + "회 반복 테스트 끝 ==========");
    }

    private static void testRandomDelete(List<Integer> list) {
        long start = System.currentTimeMillis();
        deleteElement(list);
        long end = System.currentTimeMillis();
        System.out.println("ArrayList 추가 시간: " + (end - start) + "ms");
    }

    private static List<Integer> makeList(List<Integer> list, int size) {
        for (int i = 0; i < size; i++) {
            list.add(i);
        }
        return list;
    }

    private static void deleteElement(List<Integer> list) {
        while (!list.isEmpty()) {
            // 중간 요소 삭제
            int randomIndex = list.size() / 2;

            list.remove(randomIndex);
        }
    }
}

```



### 실행 횟수 별 비교

| 반복 횟수 | ArrayList 삭제 시간 | LinkedList 삭제 시간 |
| --------- | ------------------- | -------------------- |
| 10,000    | 4ms                 | 47ms                 |
| 100,000   | 238ms               | 4008ms               |
| 1,000,000 | 24432ms             | 485413ms             |

- 1,000,000 건 이후로는 너무 많은 시간이 소요되어서 중단하였다.



## 테스트 케이스 3. 배열의 특정 중간 요소를 빼서 배열의 맨 뒤에 추가하는 경우

### 코드

```java
public class ExtractMiddleAndAddLast {

    public static void main(String[] args) {
        doTest(10_000);
        doTest(100_000);
        doTest(1_000_000);
//        doTest(10_000_000);
//        doTest(50_000_000);
//        doTest(70_000_000);
    }

    private static void doTest(int repeatCount) {
        List<Integer> arrayList = makeList(new ArrayList<>(), repeatCount);
        List<Integer> linkedList = makeList(new LinkedList<>(), repeatCount);

        System.out.println("========== " + repeatCount + "회 반복 테스트 시작 ==========");

        // ArrayList 테스트
        testExtractMiddleAndAddLast(arrayList, repeatCount);

        // LinkedList 테스트
        testExtractMiddleAndAddLast(linkedList, repeatCount);

        System.out.println("========== " + repeatCount + "회 반복 테스트 끝 ==========");
    }

    private static void testExtractMiddleAndAddLast(List<Integer> list, int repeatCount) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < repeatCount; i++) {
            // 중간 요소 삭제
            int randomIndex = list.size() / 2;

            list.add(list.remove(randomIndex));
        }
        long end = System.currentTimeMillis();

        String listType = list instanceof ArrayList ? "ArrayList" : "LinkedList";
        System.out.println(listType + " 추가 시간: " + (end - start) + "ms");
    }

    private static List<Integer> makeList(List<Integer> list, int size) {
        for (int i = 0; i < size; i++) {
            list.add(i);
        }
        return list;
    }
}
```



### 실행 횟수 별 비교

| 반복 횟수 | ArrayList 추가 시간 | LinkedList 추가 시간 |
| --------- | ------------------- | -------------------- |
| 10,000    | 7ms                 | 89ms                 |
| 100,000   | 483ms               | 7954ms               |
| 1,000,000 | 49932ms             | 881514ms             |



## 번외 테스트 케이스. 배열의 맨 앞에 요소를 추가하는 경우

### 코드

```java
package checkList;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class FirstAdd {
    public static void main(String[] args) {
        doTest(10_000);
        doTest(100_000);
        doTest(1_000_000);
        doTest(10_000_000);
//        doTest(50_000_000);
//        doTest(70_000_000);
    }

    private static void doTest(int repeatCount) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();

        System.out.println("========== " + repeatCount + "회 반복 테스트 시작 ==========");

        // ArrayList 추가 테스트
        addFirstList(arrayList, repeatCount);

        // LinkedList 추가 테스트
        addFirstList(linkedList, repeatCount);

        System.out.println("========== " + repeatCount + "회 반복 테스트 끝 ==========");
    }


    private static void addFirstList(List<Integer> list, int repeatCount) {
        if (list instanceof ArrayList) {
            addFirstArrayList((ArrayList<Integer>) list, repeatCount);
        } else if (list instanceof LinkedList) {
            addFirstLinkedList((LinkedList<Integer>) list, repeatCount);
        }
    }

    private static void addFirstArrayList(ArrayList<Integer> list, int repeatCount) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < repeatCount; i++) {
            list.add(0, i);
        }
        long end = System.currentTimeMillis();
        System.out.println("ArrayList 추가 시간: " + (end - start) + "ms");
    }

    private static void addFirstLinkedList(LinkedList<Integer> list, int repeatCount) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < repeatCount; i++) {
            list.addFirst(i);
        }
        long end = System.currentTimeMillis();
        System.out.println("LinkedList 추가 시간: " + (end - start) + "ms");
    }
}
```



### 실행 횟수 별 비교

| 반복 횟수 | ArrayList 추가 시간 | LinkedList 추가 시간 |
| --------- | ------------------- | -------------------- |
| 10,000    | 5ms                 | 2ms                  |
| 100,000   | 477ms               | 3ms                  |
| 1,000,000 | 201475ms            | 23ms                 |



## 결론

- LinkedList 는 첫 번째 인덱스에 요소를 추가하는 상황에서만 ArrayList 보다 성능 우위에 있다.
