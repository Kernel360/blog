---
layout: post
title: ArrayList vs LinkedList
author: 박수형
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [ArrayList, LinkedList, 기술세미나]
---

# 배열(Array)
- **동일한 타입**의 데이터를 **연속적인 메모리 공간**에 저장하는 자료구조
- 선언할 때 크기를 지정해야 하며, 한번 정해진 **크기를 변경할 수 없다.**
- 인덱스를 이용해 **상수시간(O(1))에 배열의 임의의 원소에 접근**할 수 있다.

타입\[ ] 배열이름 = new 타입\[ ] 형태로 선언한다.
``` 
int[] arr = new int[5];
```

# 리스트(List)
- 데이터를 순서대로 저장하는 자료 구조
- 선언 이후에도 크기를 동적으로 변경 가능
- 구현체로 `ArrayList`, `LinkedList` 등이 있다.
```
List<Integer> list = new ArrayList<>();
List<Character> list = new LinkedList<>();
```

---

# ArrayList
- 원소들을 **내부 배열**에 저장한다.
  - 조회 : index를 이용해 상수 시간에 임의의 원소에 접근 가능
  - 삽입 / 삭제 / 수정 자체는 상수 시간에 처리할 수 있지만, 그 과정에서 배열을 재할당 하거나 원소들을 이동시키는 경우 추가 작업이 필요하다.

