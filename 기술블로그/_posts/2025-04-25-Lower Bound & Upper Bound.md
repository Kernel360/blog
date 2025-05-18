---
layout: post  
title: "Lower Bound & Upper Bound란?"
author: "조형준"
categories: "기술블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["알고리즘", "이진탐색", "Lower Bound & Upper Bound"]
---

# Lower Bound & Upper Bound란?
- Lower bound & Upper bound는 특정 경계 값을 찾는 알고리즘이다. 
- 이진 탐색을 기반으로 하기 때문에 정렬된 데이터에서만 사용할 수 있다.
- 시간 복잡도는 이진 탐색과 같은 O(log n)이다.

## Lower Bound
> 특정 값의 시작 위치를 찾는 알고리즘

- 찾고자 하는 값이 존재하면 그 값의 첫 번째 위치를 반환
- 찾고자 하는 값이 없으면 해당 값이 들어갈 수 있는 첫 번째 위치를 반환

- 아래 사진에서 3의 Lower bound는 3(index)이다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113194448.png)
### 동작 방식
- left: 배열 시작 위치
- right: 배열의 길이(마지막 index + 1)

1. 배열의 중간 값(mid)을 구한다.
2. 중간 값과 찾고자 하는 값을 비교한다.
	- 중간 값 < 목표 값
		- left = mid + 1
	- 중간 값 >= 목표 값
		- right = mid
			- 이 부분이 일반 이진 탐색과 다르다.
			- 일반 이진 탐색은 right = mid - 1
3. left >= right일 때까지 위 과정을 반복하면 right값이 Lower Bound가 된다.

- mid 값과 목표 값(3)을 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113194923.png)

- 목표 값(3)과 mid 값이 같으므로 right의 위치를 mid로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113195038.png)

- 다시 mid 값을 구하고 목표 값(3)과 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113195109.png)

- 목표 값(3)이 mid 값보다 작으므로 left의 위치를 mid + 1로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113195252.png)

- mid 값을 구하고 목표 값(3)과 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113195258.png)

- 목표 값(3)과 mid 값이 같으므로 right의 위치를 mid로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113195303.png)

- left가 right와 같으므로 순환을 종료하고 right 값이 Lower Bound값이 된다.
	- `if (left >= right) then lower bound = right`

### 구현 코드

``` java
public class BinarySearch {

    public static int lowerBound(int[] arr, int left, int right, int k) {
        while (left < right) {
            int mid = (left + right) / 2;
            if (arr[mid] < k) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return right;
    }

    public static void main(String[] args) {
        int[] arr = {1, 2, 4, 4, 5, 7, 9};
        int k = 4;

        int index = lowerBound(arr, 0, arr.length, k);
        System.out.println("Lower Bound Index: " + index);
    }
}

```

## Upper Bound
> 특정 값보다 큰 값이 처음 나타나는 위치를 찾는 알고리즘

- 찾고자 하는 값보다 큰 첫 번째 값을 반환
- 값이 없다면 값이 들어갈 마지막 위치를 반환

- 아래 사진에서 3의 Upper Bound는 6(index)이다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113210615.png)

### 동작 방식
- left: 배열 시작 위치
- right: 배열의 길이(마지막 index + 1)

1. 배열의 중간 값(mid)을 구한다.
2. 중간 값과 찾고자 하는 값을 비교한다.
	- 중간 값 <= 목표 값
		- left = mid + 1
	- 중간 값 > 목표 값
		- right = mid
			- 이 부분이 일반 이진 탐색과 다르다.
			- 일반 이진 탐색은 right = mid - 1
3. left >= right일 때까지 위 과정을 반복하면 right값이 Upper Bound가 된다.


- mid 값과 목표 값(3)을 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211038.png)

- 목표 값이 mid 값과 같으므로 left의 위치를 mid + 1로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211123.png)

- mid 값과 목표 값(3)을 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211149.png)

- 목표 값이 mid 값보다 작으므로 rigth를 mid로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211239.png)

- mid 값과 목표 값(3)을 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211251.png)

- 목표 값이 mid 값보다 작으므로 rigth를 mid로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211423.png)

- mid 값과 목표 값(3)을 비교한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211432.png)

- 목표 값이 mid 값보다 크므로 left를 mid + 1로 변경한다.
![](2025-05-25-이진탐색/Pasted%20image%2020250113211452.png)
- left가 right와 같으므로 순환을 종료하고 right 값이 Upper Bound값이 된다.
	- `if (left >= right) then upper bound = right`

### 구현 코드

``` java
public class BinarySearch {

    public static int upperBound(int[] arr, int left, int right, int k) {
        while (left < right) {
            int mid = (left + right) / 2;
            if (arr[mid] <= k) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return right;
    }

    public static void main(String[] args) {
        int[] arr = {1, 2, 4, 4, 5, 7, 9};
        int k = 4;

        int index = upperBound(arr, 0, arr.length, k);
        System.out.println("Upper Bound Index: " + index);
    }
}

```

### 참고
[참고 블로그](https://yoongrammer.tistory.com/105)
