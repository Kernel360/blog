---
layout: post  
title: "Next.js에서 Result 패턴을 활용한 비동기 요청 에러 처리"
author: "오송민"
categories: "프론트엔드 기술블로그"
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`next.js`, `result type`, `result pattern`, `typescript`]
---

# 서론

Typescript에서 런타임 에러를 잡기 위해서 `try…catch`를 사용할 수 있습니다.
`try` 블록 내 코드에서 비동기 요청을 수행하고, 만약 그 안에서 예외가 발생한다면 `catch` 블록 내 코드를 실행하여 에러를 처리하는게 일반적입니다.

이번 프로젝트에서는 토스트 형식으로 사용자에게 에러가 발생했음을 알리고 있습니다.
클라이언트측 요청인 경우, API 클라이언트에서 공통적으로 `catch` 블록에서 발생한 에러에 대한 토스트 메세지 상태를 세팅해주고 있습니다.

하지만 `Next.js server`를 통해 요청을 보낼 경우는 해당 방식을 사용할 수 없었고, 요청을 호출한 `RCC`에서 **다시 에러를 핸들링** 해야 하는 문제가 발생했습니다.

또한, 비동기 함수를 호출할 때 마다 반복적으로 작성해야하는`try…catch`문은 중복되는 코드를 증가시키며, 코드 가독성을 해치는 것이 아닌가라는 생각이 들었습니다.

해당 문제들을 해결하기 위해서 **에러 처리 로직을 모듈화**하여 중복을 줄이고 유지보수성을 높이는 방향으로 리팩토링 해보았습니다.

# Result 타입

중앙 집중화된 에러 처리 로직을 만든다면 해당 문제를 해결할 수 있을 듯 보입니다. 그러기 위해선 모든 비동기 요청에 대해 **정형화된 반환값을 정의해야 합니다.**

