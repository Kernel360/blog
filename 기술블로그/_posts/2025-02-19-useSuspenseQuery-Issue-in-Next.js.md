---
layout: post
title: "Next.js에서 useSuspenseQuery 사용 시 발생하는 SSR 이슈 상황"
author: "심정아"
categories: "기술블로그"
banner:
  image: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTq3Lba7HhrxkJyr_64dq6SeXaOpNFbSrv8Zg&s"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Next.js", "useSuspenseQuery", "SSR", "Suspense"]
---

# Next.js에서 useSuspenseQuery 사용 시 발생하는 SSR 이슈 분석

## 문제 상황

```
Error: Switched to client rendering because the server rendering errored
```

데이터 페칭시 공통적으로 사용하는 axiosClient에서 throw 하고 있던 에러.
서버렌더링시 문제라는 메세지를 보고 서버사이드 페칭로직들을 하나씩 클라이언트로 전환하며 문제지점을 확인하려 했다.
전체 페칭로직을 클라이언트로 전환했지만 문제가 해결되지 않았다.
그러던 중 `useSuspenseQuery` 사용 시 문제가 발생하는 것을 확인했다.

## useSuspenseQuery를 선택한 이유

다음과 같은 이유로 `useSuspenseQuery`를 사용하고 있었다 :

1. 로딩 상태 관리의 간소화
2. 초기 데이터 설정의 불필요
3. Error Boundary와의 통합 가능성

예시) 기존의 `useSuspenseQuery` 사용 방식:

```tsx
"use client";

export default function ProjectSection() {
  const { data, isLoading } = useSuspenseQuery({
    queryKey: getGetMyProjectQueryKey(),
    queryFn: () => getMyProject(),
  });

  if (!data?.myProjects) {
    throw new Error("프로젝트 목록을 불러오는데 실패했습니다.");
  }

  return (
    <ProjectList projects={data.myProjects} title="내 프로젝트" />
    //..다른 컴포넌트들
  );
}

// 상위 컴포넌트
function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <ErrorBoundary fallback={<Error />}>
        <ProjectList />
      </ErrorBoundary>
    </Suspense>
  );
}
```

## 원인 추측

Next.js에서 useSuspenseQuery를 실행시 서버에서 fetch를 시도하고 api 응답을 기다림 -> SSR 실패 -> CSR로 전환

: 그럼 그냥 클라이언트에서 렌더링 하도록 수정하자

## 해결 시도

### 1. Dynamic Import 시도

React.lazy와 Next.js의 dynamic import를 사용해 데이터 페칭 컴포넌트 코드 스플리팅 시도.

```tsx
const UserProfile = dynamic(() => import("./UserProfile"), {
  ssr: false,
  loading: () => <Loading />,
});
```

결과: 문제 해결 실패.

### 2. 최종 해결책: useQuery로 전환

`useSuspenseQuery`를 제거하고 일반적인 `useQuery`로 전환하여 문제 해결.

```tsx
"use client";

export default function ProjectSection() {
  const { data, isLoading } = useQuery({
    queryKey: getGetMyProjectQueryKey(),
    queryFn: () => getMyProject(),
  });

  if (isLoading) {
    return <ProjectListSkeleton />;
  }

  if (!data?.myProjects) {
    return <div>프로젝트를 불러오는데 실패했습니다.</div>;
  }

  return (
    <div className="space-y-6">
      <ProjectList projects={data.myProjects} title="내 프로젝트" />
    </div>
  );
}
```

## 교훈 및 향후 계획

React의 Suspense와 같은 최신 기능들은 강력하고 편리하지만, SSR 환경에서는 아직 완벽하게 지원되지 않는 경우가 있다고 생각된다.

명확한 이유와 동작원리를 파악하지도 못했고, 문제가 된 useSuspenseQuery를 제거하는 방식으로 해결해서 아쉬움이 남는다. 비슷한 에러를 겪는 상황이라면 useSuspenseQuery를 의심해 보라는 의견 정도를 남긴다.
리팩토링 진행하면서 꼭 관련 개념과 이슈들을 제대로 파악해서 후속 포스트를 남기고 싶다.
