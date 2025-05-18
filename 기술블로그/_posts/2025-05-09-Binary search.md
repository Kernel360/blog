---
layout: post  
title: "이진 탐색 알고리즘"
author: "조형준"
categories: "기술블로그"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: ["알고리즘", "이진탐색"]
---


# 이진 탐색이란?
> 이진 탐색이란?
> '정렬된 배열(리스트)'에서 탐색 범위를 절반으로 줄여가며 특정 값을 찾는 알고리즘이다.
> 탐색할 때마다 탐색 범위가 절반으로 줄어들기 때문에 '탐색해야 할 데이터의 양이 많을수록 유리'하다.
# 동작 방식
![[이진 탐색.gif]]

- 동작 방식
	1. 정렬된 배열의 중간 값을 찾아서 목표 값과 비교한다.
	2. 목표 값보다 작다면 왼쪽 절반을 탐색하고 크다면 오른쪽 절반을 탐색한다.
	3. 목표 값과 중간 값이 같을 때까지 위 과정을 반복한다.
# 구현 방법
## 반복문 이용한 방법
1. 정렬된 배열의 첫 번째 인덱스를 `low`로 선언하고 마지막 인덱스를 `high`로 선언한다.
2. `while`문을 이용하여 `high`가 `low`보다 크거나 같을 때까지 아래 탐색을 실행한다.
3. 중간 인덱스(`mid`)와 중간 값(`midValue`)를 구한다.
4. **중간 값 > 목표 값**이라면 왼쪽 배열을 탐색한다.
5. **중간 값 < 목표 값**이라면 오른쪽 배열을 탐색한다.
6. **중간 값 == 목표 값**이라면 목표 값을 반환한다.
``` java
public class BinarySearch {
    public static int binarySearchIterative(int[] arr, int target) {
        int low = 0;
        int high = arr.length - 1;

		// 마지막 인덱스가 첫 번째 인덱스보다 같거나 작을 때 탐색 실행
        while (low <= high) {
            int mid = (low + high) / 2;  // 중간 인덱스 계산
            int midValue = arr[mid];     // 중간 값

			// 목표값을 찾은 경우 인덱스 반환
            if (midValue == target) {
                return mid;
            // 목표값이 더 크면 오른쪽 절반 탐색
            } else if (midValue < target) {
                low = mid + 1;
	        // 목표값이 더 작으면 왼쪽 절반 탐색
            } else {
                high = mid - 1;
			}
        }

        return -1;  // 목표값을 찾지 못한 경우
    }

    public static void main(String[] args) {
        int[] arr = {2, 4, 7, 10, 13, 18, 20};
        int target = 10;
        int index = binarySearchIterative(arr, target);

        System.out.println("Iterative: " + index);  // 출력: Iterative: 3
    }
}

```
## 재귀 함수 이용한 방법
재귀 함수에는 정렬된 배열, 목표 값, 첫 번째 인덱스, 마지막 인덱스가 들어간다.
1. 첫 번째 인덱스(`low`)가 마지막 인덱스(`high`)보다 작거나 같을 때 탐색을 실행한다. 
2. **중간 값 > 목표 값**이라면 왼쪽 배열을 탐색한다.
3. **중간 값 < 목표 값**이라면 오른쪽 배열을 탐색한다.
4. **중간 값 == 목표 값**이라면 목표 값을 반환한다.
``` java
public class BinarySearch {
    public static int binarySearchRecursive(int[] arr, int target, int low, int high) {
	    // 목표값이 리스트에 없는 경우 탈출
        if (low > high)
            return -1;
        
		
        int mid = (low + high) / 2; // 중간 인덱스
        int midValue = arr[mid]; // 중간 값

		// 목표값을 찾은 경우 인덱스 반환
        if (midValue == target) {
            return mid;
        // 오른쪽 절반 탐색
        } else if (midValue < target) {
            return binarySearchRecursive(arr, target, mid + 1, high);
        // 왼쪽 절반 탐색
        } else {
            return binarySearchRecursive(arr, target, low, mid - 1);
        }
    }

    public static void main(String[] args) {
        int[] arr = {2, 4, 7, 10, 13, 18, 20};
        int target = 10;
        int index = binarySearchRecursive(arr, target, 0, arr.length - 1);

        System.out.println("Recursive: " + index);  // 출력: Recursive: 3
    }
}

```

## 이진 탐색 문자열 탐색 - Arrays.binarySearch()
정렬된 문자열로 이루어진 배열은 내장된 이진 탐색 메서드인 Arrays.binarySearch()로 이진 탐색을 수행할 수 있다.

- 매개 변수
	- `Object[]`: 탐색하고자 하는 배열
	- `Object`: 찾고자 하는 값
- 반환 값
	- 목표 값이 있을 경우 목표 값의 인덱스 반환
	- 목표 값이 없을 경우 예상 위치의 음수 인덱스-1 반환(ex. 인덱스 6에 있을 거라 예상된다면 -7 반환)

``` java
public static int binarySearch(Object[] a, Object key)
```

### 예시 코드

``` java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] arr = {2, 4, 7, 10, 13, 18, 20};
        
        // 정렬된 배열에서 이진 탐색 수행
        int target = 10;
        int index = Arrays.binarySearch(arr, target);
        
	    // 출력: Index of 10: 3
        System.out.println("Index of " + target + ": " + index);

        // 배열에 존재하지 않는 값 탐색
        int missingTarget = 15;
        int missingIndex = Arrays.binarySearch(arr, missingTarget);
    
	    // 출력: Index of 15: -6
        System.out.println("Index of " + missingTarget + ": " + missingIndex);  
    }
}

```
# 문제 유형
## 특정 조건을 만족하는 최대/최소 값 찾기
- 정답이 특정 구간 안에 있을 때, 이진 탐색을 통해 그 구간을 좁혀가며 최대 또는 최소 값을 찾는 문제입니다.
- 예시
	- 최대한 많은 수의 작업을 할 수 있는 작업 스케줄링 문제, 최소 비용으로 특정 작업을 완료하는 문제.
## 경계 찾기 (Lower Bound / Upper Bound)
- Lower Bound
	- 특정 값 이상이 처음으로 나타나는 위치를 찾는 문제.
- Upper Bound
	- 특정 값보다 큰 값이 처음으로 나타나는 위치를 찾는 문제.
- 예시
	- [1, 2, 2, 2, 3, 4]에서 값 2의 시작과 끝 위치 찾기.
# 참고
https://adjh54.tistory.com/187
