---
layout: post
title: “SSR과 Cookie”
author: 양하연
categories: 프론트엔드 기술블로그
banner: 
  image: assets/images/post/2023-11-05.webp
  background: “#000”
  height: “100vh”
    min_height: “38vh”
  heading_style: “font-size: 4.25em; font-weight: bold; text-decoration: underline”
  tags: ['SSR', 'Cookie','기술블로그' ]
---

### 배경
![initialData적용전_로딩해오기까지오래걸림_main](https://github.com/user-attachments/assets/5ea8c06b-c51c-473d-9cb4-d57f19cdd7fa)
</br>

<p>여러분이 처음 사이트에 들어갔을 때 위 움짤처럼 스크랩했는지 여부가 약 1초가 지난 후에야 뒤늦게 채워진다면 어떤 느낌이 드시나요? 신경쓰지 않는 분들이 있을 수도 있지만 제 경우에는 답답하게 느껴집니다.
<br/>
이런 현상이 발생하는 이유는 API호출이 완료된 이후에야 데이터 보여주는 데에서 발생하기 때문입니다. 사이트에 들어가자마자 데이터가 채워있도록 보이게 만들려면 미리 데이터를 불러와서 꽂아주는 방식으로 해결할 수 있습니다.
</br>
이번 프로젝트에서 사용하고 있는 TanStack Query라이브러리에서 데이터를 미리 가져와서 보여주는 방법은  initialData를 이용하는 방식과 prefetchQuery를 이용하는 방식이 일반적인데, 정적인 데이터에 더 적합한 initialData를 이용하기로 결정했습니다. 이 방법을 이용하면 데이터를 서버사이드에서 미리 가져와 사용자에게 빠르게 표시할 수 있습니다.
 
</p>
<br/>

### 문제발생 : 그런데, 왜 분명히 true였던 스크랩 데이터가 false로 오는거지?

아래와 같이 initialData를 이용해 데이터를 미리 꽂아주는 코드를 작성했고 프리뷰 컴포넌트의 북마크 부분이 색깔로 채워질 것을 기대했습니다.
</br>

```js
import { cookies } from "next/headers";

import {
  fetchReadingPreview,
  fetchListeningPreview,
} from "@/api/queries/contentsQueries";

import HomePageClient from "../../components/HomePageClient";

export default async function HomePage() {
  const initialReadingContents = await fetchReadingPreview({});
  const initialListeningContents = await fetchListeningPreview({});

  return (
    <HomePageClient
      initialReadingContents={initialReadingContents}
      initialListeningContents={initialListeningContents}
    />
  );
}

---


export default function HomePageClient({
  initialReadingContents,
  initialListeningContents,
}: HomePageClientProps) {
  const { data: readingList, isLoading: readingLoading } = useQuery({
    queryKey: ['readingPreview'],
    queryFn: () => fetchReadingPreview(),
    initialData: initialReadingContents,
  });

  const { data: listeningList, isLoading: listeningLoading } = useQuery({
    queryKey: ['listeningPreview'],
    queryFn: () => fetchListeningPreview(),
    initialData: initialListeningContents,
  });

  //...생략
}

```
<br/>

하지만 예상과 달리, initialData를 적용하기 전에는 제대로 불러오던 스크랩 데이터가, 오히려 initialData문법을 적용한 후에는 제대로 불러와지지 않는 문제를 발견했습니다. 
<br/>

<br/>
왜 그런지 알아보기 위해서 우선, TanStack Query Dev Tools를 확인해보았습니다. isScrapped가 false로 표시되고 있었습니다. 
![image](https://github.com/user-attachments/assets/5550fdf1-6580-49ba-bd80-4ee55562c1bc)



<br/>
Tanstack query의 문제가 아니라 데이터 자체가 false로 오고있는 것일 수도 있기 때문에 Swagger를 통해 실제 API를 호출하여 확인해보지만, 실제 isScrapped 값은 true로 오고있었습니다. 
![image](https://github.com/user-attachments/assets/fab59d2f-e864-4e7c-8ee9-4be80247619b)

게다가, 해당 컴포넌트에서 console.log로 값을 찍어봐도 Dev Tools에서 확인했던 것처럼 isScrapped가 잘못된 값인 false로 오는 상황이었습니다.
즉 실제 데이터와 Tanstack query로 불러오고 있는 데이터가 다른 상황인 것이었습니다!

<br/>

### 원인 분석
왜 이런 문제가 발생하는 것 일까?
처음에는 TanStack Query의 staleTime 설정 때문일까 싶어서, 6분으로 설정되어있던 staleTime을 0으로 설정해 보았고 데이터가 제대로 불러와지는 것을 확인할 수 있었습니다.
staleTime이 지나면 initialData로 불러온 데이터 대신 useQuery로 설정된 쿼리를 다시 요청하게 되기 때문입니다. 이 과정에서 처음 마주했던 false 값은 initialData로 데이터를 불러오는 과정에서 발생한 잘못된 값임을 추측할 수 있었습니다.
<br/>

그렇다면 왜 서버사이드 렌더링에서 문제가 발생한 것일까요? 다른 데이터들은 잘 불러와지는데 왜 스크랩 데이터만 잘못 불러오는 걸까요?

<br/>

### 스크랩 데이터를 못 불러왔던 이유

스크랩 데이터가 다른 데이터와 다른 점은 로그인 여부에 따라 값이 결정된다는 점입니다. 다른 말로 하면, 로그인을 해야만 true로 설정될 수 있는 데이터입니다.
분명 스크랩을 했는데도 isScrapped가 false로 온다는 것은 로그인 여부가 제대로 전달되지 않았기 때문이라는 결론을 내릴 수 있었습니다.

<br/>

### 로그인 여부가 제대로 전달되지 않은 이유
그렇다면 왜 로그인 여부가 제대로 전달되지 않고 있는 것일까요? 결론부터 말하자면, 이는 우리 프로젝트의 로그인 인증 방식과 Next.js 프레임워크의 특성에서 비롯되었다고 말할 수 있습니다.
현재 이 프로젝트에서는 인증정보가 담긴 JWT 토큰을 쿠키에 담아 백엔드와 주고받고 있습니다. 즉, 인증 정보가 쿠키라는 매개체를 통해 전달되고 있는 상황입니다. 
이것이 왜 문제가 될까요?
<br/>
Nextjs에서 tanstack query로 initialData를 이용해 서버 사이드 렌더링을 할 때는 Nextjs의 자체서버에서 데이터를 호출하게 되는데요.
바로 이 지점에서 문제가 발생합니다. Next.js는 자체 서버를 사용해 SSR(Server-Side Rendering)을 수행하는데, 쿠키는 브라우저에만 존재하는 개념이기 때문에, 쿠키로 주고받아지는 토큰에 접근할 수 없습니다.
결국, SSR을 수행할 때 쿠키를 읽지 못하므로 로그인과 관련된 isScrapped 값이 실제로는 true임에도 불구하고, 잘못된 false 값을 반환했던 것입니다.

### 해결책 : 쿠키를 헤더에 직접 담아 전송하기
그렇다면 이 문제를 어떻게 해결할 수 있을까요?
SSR에서도 쿠키를 제대로 전달할 방법은 쿠키를 헤더에 직접 담아 전송하는 것입니다.

우리가 SSR을 시도한 이 컴포넌트는 서버 컴포넌트여서 cookies()를 이용해 cookie를 직접 가져올 수 있습니다.
cookies()를 통해 가져온 쿠키를 서버컴포넌트에서 데이터를 호출하는 함수마다직접 박아주면 인식하지 못 했던 쿠키를 인위적으로 얻게 되기 때문에, 로그인 여부를 알 수 있게 됩니다.

```js
import { cookies } from "next/headers";

import {
  fetchReadingPreview,
  fetchListeningPreview,
} from "@/api/queries/contentsQueries";

import HomePageClient from "../../components/HomePageClient";

export default async function HomePage() {
  const initialReadingContents = await fetchReadingPreview({
    Cookie: cookies().toString(),
  });
  const initialListeningContents = await fetchListeningPreview({
    Cookie: cookies().toString(),
  });

  return (
    <HomePageClient
      initialReadingContents={initialReadingContents}
      initialListeningContents={initialListeningContents}
    />
  );
}
```

<br/>
![initialData에cookie박아넣은후_main](https://github.com/user-attachments/assets/fa1b25b0-ef66-4587-b61c-a9c649b85335)
<br/>

이제 쿠키가 잘 전달되어 로그인 정보가 제대로 반영되었고, 사이트에 들어가자마자 미리 스크랩 아이콘이 색칠되어있는 것을 확인할 수 있습니다. 

<br/>

간단하다면 간단한 방법으로 문제가 해결되었지만, 뭔가 석연치 않은 느낌이들 수도 있습니다. 분명 다른 컴포넌트에서도 SSR을 사용할 일이 꽤 있을텐데, 그때마다 이렇게 cookie를 직접 박아주는 것은 분명 번거로운 방법이라는 생각이 듭니다.

사실, 쿠키를 사용해서 로그인 정보를 주고받는 것은 브라우저를 사용하는 웹 개발에서나 가능한 방법이지, 브라우저를 사용하지 않는 앱개발의 경우에는 이 쿠키 방식을 사용할 수가 없습니다.
그렇기 때문에, 보통은 웹 개발에서도 bearer토큰 방식을 이용해 로그인 정보를 주고받는 것이 일반적이라고 합니다.

<br/>

지금은 백엔드와 토큰을 주고받는 방식을 쿠키로 정하여 프로젝트가 진행되고 있는 상황이어서 로그인 방법을 migration하기에는 쉽지 않은 일이지만, 추후 로그인 방식은 쿠키를 사용하지 않는 방법으로 변경할 필요는 있어보입니다.
이제서야 “토큰을 쿠키에 담아 주고받는 방식은 권장되지 않는다”는 주장이 왜 주류인지 이해하게 되었습니다. 더불어 쿠키와 토큰이라는 다양한 인증방식의 동작원리를 조금 더 제대로 공부하고 프로젝트에 적용해야겠다는 반성이 들기도 하네요. 프로젝트 마무리 시기에 조금 더 시간이 남는다면 쿠키방식에서 토큰을 직접 주고 받는 방식으로 꼭 변경을 해보아야겠습니다.



