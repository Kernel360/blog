---
layout: post  
title: "중복 여부 판단을 위한 Set 과 반복문의 성능 비교"
author: "박준홍"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#2a9d8f"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Java", "Set", "For Loop"]

---


# 중복 여부 판단을 위한 Set 과 반복문의 성능 비교

리스트로 들어온 요소들의 중복여부를 판단하기 위한 방법을 생각했을 때, 바로 떠오르는 것은 set 이었다.

여기에서 궁금한 것은 중복을 판단하기 위한 다른 방법이 있는지와 각각의 방법의 성능을 알고 싶었다.

크게 3가지의 방법이 존재한다.

1. List 를 Set 으로 변경하여 List 와 Set 의 길이를 비교하는 방법
2. 반복문으로 순회하면서 Set 에 순회 요소를 추가하면서 추가가 되지 않는다면 리턴하는 방법
3. Stream API 를 통한 중복 제거 (distinct 사용)

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



## 테스트 케이스 1. 마지막에 중복된 케이스가 있는 경우

### 코드

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.stream.IntStream;

public class CheckPerformance_last {

    public static void main(String[] args) {
        int listSize = 70_000_000; // 반복 횟수 설정
        List<Integer> list = generateList(listSize, true);

        // Set 방식
        long start = System.currentTimeMillis();
        boolean hasDuplicatesSet = hasDuplicatesUsingSet(list);
        long end = System.currentTimeMillis();
        System.out.println("Set 방식: " + hasDuplicatesSet + ", 시간: " + (end - start) + "ms");

        // Set + 반복문 방식
        start = System.currentTimeMillis();
        boolean hasDuplicatesLoop = hasDuplicatesUsingLoop(list, list.size());
        end = System.currentTimeMillis();
        System.out.println("반복문 방식: " + hasDuplicatesLoop + ", 시간: " + (end - start) + "ms");

        // Stream API 방식
        start = System.currentTimeMillis();
        boolean hasDuplicatesStream = hasDuplicatesUsingStream(list);
        end = System.currentTimeMillis();
        System.out.println("Stream API 방식: " + hasDuplicatesStream + ", 시간: " + (end - start) + "ms");
    }

    // Set 방식
    public static boolean hasDuplicatesUsingSet(List<?> list) {
        return list.size() != new HashSet<>(list).size();
    }

    // 반복문 방식
    public static boolean hasDuplicatesUsingLoop(List<?> list, int lSize) {
        Set<Object> seen = new HashSet<>(lSize);
        for (Object item : list) {
            if (!seen.add(item)) {
                return true; // 중복 발견
            }
        }
        return false; // 중복 없음
    }

    // stream api 방식
    public static boolean hasDuplicatesUsingStream(List<?> list) {
        return list.stream().distinct().count() != list.size();
    }

    // 테스트 데이터 생성
    public static List<Integer> generateList(int size, boolean hasDuplicates) {
        List<Integer> list = new ArrayList<>();
        IntStream.range(0, size).forEach(list::add);
        if (hasDuplicates) {
            list.add(0); // 중복 요소 추가
        }
        return list;
    }
}

```

### 실행 횟수 별 비교

| 반복 횟수 \ 형식 | Set 으로 변환 | 반복문 방식(Set으로 추가) | Stream API       |
| ---------------- | ------------- | ------------------------- | ---------------- |
| 10,000           | 1ms           | 2ms                       | 1ms              |
| 100,000          | 6ms           | 5ms                       | 5ms              |
| 1,000,000        | 53ms          | 30ms                      | 31ms             |
| 10,000,000       | 366ms         | 313ms                     | 322ms            |
| 50,000,000       | 1230ms        | 1790ms                    | 2498ms           |
| 70,000,000       | 3024ms        | 2277ms                    | OutOfMemoryError |



## 테스트 케이스 2. 첫 번째에 중복된 케이스가 있는 경우

### 코드

```java
// 테스트 데이터 생성
public static List<Integer> generateListFirst(int size, boolean hasDuplicates) {
    List<Integer> list = new ArrayList<>();

    if (hasDuplicates) {
        list.add(0); // 중복 요소 추가
    }

    IntStream.range(0, size).forEach(list::add);
    return list;
}
```

- 나머지 코드는 테스트 케이스 1과 동일하다.



### 실행 횟수 별 비교

| 반복 횟수 \ 형식 | Set 으로 변환 | 반복문 방식(Set으로 추가) | Stream API |
| ---------------- | ------------- | ------------------------- | ---------- |
| 10,000           | 1ms           | 0ms                       | 3ms        |
| 100,000          | 5ms           | 0ms                       | 5ms        |
| 1,000,000        | 57ms          | 1ms                       | 30ms       |
| 10,000,000       | 286ms         | 2ms                       | 247ms      |
| 50,000,000       | 1234ms        | 77ms                      | 3210ms     |
| 70,000,000       | 2951ms        | 495ms                     | 4051ms     |



## 테스트 케이스 3. 배열의 중간에 중복된 케이스가 있는 경우

### 코드

```java
// 테스트 데이터 생성
public static List<Integer> generateListFirst(int size, boolean hasDuplicates) {
    List<Integer> list = new ArrayList<>();


    IntStream.range(0, size).forEach(list::add);

    if (hasDuplicates) {
        int middleIndex = size / 2; // 배열 중간 인덱스
        list.set(middleIndex, 0); // 중복 요소 추가
    }

    return list;
}
```

- 나머지 코드는 테스트 케이스 1과 동일하다.



### 실행 횟수 별 비교

| 반복 횟수 \ 형식 | Set 으로 변환 | 반복문 방식(Set으로 추가) | Stream API |
| ---------------- | ------------- | ------------------------- | ---------- |
| 10,000           | 2ms           | 0ms                       | 1ms        |
| 100,000          | 6ms           | 3ms                       | 5ms        |
| 1,000,000        | 53ms          | 16ms                      | 42ms       |
| 10,000,000       | 343ms         | 163ms                     | 351ms      |
| 50,000,000       | 1423ms        | 829ms                     | 2098ms     |
| 70,000,000       | 3820ms        | 1276ms                    | 4949ms     |



## 결론

처음에는 Set 으로 중복 체크를 하는 것이 어느 정도의 성능이 나올 것이라고 생각했는데, for 문의 성능이 여러모로 좋다.
