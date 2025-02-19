---
layout: post
title: React Virtual DOM과 useState 타입 정의
author: 권혁준
categories: 기술블로그
banner:
  image: ""
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Virtual Dom", "useState"]
---

# React Virtual DOM과 useState 타입 정의

## Intro

React는 가상의 DOM을 실제 DOM과 비교하여, DOM 조작을 최소화하고 애플리케이션의 성능을 최적화시킬 수 있다. 그러다 생긴 의구심은 비교하는 타이밍이다. 이에 대해서는 컴포넌트 생명주기에 대해 알 필요가 있지만, 이번 내용에서는 가상 DOM과 실제 DOM의 비교 타이밍 및 그 과정의 순서에 대해 기술해보려고 한다.

추가적으로 Virtual DOM과 관련이 큰 `useState` 훅에 대해서도 간단하게 알아보자.

---

## 비교 타이밍

가상 DOM의 비교는 리액트 컴포넌트의 상태 `state`나 속성 `props`이 변경될 때 발생한다. 이는 다음과 같은 경우에 해당한다.

1. `setState()` 메서드 호출
2. 부모 컴포넌트로부터 새로운 `props` 전달
3. `forceUpdate()` 메서드 호출

### 프로세스

조금 더 자세히 알아보자.

1. 상태 변경: 데이터(`state` 또는 `props`)가 수정됨
2. 렌더링: 리액트는 컴포넌트를 다시 렌더링. 이때 새로운 가상 DOM 트리가 생성
3. 비교: 리액트는 이전 가상 DOM 트리와 새로 생성된 가상 DOM 트리를 비교. 이 과정을 "재조정"이라고 함.
4. 차이점 계산: 두 트리 간의 차이를 계산
5. 실제 DOM 업데이트: 계산된 차이점만 실제 DOM에 적용

따라서, **비교 타이밍은 데이터 수정 직후가 아니라, 데이터 수정으로 인해 리액트가 컴포넌트를 다시 렌더링한 후**입니다. 이 과정은 매우 빠르게 일어나며, 리액트의 효율적인 Diff 비교 알고리즘 덕분에 성능 저하 없이 수행된다.

