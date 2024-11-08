---
layout: post
title: "Optimistic update 적용하기"
author: 양하연
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["optimistic-update", "기술블로그"]
---

<!-- 북마크, 하이라이팅, 형광펜 용어 통일 필요 -->
<!-- 영어 대문자 소문자 통일 -->
<!-- 수정 필요 = -->

## 문제상황, 버튼을 눌렀는데 바로 하이라이팅이 안 되는데?

저희 프로젝트에는 북마크를 추가하면 문장에 하이라이트 색깔이 입혀지는 기능이 있습니다. 그런데 아래 사진을 확인하면 알 수 있듯이, 형광펜 버튼을 누르자마자 하이라이팅이 생기지는 않습니다.

</br>

<p align="center"><img src="https://github.com/user-attachments/assets/b1d65973-b0d7-40bb-b7f0-8c630f69ec73" ></p>

형광펜을 설정하는 api가 성공 요청을 보내야만 하이라이팅이 칠해지는 과정을 가지고 있습니다. api응답이 중간에 실패할 수도 있으니 성공(success)이라는 응답을 받고나서야 하이라이팅이 적용되어야하는게 어떻게 보면 아주 당연한 작업이기도 하죠. 그러나 사용자에게 보여주는 화면에서 이렇게 응답이 올 때까지 기다리는 것은 많은 경우에 사용자경험을 해칠 수도 있는 요소입니다.

</br>

그렇다면 이렇게 사용자가 누른 동작에 비해 이에 대한 응답이 느린 이런 문제는 어떻게 해결하는 것이 좋을까요? api응답이 느리다는 관점에서 문제 해결에 접근해 볼수도 있겠지만, 와이파이라는 준수한 네트워크 환경에서 api를 날리는데도 1초에 가까운 시간에 걸리는 건 더이상 이 시간차이 문제는 네트워크환경을 향상시킨다고 해서 나아지는 문제는 아니라는 생각이 듭니다. 또한, 열악한 네트워크 환경에서 api요청을 날리게 되는 경우도 있을텐데 버튼에 대한 반응을 사용자에게 넘기는 것은 좋지 않다는 생각이 듭니다. 그렇다면 대체 어떻게 해결할 수 있을까요?

</br>

## 문제해결을 위한 핵심은 빨리 오는 것처럼 '보이는 것'

사실 우리에게 중요한 건 api요청에 대한 응답이 실제로 빨리 오는 것이 아닌, 그럴듯하게 보이면 되는 것입니다. 실제로 api 응답이 빠르게 오지 않더라도 버튼을 누르자마자 바로 변하는 것처럼 보이면 되는 것이죠.
이런 점을 잘 이용한 예시가 스켈레톤입니다.

<p align="center" ><img src="https://miro.medium.com/max/700/0*nS0d--Obi0NRNtu6.gif" alt="출처 : https://ui.toast.com/weekly-pick/ko_20201110" height="300px"></p>
스켈레톤은 데이터 응답이 오기 전까지 화면에 띄워질 컨텐츠의 틀만 미리 그림으로 보여주는 방식으로 사용자가 데이터를 받기까지의 지루한 상황을 극복할 수 있게 도와줍니다. 스켈레톤처럼 눈속임을 해서 실제로 버튼을 누르자마자 화면에 바로 하이라이트가 되게 보이게 만들려면 어떻게 하는 게 좋을까요?

</br>

## 눈속임을 가능하게 하는 Optimistic Update

이는 Optimistic Update라는 개념을 이용해서 해결할 수 있습니다. Optimistic Update는 낙관적 업데이트라는 이름 그대로, api요청이 성공할 것이라는 낙관적인 기대를 바탕으로 미리 데이터를 업데이트 하는 것입니다. 미리 '데이터'를 업데이트한다는게 어떻게 가능할까요? 진짜 데이터가 아닌 캐시 데이터를 업데이트 해주면 됩니다. 저희 프로젝트에서는 Tanstack query를 사용하는데, Tanstack query는 각 api마다 고유한 키를 설정해서 그 키에 해당하는 데이터는 일정시간 이상동안 직접 api를 호출하지 않고 이 캐시 데이터를 화면에 보여주는 식으로 동작합니다. 이렇게 캐시 데이터를 사용하는 Tanstack Query에서 Optimistic Update는 어떻게 적용할 수 있을까요? 아래 코드를 봅시다.

