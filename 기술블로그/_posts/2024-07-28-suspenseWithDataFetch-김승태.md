---
layout: post
title: "Suspense를 사용하여 Data Fetching 처리하기"
author: "김승태"
banner:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["React", "Suspense"]
---

# Suspense를 사용하여 Data Fetching 처리하기

이번 글에서는 `Suspense`를 활용해 data fetching을 처리해보겠습니다.

각각 `useEffect`를 사용한 방식과, `Suspense`를 활용한 방식의 컴포넌트를 구현하고 성능을 비교해보겠습니다.

구현할 컴포넌트의 조건은 아래와 같습니다.<br>

1. `CatProfile`, `CatInfo` 모두 로딩중일 때는 `CatProfile`이 로딩중임을 표시하기.
2. `CatProfile` 컴포넌트가 로딩 중일 때는 `CatProfile`이 로딩중임을 표시하기.
3. `CatInfo`가 로딩 중에는 `CatInfo`가 로딩 중임을 표시하기.

조건에 따라 구현된 화면은 아래와 같습니다.<br><br>
`CatProfile`, `CatInfo` 모두 로딩중일 때는 `CatProfile`이 로딩중일 때.<br>
`CatProfile` 컴포넌트가 로딩 중일 때는 `CatProfile`이 로딩중일 때.<br>

<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/5.png" /><br>

`CatInfo`가 로딩 중에는 `CatInfo`가 로딩 중일 떄.

<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/6.png" /><br>

두 컴포넌트의 컨텐츠가 렌더딩 되었을 떄.

<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/7.png" /><br>

## Suspense를 사용하지 않은 방식

### useEffect로 data fetching 하기

#### `CatProfile` 컴포넌트

```tsx
export function CatProfile() {
  const [cat, setCat] = useState<CatDataType>();

  useEffect(() => {
    const getCatInfo = async () => {
      // 3. 데이터를 받아옵니다.
      const response = await fetch(getCatDateUrl);
      const data: CatDataType = await response.json();

      // 4. 받아온 데이터를 이용해 state를 업데이트 합니다.
      setCat(data);
    };

    // 2. 컴포넌트 마운트 이후 getCatInfo() 함수가 호출됩니다.
    getCatInfo();
  }, []);

  // 1. 데이터가 오기 전까지는 로딩상태가 표시됩니다.
  if (!cat) {
    return <h1>Cat profile loading</h1>;
  }

  // 5. 데이터 페칭이 완료된 이후에 실제 컨텐츠가 렌더링 됩니다.
  return (
    <div>
      <h1>CAT Profile</h1>
      <p>{cat.id}</p>
      <img src={cat.url} />
      <CatInfo id={1} />
    </div>
  );
}
```

#### `CatInfo` 컴포넌트

```tsx
function CatInfo({ id }: { id: number }) {
  const [catInfo, setCatInfo] = useState<CatInfoType>();

  useEffect(() => {
    const getCatInfo = async () => {
      // 3. 데이터를 받아옵니다.
      const response = await fetch(getCatInfoUrl(id));
      const data = await response.json();

      // 4. 받아온 데이터를 이용해 state를 업데이트 합니다.
      setCatInfo(data);
    };

    // 2. 컴포넌트 마운트 이후 getCatInfo() 함수가 호출됩니다.
    getCatInfo();
  }, []);

  // 1. 데이터가 오기 전까지는 로딩상태가 표시됩니다.
  if (!catInfo) {
    return <h1>Cat info loading</h1>;
  }

  // 5. 데이터 페칭이 완료된 이후에 실제 컨텐츠가 렌더링 됩니다.
  return (
    <div>
      <h1>CAT Info</h1>
      <p>{catInfo}</p>
    </div>
  );
}
```

해당 컴포넌트는 CatProfile의 data fetching이 끝난 후, CatInfo가 렌더링되며 네트워크 waterfall을 일으킵니다.<br>
<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/1.png" /><br>

어떻게 개선할 수 있을까요?

## Suspense를 사용한 방식(Render as you fetch)

