---
layout: post
title: "API Cancelled 문제 해결하기"
author: 양하연
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["API-Cancelled", "기술블로그"]
---

## 1. 이런 글을 쓰게 된 배경

최근 진행한 프로젝트에서 평소처럼 form에 데이터를 보내는 Button을 누르면 API호출이 이뤄지는 로직을 구현하고 있었습니다.
그런데 이번에는 API호출이 되지 않고, 콘솔창에 `API Cancelled`라는 메시지가 뜨는 것을 확인할 수 있었습니다.
이 글에서는 API cancelled 문제가 일어난 배경과 그 문제를 해결하는 과정을 설명하려 합니다.

## 2. 구체적인 상황 및 코드

<img src="/Users/hayeon/Downloads/마이네임안바뀜.gif" alt="이름안바뀜">

<img alt="cancelled" src="![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/1ee811a0-1243-4c1a-a362-fece1e87dc36/0af29550-d840-46b6-b948-b6d32e48c6db/image.png)"/>

```js
// 앞 부분 생략...

const handleNicknameChange = () => {
  updateUserInfo({ nickname }); //tanstack query function
};

return (
  <form className="space-y-4 w-full " name="userInfo">
    <div>
      <label
        htmlFor="name"
        className="block text-sm font-medium text-gray-700 mb-1 "
      >
        닉네임 *
      </label>
      <div className="flex gap-2">
        <Input
          id="name"
          placeholder="닉네임"
          value={nickname}
          onChange={(e) => setNickname(e.target.value)}
          className="flex-1"
          autoComplete="on"
        />
        <Button
          variant="outline"
          size="sm"
          disabled={!isNicknameChanged}
          onClick={handleNicknameChange}
          // type="submit"
        >
          변경하기
        </Button>
      </div>
    </div>
    // 생략...
  </form>
);
```

## 원인 추측

### 접근1 - 로그에 찍힌 메세지를 먼저 읽어보자