</br>

원래의 코드입니다. 북마크 조회 훅이 `queryKey: ['bookmarks', contentId],`를 가지고 있다는 것만 봐주시면 됩니다. createBookmark라는 요청이 성공하고나면, `queryClient.invalidateQueries({ queryKey: ['bookmarks', contentId] });` 를 통해 북마크 캐시를 무효화하는 작업, 즉 다시 query를 날려 북마크 목록을 새로 조회하는 api를 날리는 작업을 수행합니다. 즉 새롭게 생성한 북마크가 반영된 북마크리스트를 다시 조회하게 되어, 화면에 새롭게 생성한 북마크에 형광펜이 칠해지게 됩니다. 동작에는 아무 문제가 없으나 onSuccess이후에 invalidate작업을 하기 때문에 버튼을 누른 다음에 북마크가 생기기까지의 시간이 걸리는 것이 그대로 보여지고 있습니다.

</br>

```js
// 북마크 조회 훅
export const useFetchBookmarksByContendId = (contentId: number) => {
  return useQueryLoginOnly<BookmarkByContentIdResponse>({
    queryKey: ['bookmarks', contentId],
    queryFn: () => fetchBookmarksByContentId(contentId),
  });
};

//...생략

// 북마크 생성 훅
export const useCreateBookmark = (contentId: number) => {
  const queryClient = useQueryClient();
  return useMutation<
    Bookmark,
    Error,
    { sentenceIndex: number; wordIndex?: number; description?: string }
  >({
    mutationFn: (newBookmark) => createBookmark(contentId, newBookmark),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['bookmarks', contentId] });
    },
  });
};
```

이 코드를 optimistic update를 적용하면 형광펜 버튼을 누르자마자 바로 하이라이팅이 되도록, 정확하게 말하면 바로 되는 것처럼 '보이는' 코드를 만들 수 있습니다.
코드가 길고 복잡해보이지만 핵심은 간단합니다.
캐시 데이터만 미리 update해서 UI에 빠르게 반영된 것처럼 보여주고, api를 뒤늦게 호출하여 실제 데이터는 나중에 업데이트하는 것입니다.

아래는 이 Optimistic update가 적용된 코드입니다.

</br>

```js
// 북마크 생성 훅 : optimistic update적용
export const useCreateBookmark = (contentId: number) => {
  const queryClient = useQueryClient();

  return useMutation<
    Bookmark,
    Error,
    { sentenceIndex: number; wordIndex?: number; description?: string }
  >({
    mutationFn: (newBookmark) => createBookmark(contentId, newBookmark),
    // mutation이 호출될때 실행
    onMutate: async (newBookmark) => {
      await queryClient.cancelQueries({ queryKey: ['bookmarks', contentId] });
      // 이전 value를 snapshot해서 저장
      const previousBookmarks =
        queryClient.getQueryData<BookmarkByContentIdResponse>([
          'bookmarks',
          contentId,
        ]);
      // 캐시에 optimistically update
      queryClient.setQueryData(['bookmarks', contentId], {
        ...previousBookmarks,
        data: {
          bookmarkList: [
            ...(previousBookmarks?.data.bookmarkList || []),
            newBookmark,
          ],
        },
      });
      return { previousBookmarks };
    },

    // 에러가 발생할 경우 onMute에서 보관한 캐시데이터가 있다면 복원
    onError: (err, newBookmark, context) => {
      const ctx = context as { previousBookmarks?: BookmarkByContentId[] };

      if (ctx.previousBookmarks) {
        queryClient.setQueryData(
          ['bookmarks', contentId],
          ctx.previousBookmarks,
        );
      }
    },

    // 성공 여부와 관계없이 항상 refetch
    onSettled: () => {
      queryClient.invalidateQueries({
        queryKey: ['bookmarks', contentId],
      });
    },
  });
};

```

