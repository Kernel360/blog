---
layout: post  
title: 시간 복잡도
author: 문찬욱
banner:
  image: https://raw.githubusercontent.com/Kernel360/blog-image/main/2023/1116/thumb.jpg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [알고리즘, 시간 복잡도]
---

안녕하세요. 저는 이번 Kernel360의 기술세미나에서 시간 복잡도라는 주제로 발표하게 문찬욱입니다.

아마 다들 e2e하면서 DTO로 데이터를 이렇게도 바꿔보고 저렇게도 바꿔보셨을 것 같은데요.

저는 이 과정에서 **시간 복잡도**를 개선할 수 있다면

이 후, 서비스 성능면에서 좋은 영향을 주지 않을까라는 생각에 이 주제를 선정했습니다.

시간 복잡도에 대해 이야기 하기 전에 알고리즘에 대해 간단히 설명드리겠습니다.

<br>

## 1. 알고리즘
어떤 문제나 목적을 달성하기 위해 거쳐야 하는 여러 과정들을 의미합니다.

이 과정들은 다양하고, 상황에 따라 알고리즘은 모두 다릅니다.

따라서 상황에 맞게 성능이 좋은 알고리즘을 선택하여 사용해야 합니다.

### 알고리즘 성능
일반적으로 알고리즘의 성능에는 알고리즘 실행 동안 사용되는 메모리와 관련된 **공간 복잡도**,

알고리즘 실행 시간과 관련된 **시간 복잡도**가 있습니다.

하지만 메모리의 발전으로 인해 시간 복잡도에 비해 공간 복잡도의 중요성이 상대적으로 낮아졌습니다.

<br>

## 2. 시간 복잡도
입력 데이터 크기에 대한 실행 시간의 증가율을 나타낸 것입니다.

알고리즘 실행시간에 중요하지 않은 상수와 항을 제외하고 가장 차수가 높은 항을 고려한 점근적 표기법을 사용합니다.

