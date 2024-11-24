## 들어가는 말

개발을 하다 보면 우리는 수많은 상황을 마주합니다. 얕게는 유저가 로그인을 했는가 아닌가. api 호출에 대한 응답이 에러인가? 로딩중인가? 성공인가? 등등 다양한 상황에 따라 이후 로직이 달라집니다. 이런 경우에 저는 if문, 삼항 연산자, AND 연산자 등을 사용하였습니다. 그러나 로직이 쌓이고 경우가 곱의 경우의 수(n x m)로 늘어난다면 이는 매우 가독성 측면에서 아쉬운 코드가 될 것입니다. 해당 문제를 고민하고 나은 방법을 찾아보고자 합니다.

## 복잡한 분기문을 마주하는 상황

다양한 상황 따라서 보이는 것이 다른 경우가 많습니다. 이런 경우 개발을 하며 보통은 if문, 삼항 연산자, AND 연산자 등을 사용하며 조건부로 개발을 해왔었습니다. 그러나 조금만 조건이 복잡해지니 코드에 수많은 if문과 &&등이 붙어 가독성 측면에서 아쉬움이 많이 느껴졌습니다.

코드를 읽어 나가며 나가며 흐름이 눈에 들어온다기 보다는 한 부분을 반복해서 읽어 나가야만 하는 순간이 반복되었습니다. 아래 예시는 개발을 하며 겪었던 순간들이었습니다.

QnA에 질문이 있을 경우, 상품의 판매자인지 아닌지에 따라 보여지는 컴포넌트가 다릅니다. 나아가 판매자라면 질문에 답변의 여부에 따라 보이는 부분이 또 달라질 것입니다.

- 판매자 아님
  - 그냥 QnA 보여줌
- 판매자임
  - 답변 O : 그냥 QnA 보여줌
  - 답변 X: 답변 남기는 컴포넌트 보여줌

```tsx
<AccordionContent className={S.accordionContent}>
  {!isSeller && item.answer_details && <A />}
  {isSeller && item.answer_details && <B />}
  {isSeller && !item.answer_details && <C />}
</AccordionContent>
```

이런 아주 간단한 분기문에서도 어떻게 컴포넌트 트리 구조를 만들 건지 고민이 되고, if문을 사용하여 조건을 걸어 렌더링을 할지 고민이 됩니다.

위 코드는 간단한 경우여서 흐름이 빨리 이해되지만 여러 요소가 중첩되어 분기문을 이룬다면 위 흐름을 이해하는 데 코드를 반복하여 읽어야 할 것입니다.

## 그렇다면 어떻게 처리 할 수 있을까?

1. if문 활용과 컴포넌트 분리

복잡한 로직이 많은 분기를 포함한다면, 조건별로 컴포넌트를 분리할 수 있습니다.

```tsx
const QnAContent = ({
  isSeller,
  answerDetails,
}: {
  isSeller: boolean;
  answerDetails: boolean;
}) => {
  if (!isSeller) {
    // 판매자가 아님
    return <QnA />;
  }

  if (isSeller && answerDetails) {
    // 판매자이며 답변이 있음
    return <AnsweredQnA />;
  }

  // 판매자이며 답변이 없음
  return <UnansweredQnA />;
};
```

1. switch 문 작성

조건이 명확히 구분되며, 상태가 적은 경우에는 `switch`문을 활용하여 명확하게 컴포넌트를 분리할 수 있습니다.

```tsx
const QnAContent = ({
  isSeller,
  answerDetails,
}: {
  isSeller: boolean;
  answerDetails: boolean;
}) => {
  const getContent = () => {
    switch (true) {
      case !isSeller:
        return <QnA />; // 판매자가 아님
      case isSeller && answerDetails:
        return <AnsweredQnA />; // 판매자이며 답변 있음
      case isSeller && !answerDetails:
        return <UnansweredQnA />; // 판매자이며 답변 없음
      default:
        return null;
    }
  };

  return <AccordionContent>{getContent()}</AccordionContent>;
};
```

1. 라이브러리 활용

TypeScript의 패턴 매칭을 도입하면 조건부 렌더링을 더욱 선언적으로 작성할 수 있습니다. 라이브러리로는 [TS Pattern](https://github.com/gvergnaud/ts-pattern)을 사용할 수 있습니다.

```tsx
import { match } from "ts-pattern";

const QnAContent = ({
  isSeller,
  answerDetails,
}: {
  isSeller: boolean;
  answerDetails: boolean;
}) => {
  const content = match({ isSeller, answerDetails })
    .with({ isSeller: false }, () => <QnA />) // 판매자가 아님
    .with({ isSeller: true, answerDetails: true }, () => <AnsweredQnA />) // 판매자이며 답변 있음
    .with({ isSeller: true, answerDetails: false }, () => <UnansweredQnA />) // 판매자이며 답변 없음
    .otherwise(() => null); // 기본 값

  return <AccordionContent>{content}</AccordionContent>;
};
```

## 고민해봐야할 점

복잡한 분기문을 다루는 이유의 가장 큰 목적은 더 나은 협업을 위해서일 것입니다. 따라서 프로젝트의 요구사항과 팀의 코드 스타일에 따라 달라질 수 있습니다. 아래의 요소를 고려하며 팀 내에서 합의된 방식을 찾아가는 것이 중요합니다.

### 1. **팀원과의 이해**

코드는 팀원 모두가 쉽게 이해할 수 있어야 합니다. 특정 방식이 아무리 효율적이라도, 팀원들이 이를 이해하지 못하거나 유지보수하기 어렵다면 좋은 선택이라고 보기 어렵습니다.

### 2. **확장성과 유지보수성**

분기문은 시간이 지나며 조건이 추가되거나 수정될 가능성이 높습니다. 따라서, 확장성이 높은 구조와 유지보수에 용이한 방식을 선택하는 것이 중요합니다.

### 3. **팀의 선호도와 코드 스타일**

코드 작성 방식은 팀의 스타일과 일관성이 있어야 합니다. 새로운 방법을 도입하려면 팀원들과 논의하여 **모두가 동의할 수 있는 기준**을 마련하는 것이 필요합니다.

더 나은 코드를 작성하기 위해 들인 노력이 오히려 상황을 복잡하게 만들 수 있습니다. 복잡한 분기문을 보다 가독성을 높이고 유지 보수성을 확보하기 위해서는 원활한 팀원 간의 소통이 오가고 결정이 되어야 합니다.

위에서 살펴본 다양한 방법 중 프로젝트 상황에 가장 적합한 방식을 선택해야 하며, 팀 내에서 충분한 논의를 통해 "우리 팀에 맞는 최적의 방법"을 찾는 것이 복잡한 분기문을 효과적으로 관리하는 best 방법이라고 생각합니다.

## 마무리 말

저에게 좋은 코드란 **유지보수하기 쉬운 코드**라고 생각합니다. 수정이 필요할 때 빠르게 문제를 파악하고 최소한의 수정만으로 해결할 수 있는 코드를 선호합니다.

복잡한 분기문이 코드 곳곳에 흩어져 있으면 흐름을 따라가 어렵지만 명확하고 직관적인 코드라면 수정과 이해가 훨씬 쉬워질 것입니다. 위와 같은 고민을 시작으로 하여 복잡한 분기문을 간결하게 처리하는 방법에 대해 생각해 보고 정리해 보았습니다.

위 주제에 한 가지 정답은 없겠지만, 팀과 합의하고 상황에 맞는 최선의 방법을 선택하는 것이 추상적인 “좋은 코드”에 대한 정의를 내려나가는 과정 될 것입니다.