</br>

Optimistic Update라는 개념을 사용하면서 주의해야할 점은, api호출이 성공할 것을 낙관적으로 기대하고 미리 캐시 데이터를 업데이트 하는 것이기 때문에 실패할 경우 낙관적으로 업데이트 했던 데이터를 원상복구해줘야한다는 점입니다.
Tanstack Query에서 Optimistic Update를 수행하면서 이를 방지하기 위해 3단계로 과정을 만들어주고 있습니다.

mutationFn에서는 수행하고 싶은 api를 호출하는 createBookmark를 실행합니다. 이 mutation이 호출된 직후, onMutate핸들러가 호출됩니다.
Optimistic Update를 정리한 구문에서 캐시데이터를 미리 바꾸고, 실제 api호출은 나중에 처리한다고 했었죠?
onMutate핸들러에서 이 api호출이 실행되기 전에 캐시 데이터를 미리 저장해서 나중에 error가 발생해서 실패했을 때 다시 원상태로 복구해주기 위한 과정을 진행합니다. 그리고 error가 발생했을 때는 onMutate에서 혹시모를 상황을 위해 미리 저장했던 이전데이터로 원상복구를 진행해줍니다.
이 단계에서 캐시 데이터를 미리 업데이트하는 허점을 극복해줄 수 있는 것입니다.
그리고 onSettle핸들러는 성공여부와 관계없이 항상 실행되는데, 여기서 업데이트된 캐시데이터와 실제 데이터를 update하기 위한 refetch작업이 수행됩니다.

</br>

코드가 길어보이지만 세 단계로 정리할 수 있습니다. 이 순서대로 호출됩니다.

1. onMutate : cancelQueries로 api호출 바로 날라가지 않도록 취소. 이전 캐시데이터를 저장하고 캐시 데이터 새롭게 업데이트.
   여기서 취소된 api는 onSettle에서 뒤늦게 호출됨.
2. onError : error가 발생하면 onMutate에서 저장한 이전 캐시데이터로 복구
3. onSettle : 성공or에러 여부 상관없이 refetch ( 처음에 취소됐던 api호출이 이 단계에서 수행됨)

</br>

## 이제 버튼 누르자마자 하이라이팅이 적용되는 것처럼 보이게 되었습니다!

이 코드를 통해서는 아래 사진처럼 형광펜 버튼을 누르자마자 하이라이팅이 칠해지는 것을 확인할 수 있습니다.

<p align="center"><img src="https://github.com/user-attachments/assets/f3e5e372-b028-4d75-993b-ea34697407e7" ></p>

개발자도구에서 api호출까지 살펴보면, bookmark 목록을 조회하는 api가 네트워크 탭에서 success되기 전인 pending상태에 이미, 화면 속 하이라이트가 생기는 것을 확인할 수 있습니다.
실제 북마크를 등록하는 api의 응답속도가 빨라진 것은 아니지만, 북마크 자체가 훨씬 빠르게 등록되는 것처럼 보임으로써 사용자 경험을 개선할 수 있게 되었습니다.

</br>

## 정리

Optimistic Update를 통해 API 응답 속도와 관계없이 사용자에게 빠르게 업데이트된 것처럼 보이도록 처리할 수 있습니다. 이 방법은 API 성공을 가정하고 캐시 데이터를 미리 업데이트해 UI에 즉시 반영하며, 실패 시 캐시를 원래 상태로 복구합니다. Tanstack Query에서는 onMutate, onError, onSettled의 3단계로 이 프로세스를 구현해 사용자 경험을 개선할 수 있습니다. Optimistic Update를 통해 사용자 경험을 향상시키는 프론트엔드 개발을 이어나가 봅시다!

## 참고 문서

https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates#updating-a-single-todo
