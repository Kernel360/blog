---
layout: post
title: "Next.js 로그인 로딩 ui를 만들어보자 (useState, react-hook-form, server action)"
author: "심정아"
categories: "프론트엔드 기술블로그"
banner:
  image: "https://github.com/user-attachments/assets/335c8714-1ed7-472e-a42c-055e27e2ce23"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  tags:
    ["Next.js", "useState", "react-hook-form", "server action", "loading", "ui"]
---

### 기존 구현 사항

react-hook-form과 zod를 이용하여 로그인 폼 작성
서버액션으로 로그인 요청

### 구현하고자 하는 것

로그인 버튼을 눌러 응답을 받기 전까지 로그인 폼 비활성화와 버튼에 스피너 ui를 보여주기

### 1. useState

```tsx
const [isLoading, setIsLoading] = useState(false);

async function handleLoginAction(formData: FormData) {
  setIsLoading(true);
  try {
    const res = await loginAction(formData);
    if (res.user) {
      const searchParams = new URLSearchParams(window.location.search);
      const redirectTo = searchParams.get("redirect_to") || "/dashboard";
      redirect(redirectTo);
    }
  } catch (error) {
    // 에러 처리
  } finally {
    setIsLoading(false);
  }
}
```

가장 먼저 떠올릴 수 있는 간단한 방식이다.  
로딩 상태를 명시하기 위해 별도의 상태를 따로 만드는 방식이 최선은 아니라고 생각했다.

### 2. react-hook-form의 formState

```tsx
export function LoginForm() {
  const form = useForm<z.infer<typeof signInSchema>>({
    // ... 로그인 폼 ...
  });

  const { isSubmitting } = form.formState; // 리액트 훅 폼에서 제공하는 (제출중) 로딩 상태

  return (
    <Button disabled={isSubmitting} type="submit" className="w-full">
      {isSubmitting ? (
        <>
          <Loader2 className="mr-2 h-4 w-4 animate-spin" />
          로그인 중...
        </>
      ) : (
        "로그인"
      )}
    </Button>
  );
}
```

### 3. server action과 useFormStatus

> React 18에서 도입된 서버 컴포넌트에서 비동기 폼 처리 상태를 관리하는 데 사용  
> 폼 제출 중, 폼 제출이 완료되었을 때 등의 상태를 추적

```tsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus(); // 서버액션에서 제공하는 로딩 상태

  return (
    <Button disabled={pending} type="submit" className="w-full">
      {pending ? (
        <>
          <Loader2 className="mr-2 h-4 w-4 animate-spin" />
          로그인 중...
        </>
      ) : (
        "로그인"
      )}
    </Button>
  );
}

export function LoginForm() {
  return (
    <form action={handleLoginAction} className="space-y-6">
      {/* ... existing form fields ... */}
      <SubmitButton />
    </form>
  );
}
```

로그인의 경우 서버 인증이 필수적이며 결과에 따른 상태 처리가 중요하기 때문에 서버액션과 더 잘 결합해서 상태를 추적하는 useFormStatus가 더 나은 선택이라고 생각했다.
