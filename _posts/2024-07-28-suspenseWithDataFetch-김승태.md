---
layout: post  
title: `Suspense with data fetch`
author: `김승태`
categories: `프론트엔드 기술블로그`
banner:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`React`, `Suspense`,]
---

# Suspense with data fetching

## Suspense란?

Suspense 컴포넌트는 children컴포넌트가 로딩될 때까지 fallback을 보여주는 컴포넌트입니다.

> `<Suspense>` lets you display a fallback until its children have finished loading.

```tsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

### 사용법

Suspense는 아래와 같은 상황일 시에 동작합니다.

- Data fetching with Suspense-enabled한 프레임워크와 같이 사용할 때(Data fetching with Suspense-enabled frameworks like [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) and [Next.js](https://nextjs.org/docs/getting-started/react-essentials))
  ```tsx
  import { Suspense } from 'react'

  function async PostFeed(){
  	const feed = await getPostFeed(); // something asynchronous action

  	return {feed}; // return value
  }

  export default function Posts() {
    return (
      <section>
        <Suspense fallback={<p>Loading feed...</p>}>
          <PostFeed />
        </Suspense>
      </section>
    )
  }
  ```
- [`use`](https://react.dev/reference/react/use) 를 이용해 Promise의 값을 읽을 때
  ```tsx

  function async ChildComponent(){
  	const value = use(resource);

  	return <div>{value}</div>
  }

  function ParentComponent(){
  	return (
  		<Suspense fallback={<Loading />}>
  			<ChildComponent/>
  		</Suspense>)
  }

  ```
- [`lazy`](https://react.dev/reference/react/lazy) 와 함께 Lazy-loading component를 사용할 때: React.lazy와 함께 쓴다면 동적으로 컴포넌트를 가져올 때, 자연스럽게 로딩 처리를 해줄 수 있습니다.

```tsx
import { lazy } from "react";
const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
function Component() {
  return (
    <Suspense fallback={<Loading />}>
      <h2>Preview</h2>
      <MarkdownPreview />
    </Suspense>
  );
}
```

## Suspense를 왜 쓰는가?

먼저 보편적으로 많이 쓰는 방식을 보겠습니다.

```tsx
const { data, isLoding } = useGetFetch(url);

if (isLoding) {
  return <h1>Loding</h1>;
}

return <Card data={data} />;
```

```tsx
const [isLoding, setIsLoding] = useState(true);
const [data, setData] = useState(null);

useEffect(() => {
  const getData = async () => {
    const remoteData = await fetch(url);
    setIsLoding(false);
    setData(remoteData);
  };
  getData();
}, []);

if (isLoding) {
  return <Loading />;
}

return <Card data={data} />;
```

이제 Suspense를 사용해보겠습니다.

```tsx
function ChildComponent(){
	const data = getFetchWithSuspense(url).read();

	return <Card data={data} />
}

function ParentComponent(){
	return(
		<Suspense fallback={<Loading />}>
			<ChildComponent>
		</Suspense>
		)
}
```

두 코드를 비교할 때 보이는 이점은 데이터 로직을 불러왔을 때의 경우와 로딩중인 때의 경우를 분리할 수 있다는 것입니다.<br>
이러한 이점은 연쇄적인 복수의 비동기 작업을 호출할 때, 더욱 두드러집니다.<br>
비동기통신을 통해 A라는 값을 받아오고 그 값을 이용해 B라는 비동기 함수를 호출할 때의 예제입니다.

```tsx
function B({ data1 }) {
  const { data, isLoding } = useGetFetch(url + data1);

  if (isLoding) {
    return <Loading />;
  }

  return <Card data={data} />;
}

function A() {
  const { data, isLoding } = useGetFetch(url);

  if (isLoding) {
    return <Loading />;
  }

  return (
    <>
      <B data1={data} />
    </>
  );
}
```

```tsx
const [isLoding, setIsLoding] = useState(true);
const [data, setData] = useState(null);

useEffect(() => {
  const getData = async () => {
    const remoteData1 = await fetch(url1);
    const remoteData2 = await fetch(url2);
    setIsLoding(false);
    setData(remoteData);
  };
}, []);

if (isLoding) {
  return <Loading />;
}

return <Card data={data2} />;
```

반면 Suspense를 활용하면 아래와 같습니다

```tsx
function ChildComponent(){
	const data1 = getFetchWithSuspense(url1).read();
	const data2 = getFetchWithSuspense(url2 + data1).read();

	return <Card data={data2} />
}

