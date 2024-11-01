---
layout: post  
title: TypeScript로 안전하게 API 호출하기
author: 윤예진
categories: 프론트엔드 기술블로그
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [JavaScript, module_bundler, webpack, vite]
---

프로젝트에서 백엔드와의 통신을 위해 fetch API를 활용하여 코드를 구현했습니다. GET API 요청을 위한 코드는 아래 사진과 같이 작성했습니다.

![image](https://github.com/user-attachments/assets/e91d379a-0c1c-4852-b957-adf8dc8fd368)

GET 메서드를 사용하는 API 요청에서는 여러 가지 공통적인 코드들이 필요했습니다. method 지정, headers 설정, credentials 관련 코드 작성, 응답이 없을 때의 오류 처리 로직, 그리고 받아온 응답을 JSON 형태로 변환하는 코드 등이 이에 해당합니다. 이러한 코드들은 대부분의 GET 요청에서 반복적으로 사용되었습니다. 코드의 중복을 최소화하고 재사용성을 높이기 위해, 이러한 공통 요소들을 하나로 모아 restClient라는 객체를 생성하게 되었습니다. 

![image](https://github.com/user-attachments/assets/440958ef-0a9b-4d12-b6a8-f8c29722d21e)

이러한 방식으로 코드를 구성하면서 한 가지 문제점이 발생했습니다. 그것은 바로 API 호출 후 반환되는 data의 타입이 자동으로 any로 지정된다는 점입니다. TypeScript에서 any 타입의 사용은 여러 가지 문제를 야기할 수 있습니다. 첫째, 타입 추론이 불가능해져 개발 과정에서 자동 완성이나 타입 관련 오류를 미리 잡아내기 어려워집니다. 둘째, 타입 안정성이 크게 저하되어 런타임 에러의 위험이 증가합니다. 

API의 응답은 엔드포인트마다 다른 상황에서 각 API 응답에 대한 data의 타입을 어떻게 정확하게 명시해줄 수 있을까요? 

저는 디렉터님과의 페어 프로그래밍을 통해 해답을 찾을 수 있었는데, 바로 제네릭 타입을 사용하는 것이었습니다. 

제네릭 타입을 사용하면 함수나 클래스를 정의할 때 타입을 매개변수화할 수 있습니다. 이를 통해 코드의 재사용성을 높이면서도 타입 안정성을 유지할 수 있습니다. restClient 객체의 메서드에 제네릭 타입을 적용하면, API 호출 시 반환되는 데이터의 타입을 동적으로 지정할 수 있게 됩니다. 제네릭 타입을 활용해 `restClient`의 `get` 메서드에 반환 타입을 동적으로 설정하면, API 요청 시 반환되는 데이터의 타입을 각 호출에서 명시적으로 지정할 수 있습니다.

### 제네릭 기본 개념

제네릭은 변수를 함수의 인자처럼 받아들이며, 실제 타입은 사용할 때 제공됩니다. 즉, 제네릭을 사용하면 특정 타입에 종속되지 않고 다양한 타입에 대해 동작하는 함수나 클래스를 작성할 수 있습니다. 일반적으로 제네릭 타입 변수는 `<T>`와 같은 형태로 나타내며, 여러 개의 타입 변수가 필요한 경우 `<T, U, V>`처럼 사용할 수 있습니다.

### `T`로 타입 매개변수 받기

제네릭을 사용하면 함수나 메서드를 정의할 때 타입을 하나의 매개변수로 전달받을 수 있습니다. 이 매개변수를 `T`라고 하면, `T`는 함수 호출 시점에서 전달받은 구체적인 타입으로 대체됩니다. 제네릭을 사용한 함수의 기본 구조는 다음과 같습니다.

```tsx
function identity<T>(arg: T): T {
    return arg;
}
```

이 `identity` 함수는 `T`라는 타입 매개변수를 받고, 인자 `arg`와 반환 타입으로 `T`를 사용합니다. 호출 시점에 `T`를 지정하거나, 인자 타입에 따라 자동으로 `T`가 추론됩니다.

### `restClient`에 제네릭 적용하기

제네릭 타입을 사용하여 `restClient`의 `get` 메서드를 정의하면, API 호출 시 데이터의 타입을 명시적으로 지정해 TypeScript의 타입 안전성을 높일 수 있습니다. 아래는 이를 적용한 `RestClient` 클래스의 예시입니다.

```tsx

const restClient = {
  get: async <T>(url: string) => {
    const response = await fetch(`${BASE_URL}${url}`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
      },
      credentials: "include",
    });

    if (!response.ok) {
      throw new Error("Network response was not ok");
    }

    // 제네릭 타입으로 반환 타입을 지정
    return response.json() as T;
  }
}

```

여기서 `get` 메서드는 `T`라는 제네릭 타입을 받아 `T` 형태로 데이터를 반환합니다. 이렇게 하면 `get` 메서드를 호출할 때 각 API 응답의 데이터 타입을 명확히 지정할 수 있으며, TypeScript는 이 타입을 기반으로 타입 검사를 수행합니다.

### 실제로 API 요청해보기

예를 들어, 사용자 정보를 가져오는 API 호출이 있다고 가정해 봅시다. 이때 `User`라는 인터페이스를 정의하고, `get` 메서드를 호출할 때 `User` 타입을 `T`에 전달해 반환 타입을 구체화할 수 있습니다.

```tsx

interface User {
  id: number;
  name: string;
  email: string;
}

const restClient = new RestClient();

async function fetchUserData() {
  const userData = await restClient.get<User>("/api/user");
  console.log(userData.name); 
}

```

위의 `restClient.get<User>("/api/user")` 호출에서 반환되는 데이터 타입은 `User`로 추론되며, `userData.name`처럼 데이터의 속성에 접근할 때 자동 완성과 타입 검사가 지원됩니다.



[참고문서]

https://www.typescriptlang.org/docs/handbook/2/generics.html