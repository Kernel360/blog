---
layout: post
title: "로그인 폼 UX 향상을 위해 고민해보기 (react-hook-form, server action, loading ui)"
author: "심정아"
banner:
  image: "https://github.com/user-attachments/assets/335c8714-1ed7-472e-a42c-055e27e2ce23"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  tags:
    ["Next.js", "useState", "react-hook-form", "server action", "loading", "ui", "zod"]
---

### 구현하고자 하는 것

react-hook-form과 zod를 이용하여 로그인 폼 작성과 검증  
서버액션으로 로그인 요청  
로그인 버튼을 눌러 응답을 받기 전까지 로그인 폼 비활성화  
폼 제출 중 제출 버튼에 스피너 ui를 보여주기

### 1. 로딩 상태 관리

#### 1-1. useState

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

#### 1-2. react-hook-form의 formState

```tsx
export function LoginForm() {
  const form = useForm<z.infer<typeof signInSchema>>({
    // ... 로그인 폼 ...
  });

  const { isSubmitting } = form.formState; // 리액트 훅 폼에서 제공하는 (제출중) 로딩 상태

  return (
    <Button disabled={isSubmitting} type="submit">
      {isSubmitting ? (
        <>
          <Loader2 className="animate-spin" />
          로그인 중...
        </>
      ) : (
        "로그인"
      )}
    </Button>
  );
}
```

#### 1-3. server action과 useFormStatus

> React 18에서 도입된 서버 컴포넌트에서 비동기 폼 처리 상태를 관리하는 데 사용  
> 폼 제출 중, 폼 제출이 완료되었을 때 등의 상태를 추적

```tsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus(); // 서버액션에서 제공하는 로딩 상태

  return (
    <Button disabled={pending} type="submit">
      {pending ? (
        <>
          <Loader2 className="animate-spin" />
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
    <form action={handleLoginAction}>
      {/* ... 로그인 폼 ... */}
      <SubmitButton />
    </form>
  );
}
```

로그인의 경우 서버 인증이 필수적이며 결과에 따른 상태 처리가 중요하기 때문에 서버액션과 더 잘 결합해서 상태를 추적하는 useFormStatus가 더 나은 선택이라고 생각했다.

2. 실시간 유효성 검사

```tsx
const form = useForm<z.infer<typeof signInSchema>>({
  resolver: zodResolver(signInSchema),
  mode: "onChange", // 입력값이 변경될 때마다 유효성 검사
});
```

onChange 모드로 설정하면 입력값이 변경될 때마다 유효성 검사를 실시간으로 수행한다. 다 입력해서 제출할 때 유효성 경고가 발생하는 것보다 훨씬 나은 경험을 줄 수 있다.

3. 접근성 처리

폼 필드로 잘 UI를 작성했다면 접근성 처리는 어렵지 않다.  
탭 키로 각 필드를 이동할 수 있는지, 엔터로 폼을 제출할 수 있는지 등을 꼭 체크해야 한다.

4. 성공 및 에러 처리

```tsx
async function onSubmit(values: z.infer<typeof signInSchema>) {
  const res = await loginAction(values);
  if (res.success && res.user) {
    setUser({ ...res.user });

    toast.info(`반갑습니다, ${res.user.name}님`);
    const searchParams = new URLSearchParams(window.location.search);
    const redirectTo = searchParams.get("redirect_to") || "/dashboard";
    redirect(redirectTo);
  } else {
    toast.error(res.message);
  }
}
```

이번 프로젝트에서는 sonner의 toast 라이브러리를 이용했다.
존재하지 않는 아이디거나 비밀번호가 맞지 않다는 에러를 백엔드에서 정의해준 메시지를 이용해서 안내하도록 했다. 백엔드가 안내메세지를 리턴하지 않는다면 응답 코드에 따라 안내메세지를 제공하도록 커스텀 할 수도 있을 것이다.

## 결론

간단한 로그인 폼이지만 어떻게 로딩상태를 추적할지 고민해 볼 수 있었고, 더 매끄러운 사용자 경험을 위해 구현해 볼 수 있는건 무엇인지 고민할 수 있었다. 서비스 구현 경험이 적은 프론트엔드 개발자들에게 조금이나마 고민해 볼 수 있는 주제가 되었으면 좋겠다.