![virtual DOM](https://raw.githubusercontent.com/fastcampus-fe-group7/TIL/refs/heads/main/React/image.png)

---

## 렌더링되는 것은 가상 DOM

"컴포넌트를 다시 렌더링"한다는 것은 실제로 화면에 그리는 것이 아니라, 새로운 가상 DOM 트리를 생성하는 것을 의미한다. 이 새로운 가상 DOM 트리는 이전 트리와 비교되고, 그 후에 필요한 변경사항만 실제 DOM에 적용된다.

이러한 과정을 통해 리액트는 아래와 같은 장점을 가질 수 있다.

1. 불필요한 DOM 조작을 최소화
2. 여러 변경사항을 일괄 처리하여 성능 최적화
3. 선언적 UI 프로그래밍 모델을 제공하면서도 효율적인 DOM 업데이트

이 과정은 매우 빠르게 일어나기 때문에, 사용자 입장에서는 거의 즉각적으로 UI가 업데이트되는 것처럼 보인다.

> React(선언형)
>
> > UI가 어떻게 보여야 하는지를 설명
> >
> > 상태(state)를 정의하고, 그 상태에 따라 UI가 어떻게 보일지 선언
>
> JavaScript(명령형)
>
> > 어떻게 UI를 변경할지 단계별로 지시
> >
> > DOM 요소를 직접 선택하고 조작

---

## 첫 번째 랜더링 때는 가상 DOM? 실제 DOM?

위의 내용을 통해, 리액트는 데이터가 변경될 때, 가상 DOM을 생성하고 기존의 실제 DOM과 비교하여 업데이트시킨다. 그렇다면 여기서 궁금한 점이 생길 수 있다. 처음 랜더링될 때는 가상 DOM이 생성되는가? 실제 DOM이 생성되는가?

이에 대한 답은, **가상 DOM**이다. 위에서 순서를 설명했지만, 다시 자세히 알아보자.

### 첫 번째 렌더링 과정

- 컴포넌트가 처음 렌더링될 때도 가상 DOM을 사용
- 렌더 단계에서 전체 가상 DOM 트리를 생성
- 이후 이 가상 DOM 트리를 기반으로 실제 DOM을 구성

### 일반 가상 DOM 생성 때와의 차이점

- 첫 렌더링 시에는 이전 가상 DOM 트리가 없으므로 비교(재조정) 과정이 없다.
- 대신, 생성된 전체 가상 DOM 트리를 실제 DOM으로 변환한다.

### 프로세스

- 가상 DOM 트리 생성
- 실제 DOM 요소 생성 및 삽입
- 생명주기 메서드 호출 (예: componentDidMount)

### 이후 업데이트

- 첫 렌더링 이후의 모든 업데이트는 이전에 설명한 비교 및 부분 업데이트 과정을 따른다.

결과적으로 첫 번째 렌더링에서도 가상 DOM을 사용하지만, 전체 DOM을 새로 구성한다는 점에서 차이가 있다.

---

위의 내용들을 요약하면 아래와 같다.

- React는 항상 가상 DOM을 먼저 다루고, 그 다음에 실제 DOM을 조작한다.
- 초기 렌더링에서는 전체 DOM을 구성하지만, 이후 업데이트에서는 변경된 부분만 효율적으로 수정한다.
- 비교 과정은 이전 가상 DOM과 새 가상 DOM 사이에서 이루어진다. 실제 DOM과 직접 비교하지 않는다.

이러한 방식으로 React는 DOM 조작을 최소화하고 애플리케이션의 성능을 최적화시킬 수 있다.

---

# useState 타입 정의

위에서 가상 DOM의 비교는 리액트 컴포넌트의 상태 `state`나 속성 `props`이 변경될 때 발생한다고 했다. 그러면 상태를 바꾸는 대표적은 React Hook인 `useState`에 대해 알아보자. 그 중에서도 내부적으로 어떻게 타입이 정의되어있는지 기술해보려고 한다.

---

## useState 타입 정의

먼저 React 프로젝트에서 `useState`의 타입 정의를 보면 아래와 같다.

1. React `useState` 코드

```jsx
const [count, setCount] = useState(0);
```

2. TypeScript로 작성된 `useState`의 타입 정의

```ts
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];
```

---

## 제네릭 타입 매개변수

자세한 타입 정의를 알아보기 전, 먼저 제네릭 타입 매개변수에 대해 알면 좋다.
클래스, 인터페이스, 메서드 등을 정의할 때 사용되는 타입 플레이스홀더로, 코드 재사용성 및 타입 안정성을 높이는 데 도움을 준다.

여기서 사용되는 알파벳은 특별한 의미가 없다. 즉 어떤 알파벳이든 사용할 수 있지만, 관례적으로 많이 사용되는 알파벳들이 있다. 관례를 따르면 코드의 가독성을 높이는 데 도움이 될 수 있다.

1. T: "Type"의 약자로, 가장 일반적으로 사용된다.
2. E: "Element"의 약자로, 컬렉션의 요소 타입을 나타낼 때 주로 사용된다.
3. K: "Key"의 약자로, 맵의 키 타입을 나타낼 때 사용된다.
4. V: "Value"의 약자로, 맵의 값 타입을 나타낼 때 사용된다.
5. N: "Number"의 약자로, 숫자 관련 타입에 사용된다.
6. S: "State"나 "String"의 약자로 사용된다.
7. A: "Any"의 약자로, 어떤 타입이든 될 수 있음을 나타낸다.

---

## 타입 정의 자세히 알아보기

그럼 다시 타입 정의에 대해 조금 더 자세히 뜯어보자.

`<S>`: 제네릭 타입 매개변수로 **상태의 타입**을 나타낸다.

`initialState: S | (() => S)`: **초기 상태의 값을 받는 매개변수**이다.

- `S` 타입의 값이거나 `S` 타입의 값을 반환하는 함수이다.

`[S, Dispatch<SetStateAction<S>>]`: **`useState` 함수의 반환 타입**을 나타낸다.

- 배열의 형태로 두 개의 요소를 반환한다.
  - `S`: 현재 상태 값
  - `Dispatch<SetStateAction<S>>`: 상태를 업데이트하는 함수

---

`Dispatch<SetStateAction<S>>` 또한 자세히 뜯어보자.

1. `Dispatch`: 상태를 업데이트하는 함수의 형태를 정의한다.

```ts
type Dispatch<A> = (value: A) => void;
```

- `<A>`: 제네릭 타입 매개변수로, 함수가 받을 인자의 타입을 나타낸다.
- `(value: A) => void`: `A` 타입의 값을 인자로 받고, 아무것도 반환하지 않는 함수를 의미한다.
- `'void'`는 함수가 명시적인 반환 값이 없다는 것을 나타낸다.

---

2. `<SetStateAction<S>>`: 상태 설정 액션(상태를 업데이트하는 방식을 나타내는 개념)의 형태를 정의한다.

```ts
type SetStateAction<S> = S | ((prevState: S) => S);
```

- `<S>`는 제네릭 타입 매개변수로, 상태의 타입을 나타냅니다.
- `S | ((prevState: S) => S)`: 다음 두 가지 중 하나임을 의미한다.
  - `S`: 새로운 상태 값
  - `(prevState: S) => S`: 이전 상태를 인자로 받아 새 상태를 반환하는 함수

위의 두 타입을 조합되면서, `useState`의 반환 타입인 `[S, Dispatch<SetStateAction<S>>]`이 된다.

`Dispatch<SetStateAction<S>>`

- 새로운 상태 값 (`S` 타입)
- 이전 상태를 받아 새 상태를 반환하는 함수 (`(prevState: S) => S` 형태)

---

### Dispatch와 SetStateAction

여기서 Dispatch와 SetStateAction의 차이가 모호할 수 있으나, React 상태 관리 시스템에서 서로 다른 역할을 하는 타입이다.

둘의 차이는 아래와 같다.

#### Dispatch

1. 상태를 업데이트하는 함수의 타입을 정의하는 역할을 한다.
2. 항상 함수 타입이다.
3. 인자를 받아 실제로 상태를 변경하는 동작을 수행한다.
4. 반환값이 없다. (void)

#### SetStateAction

1. 상태를 어떻게 업데이트할지 정의하는 액션의 타입 역할을 한다.
2. 값 또는 함수일 수 있다.
3. 실제 상태 변경을 수행하지 않고, 어떻게 변경할지를 정의한다.

즉, `SetStateAction`는 `Dispatch`가 상태를 업데이트하는데 있어, 어떻게 업데이트할지를 정의하는 역할을 한다. 또 `Dispatch`가 `useState`의 두 번째 요소의 타입으로 사용된다면, `SetStateAction`은 `Dispatch` 함수의 인자 타입으로 사용된다.

---

## 타입 정의를 통한 `useState` 동작 알아보기

위 타입 정의를 보면 `useState`가 다음과 같이 동작함을 알 수 있다.

1. `useState`는 초기 상태 값(`initialState`) 혹은 초기 상태를 생성하는 함수(`(() => S)`)를 받는다.

2. 현재 상태 값(`S`)과 그 상태를 업데이트할 수 있는 함수(`Dispatch<SetStateAction<S>>`), 두 가지 요소를 배열로 반환한다.

3. 상태 업데이트 함수(`Dispatch<SetStateAction<S>>]`)는 새 상태 값(`S`)을 직접 받거나, 이전 상태를 기반으로 새 상태를 계산하는 함수(`(prevState: S) => S`)를 받을 수 있다.

---

## 요약

다시 돌아가 React의 `useState`는 아래와 같이 구성되어 있다.

```jsx
const [count, setCount] = useState(0);
```

여기서 `useState`는 타입 정의를 보면, 다음과 같다는 것을 알 수 있다.

```ts
// 초기 상태의 값을 받는 매개변수
0 === `initialState: S | (() => S)`;
```

`useState`이 반환하는 타입 정의를 보면 아래와 같다.

```ts
// 초기 상태의 값을 받는 매개변수
[count, setCount] === [S, Dispatch<SetStateAction<S>>];
```

즉 `count`는 `S` 타입 정의를 갖는 현재 상태 값이고, `setCount`는 `Dispatch<SetStateAction<S>>`의 타입 정의를 갖는 상태를 업데이트하는 함수이다.

여기서 더 들어가보면,

```ts
type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```

`setState`는 `A` 제네틱 타입 매개변수를 통해 다양한 타입의 값을 처리할 수 있고, 반환값이 없다.

`<SetStateAction<S>>` 타입 정의를 통해 새로운 상태 값 또는 이전 상태를 인자로 받아 새 상태를 반환하는 함수를 인자로 받을 수 있다.