function ParentComponent(){
	return(
		<Suspense fallback={<Loading />}>
			<ChildComponent>
		</Suspense>
		)
}
```

데이터를 받아왔을 때와 로딩 상태를 분리함으로써 비동기 호출이 복잡해도 간결하게 이해될 수 있는 코드가 되었습니다. <br>
이번 프로젝트는 오픈 API를 이용했기 때문에 화면에 맞는 API호출이 아닌, 연쇄적인 API호출(유저 정보 조회 → 유저 매치 목록 조회 → 유저 매치 정보 조회)을 해야했고, 코드의 복잡함을 줄이기 위해서 적용했습니다.

## Suspense의 원리

suspense는 어떻게 하위 컴포넌트의 상태를 파악할 수 있을까요?

```tsx
// 실제로 React의 Suspense가 이것과 똑같이 동작하지는 않겠지만
// 구현 컨셉을 잘 드러내고 있는 코드 조각입니다.

async function runPureTask(task) {
  for (;;) {
    // while true
    //!!! 태스크를 리턴할 수 있을 때까지 바쁜대기를 함(무한루프) !!!
    try {
      return task(); // 태스크 값을 리턴할 수 있게 되면 무한루프에서 벗어난다
    } catch (x) {
      // throw를 거른다
      if (x instanceof Promise) {
        await x; // pending promise가 throw된 경우 await으로 resolve 시도 => suspense
      } else {
        throw x; // Error가 throw된 경우 그대로 error throw => ErrorBoundary, 종료
      }
    }
  }
}
```

즉 위와 같은 코드에서 Promise를 받는다면 promise를 기다려줍니다. 따라서 Suspense로 감싸져있는 컴포넌트가 Promise를 던진다면 그 Promise를 실행하는 동안 fallback을 보여줍니다.

```tsx
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log("Promise over!");
    resolve(3);
  }, 300);
}).then((value = "") => {
  console.log(`Promise then ${value}`);
  return 3;
});

const wrapPromise = (promise) => {
  let status = "pending";
  let response;

  const suspender = promise.then(
    (res) => {
      status = "success";
      response = res;
    },
    (err) => {
      status = "error";
      response = err;
    }
  );

  const read = () => {
    switch (status) {
      case "pending":
        throw suspender;
      case "error":
        throw response;
      default:
        return response;
    }
  };
  return { read };
};

const resource = wrapPromise(p);

function App() {
  const res = resource.read();

  return <h1>App.tsx {res}</h1>;
}

export default App;
```

```tsx
const res = resource.read();
```

case 'pending'이고 suspender를 throw합니다.

```tsx
if (x instanceof Promise) {
  await x; // pending promise가 throw된 경우 await으로 resolve 시도 => suspense
}
```

받은 promise를 resolve합니다.

```tsx
const res = 3; // resource.read();
/*
new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('Promise over!');
    resolve(3);
  }, 300);
}).then((value = '') => {
  console.log(`Promise then ${value}`);
  return 3;
});
*/
```

```tsx
function App() {
  const res = resource.read();

  return <h1>App.tsx {res}</h1>; // App.tsx 3
}
```

## 실제 적용 코드

```tsx
import axios from "axios";

const cache = {};

const wrapPromise = (promise) => {
  let status = "pending";
  let response;

  const suspender = promise.then(
    (res) => {
      status = "success";
      response = res;
    },
    (err) => {
      status = "error";
      response = err;
    }
  );

  const read = () => {
    switch (status) {
      case "pending":
        throw suspender;
      case "error":
        throw response;
      default:
        return response;
    }
  };
  return { read };
};

const getFetchWithSuspense = (url) => {
  if (!cache[url]) {
    cache[url] = wrapPromise(
      axios
        .get(url)
        .then((response) => response.data)
        .catch((error) => {
          return Promise.reject(error);
        })
    );
  }

  return cache[url];
};

export { getFetchWithSuspense };
```

```tsx
const getIdUrl = `/riot/account/v1/accounts/by-riot-id/${gameName}/${tagLine}?api_key=${API_KEY}`;
const { puuid } = getFetchWithSuspense(getIdUrl).read();
const getMatchIdListUrl = `/lol/match/v5/matches/by-puuid/${puuid}/ids?start=0&count=${MATCH_COUNT_LENGTH}&api_key=${API_KEY}`;
const matchIdList = getFetchWithSuspense(getMatchIdListUrl).read();
```

## 참고자료

https://react.dev/reference/react/Suspense

https://17.reactjs.org/docs/concurrent-mode-suspense.html#approach-3-render-as-you-fetch-using-suspense

https://fe-developers.kakaoent.com/2021/211127-211209-suspense/

[https://medium.com/@itsinil/react-suspense-for-async-data-fetch-컨셉-이해하기-cfa3d470ec4f](https://medium.com/@itsinil/react-suspense-for-async-data-fetch-%EC%BB%A8%EC%85%89-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-cfa3d470ec4f)

https://kasterra.github.io/data-fetching-and-react-suspense/

https://maxkim-j.github.io/posts/suspense-argibraic-effect/

https://www.youtube.com/watch?v=FvRtoViujGg&t=20s

https://tech.kakaopay.com/post/react-query-2/

https://velog.io/@shinhw371/React-suspense-throw
