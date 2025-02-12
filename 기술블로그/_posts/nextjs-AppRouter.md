---
layout: post  
title: "Pages Router vs. App Router: What’s New?"
author: "전상호"
categories: "프론트 기술블로그"
banner:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["NextJS", "PageRouter", "AppRouter"]
---

# Pages Router vs. App Router: What’s New?

Next.js 13부터 새롭게 도입된 **App Router**는 기존 **Pages Router** 방식과 비교하여  
**더 유연한 파일 기반 라우팅 시스템, 서버 컴포넌트 지원, 자동 레이아웃 캐싱 등**의 기능을 제공합니다.

이 글에서는 **Pages Router와 App Router의 주요 차이점과 전환(Migration) 시 고려해야 할 사항**을 다룹니다.

---

## 1. Pages Router vs. App Router 핵심 차이점

### 🔹 Pages Router 방식 (Next.js 12 이하)
Next.js의 기존 Pages Router 방식은 **파일 기반 라우팅과 클라이언트 중심의 라우팅 구조**를 사용했습니다.

#### ✅ 특징:
- `pages/` 디렉터리 내의 **각 파일이 개별적인 라우트**로 작동
- `getStaticProps`, `getServerSideProps`를 사용한 데이터 패칭
- Next.js API Routes(`/pages/api`)를 활용한 서버 기능 제공
- **CSR(Client-Side Rendering) 기반이 기본**

#### 📌 코드 예시 (`pages/about.tsx`)
```tsx
export default function AboutPage() {
  return <h1>About Us</h1>;
}

export async function getStaticProps() {
  return { props: { data: "Static data" } };
}

export async function getServerSideProps() {
  return { props: { data: "Dynamic data" } };
}
```

---

### 🔹 App Router 방식 (Next.js 13+)
Next.js 13부터 **새로운 라우팅 시스템**인 **App Router**가 도입되었습니다.  
이 방식에서는 `app/` 디렉터리를 사용하며, **React Server Components(RSC)**와 결합된  
더 강력한 **서버-클라이언트 하이브리드 렌더링**을 제공합니다.

#### ✅ 특징:
- `app/` 디렉터리에서 **폴더 기반의 파일 라우팅**
- **React Server Components(RSC) 기본 지원**
- **Layout & Loading** 파일을 활용한 최적화된 UI 관리
- `fetch`와 `async/await`을 활용한 서버 데이터 패칭

#### 📌 코드 예시 (`app/about/page.tsx`)
```tsx
// 서버 컴포넌트 활용 (자동 서버 렌더링)
export default async function AboutPage() {
  const data = await fetch("https://api.example.com/about", { cache: "no-store" }).then((res) =>
    res.json()
  );

  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.description}</p>
    </div>
  );
}
```

#### 📌 Suspense를 활용한 로딩 처리 (app/about/loading.tsx)
```tsx
export default function Loading() {
  return <p>페이지 로딩 중...</p>;
}
```

#### 📌 클라이언트 컴포넌트 활용 (app/about/client-component.tsx)
```
"use client";

import { useState, useEffect } from "react";

export default function ClientComponent() {
  const [message, setMessage] = useState("");

  useEffect(() => {
    fetch("/api/message")
      .then((res) => res.json())
      .then((data) => setMessage(data.message));
  }, []);

  return <p>{message || "불러오는 중..."}</p>;
}
```

#### 📌 App Router 구조 예시
```
app/
 ├── about/
 │    ├── page.tsx  // 서버 컴포넌트
 │    ├── loading.tsx  // Suspense 기반 로딩 처리
 │    ├── client-component.tsx  // 클라이언트 컴포넌트
```
---

## 2. Pages Router → App Router 전환 가이드

### 🏗 주요 변경 사항
| 항목              | Pages Router (Next.js 12) | App Router (Next.js 13+) |
|-----------------|--------------------|--------------------|
| 디렉터리 구조   | `pages/`            | `app/`             |
| 라우팅 방식     | 파일 기반 라우팅    | 폴더 기반 라우팅    |
| 데이터 패칭     | `getServerSideProps`, `getStaticProps` 사용 | `fetch`, `async/await` 활용 |
| 상태 관리 방식  | 클라이언트 기반     | 서버-클라이언트 하이브리드 |
| 미들웨어 지원   | `middleware.ts` 사용 | `app/middleware.ts` 또는 API 라우트 활용 |

### 📌 기존 Pages Router 프로젝트를 App Router로 변환하는 단계
1. `pages/` 디렉터리를 `app/` 디렉터리로 이동
2. `app/layout.tsx`를 생성하여 공통 레이아웃 정의
3. `page.tsx` 파일을 생성하여 페이지 구성
4. API 라우트(`/pages/api`) 대신 **Server Actions** 활용
5. `use client` 지시어를 사용하여 클라이언트 컴포넌트로 분리

---

## 3. App Router의 장점과 개선점

### ✅ 서버 중심의 데이터 패칭
App Router는 **서버 컴포넌트를 기본으로 사용**하기 때문에  
이전보다 더 효율적인 데이터 패칭과 **SSR, ISR, SSG 조합이 가능**합니다.

```tsx
// ✅ 서버에서 직접 데이터 패칭 가능 (App Router 방식)
export default async function UserProfile() {
  const user = await fetch("https://api.example.com/user").then((res) => res.json());
  
  return <h1>{user.name}</h1>;
}
```

---

### ✅ 레이아웃(Layout)과 상태 유지
App Router에서는 **레이아웃을 자동으로 캐싱**하고, 페이지 이동 시 레이아웃을 유지할 수 있습니다.

📌 레이아웃 예시 (`app/layout.tsx`)
```tsx
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <nav>Navbar</nav>
      <main>{children}</main>
    </div>
  );
}
```

📌 중첩 레이아웃 예시
```tsx
// `app/dashboard/layout.tsx`
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <nav>Navbar</nav>
      <main>{children}</main>
    </div>
  );
}
```

이전 Pages Router에서는 **`_app.tsx`에서 글로벌 레이아웃을 관리**했지만,  
App Router에서는 각 폴더별로 **레이아웃을 계층적으로 분리**할 수 있습니다.

---

## 4. 결론

Next.js 13+에서는 **App Router를 통해 서버 렌더링을 강화하고, 클라이언트 컴포넌트와 분리된 구조**를 제공합니다.  
- 기존 **Pages Router를 사용하던 프로젝트는 단계적으로 App Router로 마이그레이션 가능**
- **폴더 기반 라우팅, Server Components, 자동 레이아웃 유지 등의 장점**을 활용 가능
- **점진적 전환을 고려하여 기존 프로젝트와 혼합 사용 가능**

🛠️ 유의사항

RSC 기반으로 변경되므로, 클라이언트 컴포넌트 사용에 주의

기존 API Routes(/pages/api 사용)가 사라지고 Server Actions로 대체

캐싱 및 상태 관리 방식이 변화하므로, React Query 또는 Zustand 같은 상태 관리 라이브러리 활용 고려

Next.js 프로젝트를 운영 중이라면,  
**App Router로 전환하면서 새로운 기능을 적극 활용하는 것**을 추천합니다. 🚀

> **🔎 참고**: Next.js 공식 문서에서 자세한 내용을 확인하세요 → [Next.js App Router Guide](https://nextjs.org/docs/app)

---