### 점근적 표기법
![1](https://github.com/Kernel360/blog-image/assets/97713997/97938a75-1dd1-4b08-aa4e-31f4d1351245)

위 예시를 보시면 알고리즘 실행 시간이 2n^2+3n+1이라면 가장 높은 차수 항인 2n^2을 고려하는 것입니다.

이 점근적 표기법에는 최선, 평균, 최악을 고려하는 표기법들이 있습니다.

- **최선의 경우**: Big-Ω 표기법이라 하며 알고리즘이 가장 적게 걸린 시간을 나타냅니다.
- **평균의 경우**: Big-θ 표기법이라 하며 알고리즘이 평균적으로 걸린 시간을 나타냅니다.
- **최악의 경우**: Big-O 표기법이라 하며 알고리즘이 가장 오래 걸린 시간을 나타냅니다.

시간 복잡도는 그 중 최악을 고려하는 Big-O 표기법를 자주 사용합니다.

### Big-O 표기법
왜 최악을 고려한 Big-O 표기법을 자주 사용할까요?

Big-O가 다른 표기법에 비해 다음과 같은 이점이 있기 때문입니다.

- 최악의 경우를 알면 여러 알고리즘 중 어디가 문제인지 예측하기 쉽습니다.
- 알고리즘의 성능이 항상 이 값 이하임을 보장할 수 있습니다.
- 최악의 경우에 대한 시간 복잡도를 비교함으로써 다양한 알고리즘을 쉽게 비교할 수 있습니다.

![ww](https://github.com/Kernel360/blog-image/assets/97713997/7e2ed745-0540-4778-9da8-b5641c375d8d)

위 그림은 입력 데이터 크기에 대한 실행시간을 나타낸 그래프입니다.

그래프에 나와있는 Big-O는 **O(1)**, **O(log n)**, **O(n)**, **O(n log n)**, **O(n^2)**, **O(2^n)**, **O(n!)** 가 있습니다.

물론, 그래프에 나와있는 것 이외에도 다양한 Big-O가 있을 수 있습니다.

그래프를 보시면 **O(1)과 O(log n)은 excellent과 good**, **O(n)은 fair**, **O(n log n)은 bad**, **나머지는 horrible**에 있습니다.

이 그래프에서 확인할 수 있는 것은 시간 복잡도가 높을수록 입력 데이터 크기에 대한 실행시간이 엄청나게 늘어난다는 것입니다.

즉, 저희는 처리하는 데이터가 많다고 할 때, horrible한 알고리즘이 있다면 이쪽을 조금만 더 개선해도 성능면에서 큰 효과가 기대할 수 있게 됩니다.

간단하게 O(1), O(log n), O(n) 이 3가지만 살펴보겠습니다.

### O(1)
입력 데이터의 크기에 상관없이 언제나 일정한 시간이 걸리는 알고리즘입니다.

```java
public boolean constantTime(int[] n) {
    return n[0] == 0;
}
```

위 코드를 보시면 입력인 int 배열의 크기에 상관없이 배열의 첫번째 값이 0인지 확인하고 끝나기 때문에 O(1)입니다.

### O(log n)
한번 돌 때마다 입력 데이터 크기가 절반이 되는 알고리즘입니다.

대표적으로 이진 탐색이 있습니다.

```java
public int binarySearch(int key, int[] arr, int start, int end) {
    if (start > end) {
        retrun -1
    }
    int mid = (start + end) / 2;
    if (arr[mid] == key) {
        return mid;
    } else if (arr[mid] > key) {
        return binarySearch(key, arr, start, mid-1);
    } else {
        return binarySearch(key, arr, mid-1, end);
    }
}
```

위 예제 코드는 재귀로 구현한 이진 탐색입니다.

재귀를 한번 할때마다 mid라는 변수를 이용해 입력인 int 배열을 절반씩 줄이기 때문에 O(logn)입니다.


코드만 봐서는 이해하기 힘드실 수 도 있을거 같아서 이진 탐색과 순차 탐색을 비교할겸 gif 하나를 가져왔습니다.

<img src="https://github.com/Kernel360/blog-image/assets/97713997/593124cd-9ece-4011-a3d2-5f267eceb2f4"  width="200%" height="200%"/>

두 알고리즘 모두 숫자 37을 찾는 알고리즘이고 위에가 이진 탐색, 밑에가 순차 탐색입니다.

이진 탐색이 순차 탐색보다 steps가 많이 적음을 확인할 수 있습니다.

### O(n)
입력데이터의 크기에 비례하여 처리시간이 늘어나는 알고리즘입니다.

```java
public void linearTime(int[] n) {
    for (int i = 0; i < n.length; i++) {
        System.out.println(i);
    }
}
```

위 코드는 일반적인 for문인데요, 입력 데이터의 크기만큼 출력해주기 때문에 O(n)입니다.

아까 보신 순차 탐색이 이에 해당합니다.

#### 그렇다면 알고리즘 속에 알고리즘이 있다면 Big-O는 어떻게 될까요?

두 Big-O를 곱하시면 됩니다!

```java
public void nestedLoop(int[] n) {
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            System.out.println("Kernel360 Let's Go!!");
        }
    }
}
```

예를 들어 위와 같이 같은 입력 데이터의 크기를 갖는 이중 for문이 있다고 한다면, 시간복잡도는 n과 n을 곱해서 O(n^2)이 되는 것입니다.

<br>

## 3. 마무리
시간 복잡도에 대해 알고 알고리즘을 비교할 수 있게 된다면,

이 gif 파일처럼 같은 문제를 푸는 것이여도 시간 복잡도가 더 나은 알고리즘을 사용해서 더 나은 성능을 낼 수 있습니다.

<br>

## 4. 참고 블로그
[[알고리즘] Time Complexity (시간 복잡도)](https://hanamon.kr/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-time-complexity-%EC%8B%9C%EA%B0%84-%EB%B3%B5%EC%9E%A1%EB%8F%84/)

[알고리즘의 시간 복잡도와 Big-O 쉽게 이해하기](https://blog.chulgil.me/algorithm/)

[시간복잡도 Big-O(빅오)란?](https://ssdragon.tistory.com/100)