![로그헤더](https://github.com/user-attachments/assets/5235fdf1-1281-4523-bd6b-b86be2bd7a83)

에러의 원인은 터미널 혹은 콘솔에 찍힌 에러를 우선적으로 보라는 말이 있듯이, 제일 먼저 로그에 찍힌 메세지를 보고 원인을 추적해보았습니다.
'provision header was shown' 과 관련된 에러 핸들링 글을 보면 adblock이라는 확장프로그램이 있는지 확인해라, CORS에러인지 확인해라, 강력 새로고침을 하여 Cache를 삭제한 후 다시 실행해라 등 여러가지 해결방법이 있었지만 제 경우에는 맞지 않았습니다. 콘솔에 찍힌 로그로는 드러나지 않은 다른 요소에 의한 문제라는 생각이 들었습니다.

### 접근2 - 백엔드 서버의 문제일까?

혹시 백엔드 서버의 문제일수도 있지 않을까 하는 마음에 swagger로 동일한 api호출을 시도해 보았는데 정상적으로 작동하는지 확인했습니다.
아무 에러 없이 request success라는 문구가 떴습니다. 이로써 이 에러는 api 자체의 오류가 아닌 내 브라우저, 혹은 내 코드에 문제가 있는 것이라는 사실로 좁혀졌습니다.
사실 별개로 서버의 문제라고 한다면 500 Internal Server Error라든가 에러코드에서 오류가 드러났어야했는데 api cancelled되는 것 말고는 에러코드가 200으로 뜨는 것을 보면 더더욱 서버와 API의 문제는 아닌 것으로 추측할 수 있었습니다.

### 접근3 - fetch 구문 문법 오류일까?

혹시 fetch구문을 문법에 맞지 않게 잘못 쓴 것일까?라는 생각에 문법을 점검해보았습니다.
다른 동료가 이미 작성해놓은 api호출문은 오류없이 정상적으로 작동하는지, 이 코드만의 문제인지 찾아보았습니다.
그나마 다른 점이 있다면 post,put,get 등 다른 http method를 이용해 api요청을 보내고 있었습니다. 이와는 다른 patch method를 사용하는 내가 혹시 이 method에 맞지 않는 헤더로 요청을 보내고있는 것은 아닌지 살펴보았지만 올바른 문법으로 잘 작성하여 문제는 없었습니다.

```js
const response = await fetch(`${BASE_URL}/user/me`, {
  method: "PATCH",
  headers: {
    "Content-Type": "application/json",
  },
  credentials: "include",
  body: JSON.stringify(userInfo),
});
```

### 접근4 - API요청이 cancelled될만한 외력이 존재하는 것일까?

API가 'cancelled'됐다는 것에 집중하여 api 요청이 보내졌지만 어떤 외부의 힘에 의해 이를 '취소'하고 있을 것이라는 관점에서 접근해보았습니다.
'과연 여기서 api요청에 가해질 수 있는 외부압력은 어떤 종류의 것일까?' 조금 더 원초적인 코드 로직이 무엇인가를 살펴보면, 해당 button을 누른다는 것은 form 태그에 종속된 input의 내용을 보내는 동작을 수행하는 것입니다.
그런데 form 태그란 어떤 동작을 하고 있을까요? form이라는 태그는 button이 눌리면 브라우저를 새로고침하면서 input의 내용을 전송하려고 합니다.
즉, submit이벤트로 인한 navigation 요청과 해당 페이지 내의 ajax 요청이 맞물리면서 기존 페이지의 ajax 요청을 취소하는 현상이 발생하고 있다는 추측을 할 수 있습니다. 과연 이 가설이 맞는지 시험해볼까요?

```js
const handleNicknameChange = (event) => {
  event.preventDefault(); // 기본 form 제출 방지
  updateUserInfo({ nickname });
};
```

handleNcknameChange에 `event.preventDefault();` 이 코드를 추가하니 정상적으로 api 호출이 이루어지는 것을 알 수 있었습니다.

![마이네임홍길동변경됨](https://github.com/user-attachments/assets/a0a22ace-5027-412e-b0e3-3f799cee8b92)

## 느낀점

이번 트러블 슈팅으로 해결한 문제는 너무나도 간단한 기초적인 개념으로 인한 문제였으나, 이를 깨닫는 과정으로는 백엔드 동작을 시험해보거나 로그에 뜨는 문제로 추측을 해보는 등 상당히 지난한 시간이었습니다. 이번 경험으로 인해 `html tag를 사용할 때 한번 더 점검하는 자세를 가지자. 내 경험과 기억에 의존해서만 개발을 진행할 때 이렇게 부메랑이 되어서 돌아올 수 있다.`라는 교훈을 얻게되었습니다.

## 참고 문서

- https://stackoverflow.com/questions/12009423/what-does-status-canceled-for-a-resource-mean-in-chrome-developer-tools
- (https://ko.javascript.info/forms-submit)
- [https://ko.react.dev/reference/react-dom/components/input#:~:text=자세히 보기-,폼 제출 시 input 값 읽기,페이지를 새로고침하며 이러한 동작은 e.preventDefault()를 호출하여 덮어쓸 수 있습니다.,-폼 데이터는 new](<https://ko.react.dev/reference/react-dom/components/input#:~:text=%EC%9E%90%EC%84%B8%ED%9E%88%20%EB%B3%B4%EA%B8%B0-,%ED%8F%BC%20%EC%A0%9C%EC%B6%9C%20%EC%8B%9C%20input%20%EA%B0%92%20%EC%9D%BD%EA%B8%B0,%ED%8E%98%EC%9D%B4%EC%A7%80%EB%A5%BC%20%EC%83%88%EB%A1%9C%EA%B3%A0%EC%B9%A8%ED%95%98%EB%A9%B0%20%EC%9D%B4%EB%9F%AC%ED%95%9C%20%EB%8F%99%EC%9E%91%EC%9D%80%20e.preventDefault()%EB%A5%BC%20%ED%98%B8%EC%B6%9C%ED%95%98%EC%97%AC%20%EB%8D%AE%EC%96%B4%EC%93%B8%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.,-%ED%8F%BC%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%8A%94%20new>)