`Rust`의 [Result](https://doc.rust-lang.org/std/result/)타입을 레퍼런스로 하여 나온 다양한 Typescript 라이브러리를 참고하여 프로젝트에서 사용할 공통된 반환 타입을 만들었습니다.

라이브러리를 사용하실 분들은 [fp-ts](https://github.com/gcanti/fp-ts), [neverthrow](https://github.com/supermacro/neverthrow), [typescript-result](https://github.com/everweij/typescript-result) 등을 설치하시면 됩니다. 다양한 메서드를 제공하고 안정성과 사용성면에서 라이브러리를 사용하는 것도 좋지만, 구현체가 복잡하지 않아 아이디어를 참고하여 프로젝트의 사용성에 맞게 구현하여 사용했습니다.

`Result` 타입은 프로젝트 위클리 때 멘토님께서 알려주신 문제 해결 방법..이자 타입입니다.
`Rust`의 `Result` 타입은 함수나 메서드가 성공했을 때와 실패했을 때의 두 가지 경우를 명시적으로 표현할 수 있도록 도와주는 표준 타입입니다.

## Rust의 Result<T, E>

Rust의 `Result` 타입은 두개의 `Generic`을 사용하여 정의됩니다.

```rust
enum Result<T, E> {
    Ok(T),   // 성공한 경우, T 타입의 값을 담습니다.
    Err(E),  // 실패한 경우, E 타입의 에러 정보를 담습니다.
}
```

- **T**: 성공했을 때 반환할 값의 타입입니다.
- **E**: 실패했을 때 반환할 에러의 타입입니다.

이렇게 하면 함수가 반환하는 값과 에러 정보를 호출자가 명시적으로 다룰 수 있게 되어, 에러 처리를 타입 시스템 수준에서 강제할 수 있습니다.

### 사용 예시

예를 들어, 파일을 읽어 문자열로 반환하는 함수를 작성할 때, 성공하면 파일 내용(String)을, 실패하면 입출력 에러(`std::io::Error`)를 반환하도록 할 수 있습니다.

```rust
use std::fs;
use std::io;

fn read_file(path: &str) -> Result<String, io::Error> {
    fs::read_to_string(path)
}
```

이 함수는 내부에서 파일 읽기 작업이 실패할 경우 자동으로 `Err` 값을 반환합니다.

또는 패턴 매칭을 통해 처리할 수도 있습니다.

호출하는 쪽에서는 `match`를 사용하여 성공/실패를 분기 처리할 수 있습니다.

```rust
fn main() {
    match read_file("example.txt") {
        Ok(contents) => println!("파일 내용: {}", contents),
        Err(e) => eprintln!("파일 읽기 실패: {}", e),
    }
}
```

이렇게 하면 에러 상황에 따라 다른 처리를 명확하게 구현할 수 있습니다.

## 프로젝트의 Result<T>

Rust의 Result Type과 위 라이브러리들은 모두 유연한 에러 정보를 제공하기 위해 제네릭을 사용하여 에러의 타입(E)을 다양하게 지정할 수 있습니다.

하지만 저는 토스트의 에러 메세지를 세팅해 주기 위해 사용할 것이기 때문에 고정된 에러 타입을 지니도록 만들었습니다.

만약 나중에 더 다양한 에러 정보가 필요해질 경우 `Result<T, E>`에 비해 확장이 어려울 수 있지만, 현재 요구사항에서는 메세지로도 충분하다 판단했습니다.

```jsx
export type Result<T> = SuccessResult<T> | ErrorResult

type SuccessResult<T> = { success: true; data: T }
type ErrorResult = { success: false; message: string }
```

- T: 성공했을 때 반환할 값의 타입입니다.

> Result<T>를 만듦으로써 문제를 50% 해결했습니다.

# 공통 에러 처리 로직

`Result<T>`를 사용하는 비동기 요청을 처리하는 함수를 정의할 차례입니다.

## 기존 에러 처리 방식

### 클라이언트 요청일 경우

클라이언트 요청에 대한 에러 메세지는 `ky`의 `beforeRequest`에서 처리해주고 있었습니다.

```jsx
import ky from 'ky-universal'

import { useToastStore } from '@/stores/toast'
import { ErrorResponseBody } from '@/types/ErrorResponseBody'
import { handleBeforeError, setServerAuthorizationHeader } from '@/apis/hooks'

const isServer = typeof window === 'undefined'

export const apiClient = ky.create({
  prefixUrl: process.env.NEXT_PUBLIC_BACKEND_URL,
  credentials: 'include',
  timeout: 10000,
  hooks: {
    beforeRequest: isServer ? [setServerAuthorizationHeader] : [],
    beforeError: isServer ? [] : [handleBeforeError],
  },
})
```

에러의 타입에 맞게 분기처리하여 에러 메세지를 지정해주었습니다.

```jsx
import { HTTPError } from 'ky';
import { useToastStore } from '@/stores/toast';
import { ErrorResponseBody } from '@/types/ErrorResponseBody';

export async function handleBeforeError(error: unknown): Promise<never> {
  let message: string;

  if (error instanceof HTTPError) {
    try {
      const body = (await error.response.json()) as ErrorResponseBody;
      message = body.message;
    } catch {
      message = 'HTTP 에러가 발생했습니다.';
    }
  } else if (error instanceof TypeError) {
    message = '서버와의 연결이 원활하지 않습니다.';
  } else {
    message = '알 수 없는 에러가 발생했습니다.';
  }

  const { setToastMessage } = useToastStore.getState();
  setToastMessage({
    type: 'error',
    message,
  });
}
```

### 서버 요청일 경우

하지만 클라이언트 요청이 아닌 서버 요청이라면, 해당 훅은 실행되지 않습니다.
때문에 **서버 요청 함수를 호출하는 부분에서 다시 에러를 핸들링** 해주어야 했습니다.

```jsx
  async function onSignupSubmit(req: SignupFormValues) {
    try {
	    const res = await postSignup(req) // <- 회원가입 요청을 보내는 server action

      setToastMessage({
        type: 'success',
        message: `${req.nickname}님 환영합니다! 가입 완료! 🥳`,
      })
      router.push('/fortune')
    } catch (error) {
      if (error instanceof HTTPError) {
	      const errorBody = await error.response.json();

	      setToastMessage({
	        type: 'error',
	        message: errorBody.message,
	      });
      } else if (error instanceof TypeError) {
        setToastMessage({
          type: 'error',
          message: '서버와의 연결이 원활하지 않습니다.',
        })
      } else {
        setToastMessage({
          type: 'error',
          message: '알 수 없는 에러가 발생했습니다.',
        })
      }
    }
  }
```

## 기존 방식의 문제점

기존에 고민하고 있던 문제점과 더불어 디렉터님이 피드백 해주신 부분을 정리하여 개선해야 할 문제를 설정해보았습니다.

### 1. 중복된 코드

요청마다 각각 `try…catch` 블록 내에서 별도의 에러 처리 로직을 작성해야 했습니다.
이로 인해 **같은 에러 핸들링 코드가 여러 곳에 중복되어, 유지보수와 확장이 어려워졌습니다.**

만약 `catch`블럭에서 토스트의 에러 메세지를 세팅하는 기능 이외의 로직이 존재했다면 해당 방식을 사용하는 것에 큰 의문을 품지 않았을 것입니다.

### 2. 단일 책임 원칙 위배

현재 API 클라이언트(`apiClient`)내부에 토스트 메시지 설정 등 에러 처리 로직이 포함되어 있습니다.

**네트워크 요청과 응답 처리에 집중해야 할 API 클라이언트의 본연의 역할과 분리되지 않고, 에러 처리까지 담당하게 되어 코드의 모듈성이 떨어졌습니다.**

### 3. **에러 메시지 처리의 불일관성**

클라이언트와 서버 요청에서 에러 메시지가 설정되는 시점과 위치가 분리되어 있어, 어느 시점에 사용자에게 에러 메시지가 전달되는지 명확하게 파악하기 어려웠습니다.

이로 인해 **에러 처리 흐름의 일관성을 유지하기 힘들었고, 개발자가 요청 환경에 따라 별도로 처리해야 하는 부담이 있었습니다.**

## 개선된 에러 처리 방식

### handleAsyncResult

앞서 만든 `Result<T>`를 사용하여 서버와 클라이언트 모두 사용할 수 있는 공통된 에러 처리 함수를 만들었습니다. 매개변수로는 실행할 비동기 함수를 받습니다.

요청이 성공할 경우, `Result<SuccessResult<T>>` 객체를 반환합니다.

만약 요청이 실패할 경우, `catch`블럭에서 이전과 동일하게 토스트에 띄워질 에러 메세지를 설정해줍니다. 이전 코드와의 차이점은 **토스트 메세지 상태를 설정해주는 대신 에러 메세지 자체를 반환**하는 것 입니다.

에러일 경우 `Error` 대신 `Result<ErrorResult>` 객체를 반환하여 에러가 전파되지 않도록 했습니다.

```tsx
import { HTTPError } from 'ky'

import { Result } from '@/types/Result'

export const handleAsyncResult = async <T>(
  fn: () => Promise<T>,
): Promise<Result<T>> => {
  try {
    const data = await fn()
    return { success: true, data }
  } catch (error) {
    if (error instanceof HTTPError) {
      return {
        success: false,
        message: await error.response.json().then(data => data.message),
      }
    }
    if (error instanceof TypeError) {
      return {
        success: false,
        message: '서버와의 연결이 원활하지 않습니다.',
      }
    }
    return {
      success: false,
      message: '알 수 없는 에러가 발생했습니다.',
    }
  }
}
```

### handleErrorToast

`handleAsyncResult()`에서 반환받은 `Result<T>`객체를 `handleErrorToast`로 넘겨주어 토스트 메세지를 설정해 줄 수 있습니다.

성공했을 때, 실패했을 때 실행할 콜백 함수를 필요에 따라 넘겨줄 수 있도록 구현해보았습니다.

```tsx
interface HandleErrorToastProps<T> {
  result: Result<T>
  onError?: () => void
  onSuccess?: () => void
}

export function handleErrorToast<T>({
  result,
  onError = () => {},
  onSuccess = () => {},
}: HandleErrorToastProps<T>) {
  const { setToastMessage } = useToastStore.getState()

  if (!result.success) {
    setToastMessage({
      type: 'error',
      message: result.message,
    })
    if (onError) {
      onError()
    }
  } else if (result.success) {
    if (onSuccess) {
      onSuccess()
    }
  }
}
```

그리고 `apiClient` 내부에서 클라이언트 요청일 때 실행되던 `beforeRequest` 훅의`handleBeforeError` 함수를 제거했습니다.

## 사용 예시

### 서버 요청일 경우

회원가입 요청을 보내는 함수를 이제 아래와 같이 사용할 수 있게 되었습니다.

`handleErrorToast` 내부에서 응답에 대한 상태를 확인하여 error 타입의 토스트를 띄울지, `onSuccess` 함수를 실행하여 success 타입의 토스트를 띄울지 결정합니다.

```jsx
 async function onSignupSubmit(req: SignupFormValues) {
    function onSuccess() {
      setToastMessage({
        type: 'success',
        message: `${req.nickname}님 환영합니다! 가입 완료! 🥳`,
      })
      router.push('/fortune')
    }

    const res = await handleAsyncResult(() => postSignup(req)) // <- 회원가입 요청을 하는 server action
    handleErrorToast({ result: res, onSuccess })
  }
```

### 클라이언트 요청일 경우

`getCheckNickname`은 회원가입시 중복된 닉네임인지를 확인하는 비동기 요청 함수입니다. 해당 API는 브라우저를 통해 요청하고 있습니다.

```tsx
  async function checkNicknameDuplicate(nickname: string) {
    function onSuccess() {
      setToastMessage({
        type: 'success',
        message: '사용 가능한 닉네임입니다.',
      })
    }

    const res = await handleAsyncResult(() => getCheckNickname({ nickname }))
    handleErrorToast({ result: res, onSuccess })
  }
}
```

### 흐름 도식화

요청의 흐름을 도식화 하면 아래와 같습니다.

<img src="https://github.com/Kernel360/blog-image/blob/main/2025/0202/mermaid.png?raw=true"/>

**API 호출 시작**

- 컴포넌트나 페이지에서 API 호출을 시작하면서 `handleAsyncResult`를 실행합니다.

**handleAsyncResult**

- 내부의 try 블록에서 비동기 함수를 실행하여 성공 시에는 `{ success: true, data }` 형태의 Result를 반환합니다.
- 실패 시 catch 블록에서 에러 타입(HTTPError, TypeError, 기타)에 따라 분기 처리하여 `{ success: false, message }` 형태의 Result를 반환합니다.

**handleErrorToast**

- 성공이면 onSuccess를 실행하고, 실패이면 에러 토스트와 onError를 실행합니다.

**최종 동작**

- 토스트 메시지 표시 후 추가 동작이 이어집니다.

## 개선된 사항

### 1. 중복된 코드 제거

에러를 처리하기 위해 반복적으로 작성해야했던 코드가 사라졌습니다.
`handleAsyncResult`, `handleErrorToast`를 호출하는 방식으로 유지보수가 용이하고 확장이 가능한 코드로 개선되었습니다.

### 2. 단일 책임 원칙 준수

API 클라이언트에 존재했던 토스트 메세지 설정이라는 UI 관련 로직을 들어내어 `apiClient`가 네트워크 요청이라는 단일 목적만 지니도록 로직을 분리하였습니다.

### 3. **에러 메시지 처리의 일관성**

서버와 클라이언트 요청 모두 동일한 시점과 방식으로 에러메세지를 설정할 수 있게 되었습니다. 개발자가 요청 환경에 따라 별도로 처리해야하는 부담이 사라지고, 사용자에게 에러 메세지가 전달되는 부분을 명확하게 파악할 수 있게 되었습니다.

> 이로서 문제의 99%가 해결되었습니다.

# 회고

`handleAsyncResult`는 단독적으로 사용할 수 있습니다. 만일 사용되는 컴포넌트 혹은 페이지가 `RSC`라면 반환받은 객체의 success를 확인하여 조건부 렌더링을 하는 방식으로 사용이 가능합니다.

반면 `handleErrorToast`를 사용하기 위해서는 선행적으로`handleAsyncResult`가 호출되어야 합니다.
하지만 현재 이런 흐름을 강제하고 있지 않습니다.. . 추후 두 함수를 순차적으로 호출하는 wrapper function을 만드는 방식으로 개선한다면 모자란 1%가 채워질 거 같습니다.

# 레퍼런스

- https://github.com/gcanti/fp-ts
- https://github.com/supermacro/neverthrow
- https://github.com/everweij/typescript-result
- https://ghlee.dev/posts/typescript-result-type
- https://ctidd.com/2018/typescript-result-type
