---
layout: post
title: "Floating UI 소개"
author: "김승태"
categories: "프론트엔드 기술블로그"
banner:
  image: 썸네일로 넣고 싶은 이미지 링크
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["Floating UI"]
---

## Floating Element란?

Floating UI를 소개하기 앞서 먼저 Floating Element에 대해서 말하고자 합니다.

Floating Element는 화면의 기본적인 문서 흐름에서 벗어나서, 다른 콘텐츠 위에 배치되는 UI 요소들을 말합니다. 보통 사용자가 특정 상호작용을 할 때 동적으로 나타나며, 화면의 특정 부분에 위치해 있거나, 특정 기준 (마우스 위치, 텍스트 필드 등)에 따라 위치가 조정됩니다.

여기서 말하는 floating element의 예시는 다음과 같습니다.

- Tooltip: 마우스를 특정 요소에 올렸을 때 나타나는 간단한 설명 텍스트.
- Popover: 사용자가 클릭했을 때 더 복잡한 정보를 표시하는 작은 팝업.
- Select Menu: 사용자가 선택할 수 있는 옵션 리스트.
- Combobox: 사용자가 텍스트를 입력하면서 선택할 수 있는 드롭다운 리스트.
- Dropdown Menu: 버튼을 클릭했을 때 선택할 수 있는 옵션들의 리스트.
- Dialog: 사용자의 상호작용에 의해 나타나는 모달 창.

## Floating UI란?

Floating UI는 floating element의 배치(Positioning)과 상호작용(Interactions)를 쉽게 할 수 있게 도움을 주는 라이브러리입니다.

상호작용(focus, click, hover 등등)에 따라 위치를 계산해 원하는 위치에 floating element를 보여주는 건 물론이고 [middleware](https://floating-ui.com/docs/middleware)를 이용해 뷰포트의 변경, 스크롤, 화면 밖에 나가는 경우에 floating element의 위치를 재조정 하기도 합니다.

## React에서 Floating UI 사용법

Floating UI는 다양한 Hook을 제공합니다. 다양한 Hook에 대한 설명과 예시는 [공식문서](https://floating-ui.com/docs/react)에 자세히 나와있으며 여기에선 useFloating에 대해서만 소개하겠습니다.

### useFloating

Floating ui의 모든 Hook과 컴포넌트에 대해 컨트롤러 역할을 하는 메인 Hook입니다. Floating UI의 어떤 Hook이든 useFloating의 context를 인자를 필요로 합니다.

useFloating의 사용법은 아래와 같습니다.

```tsx
function App() {
  const { refs, floatingStyles } = useFloating();
  return (
    <>
      <button ref={refs.setReference}>Hover me</button>
      <div ref={refs.setFloating} style={floatingStyles} />
    </>
  );
}
```

`refs.setReference` 는 floating element가 참조할 기준(reference) element에 사용하는 함수입니다. 예를 들어, 툴팁이 버튼 위에 표시되도록 하려면, 이 버튼의 DOM element를 setReference로 지정해야 합니다.

`refs.setFloating` 는 floating element에 사용하는 함수입니다.

다양한 요구사항을 충족하기 위해 useFloating은 아래와 같은 다양한 return value와 option이 존재합니다. 자세한 예시는 [공식 문서](https://floating-ui.com/docs/useFloating)에 나와있습니다.

**return value**

```tsx
interface UseFloatingReturn {
  context: FloatingContext;
  // 이 context에는 관련 Hook과 컴포넌트에 제공되는 컨텍스트 데이터가 포함되어 있습니다.
  // useHover, useClick, useDusmiss 등의 다른 hook을 사용할 때 해당 context를 이용합니다.
  placement: Placement;
  // reference에 대한 floating 의 최종 배치입니다.
  // 옵션에서 전달된 것과 달리 이 옵션은 flip()과 같은 미들웨어에 의해 변경될 수 있습니다.
  strategy: Strategy; // floating element의 포지셔닝 전략입니다.
  x: number; // floating element의 최종 x 좌표입니다.
  y: number; // floating element의 최종 y 좌표입니다.
  middlewareData: MiddlewareData; // 사용된 미들웨어에서 제공하는 데이터입니다.
  isPositioned: boolean; // floating element가 아직 위치가 지정되지 않았는지 여부입니다.
  update(): void; // floating element의 위치를 수동으로 업데이트하는 함수입니다.
  floatingStyles: React.CSSProperties; // floating element에 적용해야 하는 스타일입니다.
  // inline으로 적용합니다. ex) <div style={...floatingStyles} />
  refs: {
    reference: React.MutableRefObject<ReferenceElement | null>;
    // reference로 지정한 element 입니다. ex) ref.reference.current?
    floating: React.MutableRefObject<HTMLElement | null>;
    // floating로 지정한 element 입니다
    domReference: React.MutableRefObject<Element | null>;
    // DOM reference element(사용 가능한 경우)에 대한 ref를 추가합니다.
    // 이벤트 핸들러 내부 또는 이펙트에서 액세스할 수 있습니다.
    setReference(node: RT | null): void; //  reference element를 설정하는 함수입니다.
    setFloating(node: HTMLElement | null): void;
    setPositionReference(node: ReferenceElement): void;
  };
  elements: {
    reference: RT | null;
    floating: HTMLElement | null;
  };
}
```

**options**

```tsx
interface UseFloatingOptions {
  placement?: Placement; // reference element에 대한  floating element의 배치입니다.
  // ex) 'top', 'right' 등등 setReference 의 어느 곳에 보여질지 정합니다.
  strategy?: 'fixed' | 'absolute'; // 사용할 CSS 위치 속성 유형입니다.
  transform?: boolean; // floating element의 위치를 지정하기 위해 CSS 트랜스폼을 사용할지 여부
  // 인라일 스타일을 부여해서 placement의 위치를 잡습니다.
  middleware?: Array<Middleware | undefined | null | false>;
  // floating element의 위치를 변경하는 미들웨어 객체 배열입니다.
  open?: boolean; // floating element가 열려 있는지 여부
  onOpenChange?( // open 상태가 변경되어야 할 때 호출되는 함수입니다.
    open: boolean,
    event?: Event,  // e.g. MouseEvent
    reason?: OpenChangeReason, // e.g. 'hover', 'click'
  ): void;
  elements?: {
    reference?: ReferenceElement | null;
    floating?: FloatingElement | null;
  };
  whileElementsMounted?(.
    reference: ReferenceElement,
    floating: FloatingElement,
    update: () => void,
  ): () => void;
  nodeId?: string;
}
```

### [Middleware](https://floating-ui.com/docs/middleware)

Middleware는 사용자 지정하고 원하는 만큼 세분화하여 배치(Positioning)할 수 있습니다. 내부적으로는 위치를 계산하는 로직인 [computePosition()](https://github.com/floating-ui/floating-ui/blob/master/packages/core/src/computePosition.ts)으로 위치를 계산한 뒤 미들웨어를 적용하여 최종적인 위치를 계산합니다.

Floating UI는 Middleware를 통해 floating element가 뷰포트 밖에서 생기거나, 스크롤 시에 가려져서 보여야할 것이 보이지 않는 등의 위치 조정 문제를 쉽게 해결해주는 기능을 제공합니다.

예를 들어서 `shift` Middleware는 floating element가 화면 밖으로 나가는 경우를 막기 위해 사용됩니다.

사용법은 아래와 같습니다.

```tsx
useFloating({
  middleware: [shift()],
});
```

## 출처

[공식문서](https://floating-ui.com/)

[repository](https://github.com/floating-ui/floating-ui/tree/master)