[공식문서](https://17.reactjs.org/docs/concurrent-mode-suspense.html#what-suspense-is-not)의 내용을 참조하여 먼저 useEffect를 활용한 방식을 다시 되짚어 보겠습니다. 기존 방식은 Fetch-on-render라고 하며 아래와 같은 동작을 하고 있습니다.

1. start fetching
2. finish fetching
3. start rendering

Suspense를 사용하면 2번과 3번의 순서를 변경할 수 있습니다.

1. start fetching
2. start rendering
3. finish fetching

즉, Suspense를 사용하면 렌더링을 시작하기 전에 응답이 돌아올 때까지 기다리지 않고, 실제로 네트워크 요청을 시작한 후 거의 즉시 렌더링을 시작합니다. 이를 **Render-as-you-fetch**라고 합니다.

먼저 Suspense를 활성화하려면 어떻게 해야할까요?

[공식문서](https://react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)에 따르면 독단적인 프레임워크를 사용하지 않는 Suspense가 가능한 데이터 불러오기 기능은 아직 지원되지 않습니다.
또한 Suspense 지원 데이터 소스를 구현하기 위한 요구 사항은 불안정하고 문서화되지 않았습니다.

하지만 방법은 존재합니다.

[리액트 RFC의 질문과 답변](https://github.com/reactjs/rfcs/pull/213#issuecomment-1077984147)에 따르면 현재로서는 Promise를 던지는 방식으로 작동한다고 합니다.<br>

## 기존 코드에 Suspense 적용해보기

### 1. `getFetchWithSuspense` 함수 구현

따라서 아래와 같은 Promise를 throw하는 함수를 만듭니다.

`getFetchWithSuspense` 함수는 결국 `read` 함수를 반환하는데, 해당 `read` 함수는 `status`에 따라 promise를 throw하거나, 값을 반환합니다.

```tsx
const cache: { [key: string]: any } = {};
// 해당 cache는 key에 대응되는 값이 존재한다면 해당 값을 return합니다.
// cache를 정의한 이유는, Promise를 React 내부가 아닌 외부에 저장하기 위해서입니다.

type WrappedPromise<T> = {
  read: () => T;
};

const wrapPromise = <T,>(promise: Promise<T>): WrappedPromise<T> => {
  let status = "pending"; // promise의 상태를 나타내는 값입니다.
  let response: T; // promise가 실행되고 난 뒤의 결과값입니다.

  // Promise입니다.
  const suspender = promise.then(
    (res) => {
      status = "success"; // 성공했다면 상태와 값을 변경합니다.
      response = res;
    },
    (err) => {
      status = "error"; // 실패했다면 상태와 값을 변경합니다.
      response = err;
    }
  );

  const read = () => {
    switch (status) {
      // 처음 read 함수를 호출했을 때는 promise를 throw 합니다.
      case "pending":
        throw suspender;
      // Promise가 실행되고 실패했다면 error를 throw 합니다.
      case "error":
        throw response;
      // 성공했다면 값을 반환합니다.
      default:
        return response;
    }
  };

  return { read };
};

const getFetchWithSuspense = <T,>(url: string): WrappedPromise<T> => {
  // 1. url이 입력됩니다.

  // 2. url에 대응되는 값이 없다면 만들어줍니다.
  if (!cache[url]) {
    cache[url] = wrapPromise<T>( //
      fetch(url).then((response) => {
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        return response.json();
      })
    );
  }

  // 3. 값을 반환합니다.
  return cache[url];
};
```

### 2. 기존 컴포넌트에 적용

컴포넌트에서 `getFetchWithSuspense`를 호출해보겠습니다.

```tsx
export function CatProfileWithSuspense() {
  return (
    <Suspense fallback={<h1>Cat profile loading</h1>}>
      <CatProfile />
      <Suspense fallback={<h1>Cat info loading</h1>}>
        <CatInfo id={1} />
      </Suspense>
    </Suspense>
  );
}

function CatProfile() {
  // data를 가져옵니다. 첫 호출시에 suspense가 throw되어 가장 가까운 Suspense에서 fallback이 표시됩니다.
  const cat = getFetchWithSuspense<CatDataType[]>(getCatDateUrl).read();

  return (
    <div>
      <h1>CAT Profile</h1>
      <p>{cat.id}</p>
      <img src={cat.url} />
    </div>
  );
}

function CatInfo({ id }: { id: number }) {
  // data를 가져옵니다. 첫 호출시에 suspense가 throw되어 가장 가까운 Suspense에서 fallback이 표시됩니다.
  const catInfo = getFetchWithSuspense(getCatInfoUrl(id)).read();

  return (
    <div>
      <h1>CatInfo</h1>
      <p>{catInfo}</p>
    </div>
  );
}
```

공식문서에 따르면 리액트는 다음과 같은 방식으로 동작합니다.<br>

1. `CatProfileWithSuspense`는 두 자식, `CatProfile`, `CatInfo`를 반환합니다.
2. react는 `CatProfile`을 렌더링 하려고 시도합니다. `read()`가 호출되고 컴포넌트는 "suspend"됩니다. react에서는 `CatProfile`을 건너뛰고 `CatInfo` 컴포넌트를 렌더링하려 시도합니다.
3. `CatInfo`에서도 `read()`함수가 호출되고 컴포넌트는 "suspend"됩니다. 렌더링을 시도할 것이 남아있지 않기에 가장 가까운 Suspense의 `fallback` 을 표시합니다.
4. 값을 확인하고 컴포넌트의 fetch가 종료되었다면 해당 컴포넌트를 렌더링합니다.

해당 방식으로 컴포넌트를 구현했을 때의 네트워크 호출을 확인해보니, 성능이 개선됨을 확인할 수 있습니다.
<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/3.png"/><br><br>

## 결론

크롬 개발자 도구에서 네트워크 속도가 Slow 4G 기준일 떄의 성능 비교는 아래와 같습니다.<br><br>
<img src="https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0905/4.png"/><br>

비동기 데이터를 가져오는데 있어서 Suspense를 활용한다면 더 좋은 성능을 만들 수 있음을 확인할 수 있었습니다. TanStack Query, SWR 등의 데이터를 가져오는 라이브러리도 Suspense를 지원하니 이를 사용한다면 사용자에게 더욱 좋은 경험을 만들 수 있을 것 같습니다.

## 출처

[Suspense for Data Fetching](https://17.reactjs.org/docs/concurrent-mode-suspense.html#what-suspense-is-not)<br>

[0213-suspense-in-react-18](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)<br>

[Suspense for Data Fetching의 작동 원리와 컨셉 (feat.대수적 효과)](https://maxkim-j.github.io/posts/suspense-argibraic-effect/)<br>

[React Query와 함께 Concurrent UI Pattern을 도입하는 방법
](https://tech.kakaopay.com/post/react-query-2/)

[리액트 Suspense 딥다이브](https://velog.io/@jay/Suspense)

[Data fetching with React Suspense](https://blog.logrocket.com/data-fetching-react-suspense/)
