---
layout: post  
title: "공통 컴포넌트 구현 고민"
author: "김난아"
categories: "기술블로그"
banner:
  image: ![스플래시](https://github.com/user-attachments/assets/dda9909e-720b-4e8f-957d-ac4f1986d2cf)
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`공통컴포넌트`, `컴포넌트`]
---

내가 작업하기로 한 공통컴포넌트는 `Input, Button, Table` 컴포넌트.
나머지는 페이지에 들어가는 컴포넌트들 `ListItem, VehicleRegisterForm, InspectionStatusModal`
그외 자잘하고 없어진 컴포넌트들도 존재한다.
이번 프로젝트를 하면서 설정한 목표는 재사용성과 확장성을 고려한 공통 컴포넌트 구현하기.
그래서 최대한 라이브러리를 사용하지 않고 직접 구현하기로 했다.

## Input

![](https://velog.velcdn.com/images/nanafromjeju/post/d00d92ca-5bf4-4158-a4a7-03d435d94ea1/image.gif)

![2025-01-041 22 09-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/9b4608d6-62bb-41ac-b629-290b92eef4c0)

`Input`을 만들어보는 건 이번이 두 번째.
기존에 만들어본 경험이 있어 저번 프로젝트를 참고하면서 만들었다.
먼저 기본적인 형태만 넣어둔 `BaseInput`을 만들고 그걸 토대로 `SearchInput`을 만들고 `Message`라는 컴포넌트를 만들었다.
`Message`는 *error*와 *success*라는 카테고리로 나누고, _constants_ 파일에서 관리하도록 했다.

처음에 `Input` 컴포넌트를 만들 때 테스트를 하려 했는데 스토리북과 로컬서버가 꺼지는 이슈가 발생했다..
알고 보니 `input`이 아닌 `Input`자체를 불러와서 무한루프 발생..ㅎ
트러블슈팅에 넣으려다가 팀원분이 이건 트러블슈팅도 아닌 그냥 트러블이라고...

![](https://velog.velcdn.com/images/nanafromjeju/post/4b8900f5-e702-40aa-943a-62000f9a0f6c/image.png)

## Button

| BaseButton                                                                                                                      | RoundButton                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| <img width="350px" height="400px"  src="https://github.com/user-attachments/assets/04bf1caa-9554-4a90-997d-6d1ae6848c0c"></img> | <img width="350px" height="400px" src="https://github.com/user-attachments/assets/24eea76a-238e-4493-afc5-765d0a3766bf"></img> |

이번에 처음 도전해 보는 `Button` 컴포넌트👀
디자인상 네모난 버튼과 둥근 버튼이 있어서, 기본이 되는 `BaseButton`을 네모난 형태로 먼저 만들고 `BackButton`, `RoundButton`과 `SquareButton`을 만들었다.
스토리북에는 디자인상 없는 disabled속성과 읽기 전용을 추가해뒀다.
확장성을 사용하면 사이즈를 타입별칭으로 받아오는 게 상당히 불리하지만
페이지나 다른 형태의 디자인이 늘어날 가능성이 굉장히 적거나,
아예 없게 만들 수 있기 때문에 (개발자가 디자인할 때 장점! 마음대로 기획 가능)

- 추후에 기획에는 없던 `ExcelButton`과 `LinkButton` 이 추가되었고,
  `LinkButton`의 경우는 Next의 공식문서를 참고해서 만들었다.

```ts
// 참고 공식문서
import Link from "next/link";

export default function Home() {
  return <Link href="/dashboard">Dashboard</Link>;
}
```

`ExcelButton`의 경우 엑셀 아이콘만 들어가고 onClick을 받는 버튼으로 만들었는데,
사실상 엑셀버튼의 역할은 하나뿐이라 기능도 같이 넣어버리는 게 좋을 것 같다는 조언을 받았다.
_리팩토링 때 실행해 볼 예정!_
![](https://velog.velcdn.com/images/nanafromjeju/post/97b66c7d-8f6b-434c-ba36-6216ac0904e2/image.png)

## Table

![](https://velog.velcdn.com/images/nanafromjeju/post/c176edbe-f1a9-431a-ac59-75cf65e8ae68/image.gif)

이번 프로젝트 내 파트에서 핵심이자 가장 고민 많이했던 `Table` 컴포넌트.
먼저 슈도코드를 작성하고 필수조건을 생각해봤다.

> **<슈도코드 작성>**

1. 테이블의 가로칸과 세로칸의 숫자를 설정한다.
2. 넓이도 자유자재로 설정 할 수 있다.
3. 안에는 텍스트를 입력하고 굵기를 조절할수 있다. (텍스트 스타일을 2개의 변수로 만들고 설정할 수 있도록 세팅)
4. 안에는 텍스트뿐만이 아닌 컴포넌트를 불러올 수 있다.

> **<테이블의 조건>**

1. 가로줄 (최소2, 3, 4, 5, 최대9)
2. 세로줄 (무한대로 늘어나야 됨)
3. 칸 안에 백그라운드 컬러를 마음대로 바꿀 수 있어야 한다.
4. 안에 텍스트의 굵기는 semibold 아니면 regular 둘 중에 하나
5. 안에는 텍스트뿐만이 아닌 뱃지 컴포넌트가 들어갈수도 있다.
6. 넓이를 자유자재로 설정할 수 있어야 한다. (세로에서 2번째 칸은 가로 넓이가 두 칸 이런 형식으로)

디렉터님께서 TanStack Table을 추천해 주셔서 사용해 봤는데 실패..
장점으로는 제공하는 훅들이 많아서 사용해보려 했는데 어려워서 포기했다.
다음에 코드에 대한 이해도가 높아졌을 때 다시 재도전 해보면 좋을 것 같다.

> **TanStack Table이 제공하는 함수**

- `getGroupingRowModel()`: 행 그룹핑
- `getSortingRowModel()`: 정렬
- `getFilteredRowModel()`: 필터링
- `getPaginationRowModel()`: 페이지네이션
- `getCoreRowModel` 은 TanStack Table의 핵심 기능 중 하나로, 테이블의 기본적인 행(row) 모델을 생성하는 함수 (데이터 배열을 테이블 행으로 변환, 컬럼 정의에 따라 각 셀의 값을 처리, 테이블의 기본 구조를 생성)

그래서 기본적으로 html에서 제공하는 테이블을 사용했다!
처음에는 이해가 어려웠는데 사용하다보니 감이 왔다.

`colspan`은 열끼리 병합되지 않고 행끼리 병합
`rowspan`은 아래방향인 열끼리 병합

#### TanStack Table을 사용 이유🌟

> 1. **테이블 상태 관리**

    - 정렬, 필터링, 페이지네이션 등의 기능을 쉽게 추가할 수 있으며 테이블의 상태를 손쉽게 관리 가능

2. **타입 안정성**
   - TypeScript와 완벽하게 통합되어 타입 안정성 보장
3. **성능 최적화**
   - 가상화(virtualization)와 같은 성능 최적화 기능을 쉽게 추가할 수 있으며 큰 데이터셋도 효율적으로 처리 가능
4. **확장성**
   - 필요한 경우 정렬, 필터링, 그룹핑 등의 추가 기능을 쉽게 구현

https://velog.io/@roong-ra/HTML-%ED%91%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0-table-%EA%B4%80%EB%A0%A8-%ED%83%9C%EA%B7%B8
👆참고 블로그

## ListHeader, ListItem

![2025-01-094 00 59-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/1e0e3d5f-e222-4bec-b703-f6a9c69949dd)

처음 만들었을 때 진짜 퍼블리싱만 해서 Mark님께 한소리 들었던 컴포넌트..
처음에 `ListHeader`같은 경우는 바뀌지 않는 거라 하드코딩으로 넣어놨는데
확장성을 고려하면 별로 좋지 않은 것 같다는 리뷰를 받고 전체를 뜯어고쳤다.
기존에 설계했던 그림은 두 개의 컴포넌트를 하나의 컴포넌트로 만들어 사용하고 싶었는데
능력+지식 부족으로 두 개의 컴포넌트를 하나에 구겨 넣지도 못하는(?) 설계가 되어버렸음🤯
조언을 얻은 결과 배열로 변환하여 map을 사용하는 방식으로 바꿨다!

> **구조 분해 할당 사용 이유:**
>
> 1. 코드가 더 명시적이고 읽기 쉬움 (각 필드가 어떤 데이터인지 바로 알 수 있음)
> 2. TypeScript의 타입 체크의 정확도가 높다
> 3. 각 필드별로 다른 스타일이나 추가 로직을 적용하기 쉬움

## VehicleRegisterForm

![2025-01-0911 40 16-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/cdd43fc5-c058-4e7a-a11f-d117c5cc0bfc)

기존에는 컴포넌트를 중복되는 구조로 만들었는데,
props로 변경하니 훨씬 간결해지고 중복코드도 사라졌다.

🌟 **옵셔널 props 사용에 대한 고민:**

초기에는 컴포넌트의 유연성을 위해 모든 `props`를 옵셔널(?)로 설정했으나,
실제 사용 시 어떤 `props`가 필수값인지 파악하기 어려운 상황이 발생했다.
TypeScript의 타입 체크의 주요 목적은 코드의 안정성과 명확성을 높이는 것인데,
과도한 옵셔널 사용은 오히려 컴포넌트 사용 시 혼란을 초래하고, 의도하지 않은 동작 발생 위험을 높였던 것 같다.
앞으로는 명확한 타입 체크를 위해 정말 필요한 경우에만 옵셔널을 사용하고,
가능한 한 **필수 `props`를 명시적으로 지정하는 방향으로 개선**하기로 했다.🫡

1. 중복되는 레이아웃 구조를 하나로 통합
2. text(label)와 input 컴포넌트를 props로 받도록 개선

## InspectionStatusModal

![2025-01-296 06 38-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/3d001ad6-49fe-42a0-b603-4f8910eb345d)

이건 구조 설계보다 스타일링이 더 어려웠던 컴포넌트.
마찬가지로 밖에서 데이터를 넣어주고 map함수를 돌려서 사용하고 있는 구조.
아직 실제 API와 연동하지 않아서 추후 구조는 바뀔 예정.

```ts
    {
		status: 'required' as const,
        iconType: 'bell' as const,
        icon: <Image src='/icons/white-bell-icon.svg' alt='bell' width={24} height={24} />,
        title: '차량 점검 안내',
        children: '123가 4567',
        message: '차량에 대한 점검이 필요합니다.',
    }
```

## Datepicker

참고로 `Pagination`과 `Datepicker`의 경우 Mantine 라이브러리를 사용했는데
캘린더를 열면 아래처럼 아이콘이 이상하게되는 문제가 발생
알고보니 styles를 import 안해줘서 발생했던 이슈였다.
![](https://velog.velcdn.com/images/nanafromjeju/post/ab99e6ae-4839-40e7-a5ae-95f27301c4e3/image.png)

자나깨나 공식문서 잘 읽자...!
참고 문서: https://mantine.dev/styles/mantine-styles/