![1](https://github.com/user-attachments/assets/a7fc2885-1e9b-4e7f-958d-a28e6b6f67e9)


- 이때 반복문으로 복사하는것이 아닌 System.arraycopy(src, srcPos, dest, destPos, length) 함수를 이용한다.
(src배열의 srcPos번째부터 length개의 원소를 dest배열의 destPos번째부터 length개의 원소에 복사)
-> 단순 반복문보다 2배이상 빠르다.


## ArrayList - get : (x축 : 원소 번호, y축 : 걸린시간(ms))

- 길이가 1,000인 ArrayList에서 get메소드로 각 원소에 접근하는데 걸린 시간

![2](https://github.com/user-attachments/assets/fbe55742-5fbb-42e8-b991-9df1d7bd5a60)




## ArrayList - add : (x축 : 삽입한 원소 개수, y축 : 걸린시간(ms))

`add` 작업 시 내부 배열이 가득 찰 경우 배열 재할당 작업이 발생(O(N))한다.
이때, 기존 배열의 1.5배 크기의 배열을 할당한다.
![3](https://github.com/user-attachments/assets/3ba12cdd-4eb0-4cb8-a5c7-4dd31cbf74c8)

- ArrayList에서 1~1000개의 원소를 맨 뒤에 add하는데 걸린 시간
![4](https://github.com/user-attachments/assets/9c12ed7c-e7b1-44af-a73c-acc57df9c34e)

배열이 가득 찼을때만, 새로운 배열을 할당 하므로, 거의 상수시간에 근접하게 원소를 삽입할 수 있다.
N개의 원소를 삽입하는데 O(N)에 근접한 시간복잡도를 가진다.

## ArrayList - 리스트 중간에서의 add : (x축 : 삽입한 원소 개수, y축 : 걸린시간(ms))

- 리스트 맨 앞에 원소를 `add`하는 경우 
![5](https://github.com/user-attachments/assets/4199807f-4e6f-455b-a1c9-f8d35a727db6)


- 리스트 중간에 원소를 `add`하는 경우
![6](https://github.com/user-attachments/assets/b41bea2c-e5e9-4750-bd03-2e1f9e1b13cf)



리스트 맨앞/중간에 원소를 `add`하는 경우, 각 `add`마다 `arraycopy`가 발생하므로, 성능이 좋지 않다.
N개의 원소를 추가하는데 O(N^2)의 시간복잡도를 가진다.

---
## LinkedList
- 내부적으로 double linked list 형태로 구현되어있다.
  - 각 노드들은 자신의 이전 노드와 다음 노드에 대한 정보만 가지고 있다.

![7](https://github.com/user-attachments/assets/94eada81-cf5b-4ef7-a229-550bf32690ac)


  - LinkedList는 전체 노드가 아니라 맨 처음 노드와 마지막 노드만 가지고있다.

![8](https://github.com/user-attachments/assets/446d1a4c-462e-4c39-a064-5738a7cf6e0b)


그렇기에 n번째 원소를 조회하기 위해서는 **맨 앞 또는 뒤에서부터 순차적으로 탐색(O(N))**을 해야 한다.
하지만, 삽입/삭제의 경우 **앞뒤 노드의 연결을 변경(O(1))**해주는것만으로 처리할 수 있기 때문에 매우 빠르게 처리할 수 있다.
![9](https://github.com/user-attachments/assets/89ff3a37-5316-4948-a78f-29bba3bb4dc0)

# LinkedList - get : (x축 : 원소 번호, y축 : 걸린시간(ms))

- 길이가 1,000인 LinkedList에서 get메소드로 각 원소에 접근하는데 걸린 시간
![10](https://github.com/user-attachments/assets/10f81e6a-2896-4eaa-bf93-c14146052aab)


맨 앞 또는 뒤에서부터 순차적으로 탐색하므로, 중앙의 원소에 가까울수록 조회 성능이 떨어진다.

## LinkedList - add : (x축 : 삽입한 원소 개수, y축 : 걸린시간(ms))

- 리스트 맨 앞에 원소를 `add`하는 경우
![11](https://github.com/user-attachments/assets/2d971a5d-797a-42fe-82a4-4c7d140150c1)

- 리스트 맨 뒤에 원소를 `add`하는 경우
![12](https://github.com/user-attachments/assets/2ebecd3f-5d5c-47cc-9b52-1ecb60e9b703)

-> 양 끝에서의 삽입/삭제는 탐색 없이 이루어지므로 N개의 원소를 삽입/삭제하는데 O(N)의 시간복잡도를 가진다.
  
- 리스트 중간에 원소를 `add`하는 경우
![13](https://github.com/user-attachments/assets/63faeffc-49d7-4895-b0d7-ec3a4ff50763)

-> 삽입/삭제 작업 자체는 상수시간내에 이루어지지만, 삽입/삭제위치까지 탐색하는데 O(N)이 필요하므로, N개의 원소를 삽입/삭제하는데 O(N^2)의 시간복잡도를 가진다.


---
# ArrayList vs LinkedList

- 인덱스로 원소에 접근하는 경우(get(idx))
  - ArrayList : O(1)
  - LinkedList : O(N)


- 맨 앞에 추가/삭제하는 경우(add(0, element), remove(0, element))
  - ArrayList : arraycopy로 전체 배열을 이동시켜야하므로 O(N)
  - LinkedList : 추가적인 노드 탐색 없이 노드 추가 가능 O(1)
  
- 중간에 추가/삭제하는 경우(add(idx, element), remove(idx, element))
  - ArrayList : 추가/삭제하는 위치(idx)부터 마지막 원소까지를 한칸씩 앞/뒤로 이동시켜야함(arraycopy)<br>
-> 이동시키는 원소의 개수에 따라 성능이 결정됨<br>
-> idx가 리스트 앞쪽에 있을수록 성능이 나쁘고,리스트 끝쪽에 있을수록 성능이 좋아짐
  - LinkedList : 삽입/삭제하는 위치(idx)까지 first/last 노드에서부터 탐색하여 idx번째 노드를 찾고 추가/삭제를 진행<br>
추가/삭제는 상수 시간에 처리되므로 idx번째 노드를 찾는데 까지 걸리는 시간에서 성능 차이가난다.<br>
노드 탐색은 양끝(first/last)에서 가능하므로, idx가 리스트 양 끝쪽에 가까울수록 성능이 좋고, 가운데에 가까울수록 성능이 나쁘다.

- 맨 뒤에 추가/삭제하는 경우(add(element), remove(element))
  - ArrayList
    - 배열이 가득 차지 않음 : 배열에 원소를 삽입/삭제 하는 것이므로 오버헤드가 적다.
    - 배열이 가득 참 : 새로운 배열(기존크기의1.5배)을 할당하고 기존 배열의 원소들을 복사(arraycopy)
  - LinkedList : 추가적인 노드 탐색 없이 노드 추가 가능 O(1) 하지만 배열에 비해 오버헤드(노드 연결 비용)가 존재한다.
-> 맨 뒤에 추가하는 경우 **간헐적으로 일어나는 arraycopy비용** vs **노드 생성 비용** 에 따라서 ArrayList와 LinkedList의 성능 차이가 난다.

## 간헐적으로 일어나는 arraycopy비용** vs **노드 생성 비용

![14](https://github.com/user-attachments/assets/d7002ad5-4828-4591-a90d-9a8fc264c5d0)


- ArrayList와 LinkedList에서 1,000~100,000개의 원소를 맨 뒤에 add하는데 걸린 시간
x축 : 원소 개수(1,000개 단위), y축 : 걸린시간(ms)

- 사실상 ArrayList와 LinkedList에 큰 차이가 나지 않는다.
  - 배열재할당이 일어난 직후에는 LinkedList가 성능이 좋다.
  - 하지만 배열에 원소를 추가하는 비용이 노드를 생성 연결 비용보다 작으므로 배열 재할당이 일어나지 않을때는 ArrayList가 성능이 좋다.
- ArrayList(pre-allocated)는 배열재할당이 일어나지 않도록 초기 크기를 충분히 크게 설정한것
-> **배열 재할당이 일어나지 않으므로 성능이 가장 좋음** 

