---
layout: post
title: Headless Component
author: 윤예진
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [JavaScript, react, component, headless-component]
---

# Headless Component

React 개발을 시작할 때부터 유지보수의 중요성에 대해 계속 들어왔습니다. 특히 코드를 컴포넌트로 분리하여 재사용성을 높이는 것이 중요하다고 강조되었습니다.

이를 염두에 두고 자주 사용되는 요소들을 독립적인 컴포넌트로 나누며 프로젝트를 진행했습니다. 하지만 사용자의 요구사항은 계속 변했고, 같은 핵심 기능에 다른 외형이나 추가 기능이 필요한 경우가 많아졌습니다.

결과적으로 컴포넌트가 점점 커져 코드 가독성이 떨어지고 재사용성도 낮아지는 딜레마에 빠졌습니다.

이런 고민을 해결할 방법을 찾던 중에 Headless Component라는 개념을 알게 되었습니다. 이는 UI와 로직을 완전히 분리하는 접근 방식입니다. 컴포넌트의 기능과 동작은 정의하되, 시각적 표현은 개발자가 자유롭게 결정할 수 있게 해 유연성을 극대화하는 방식입니다.

Headless Component의 핵심은 '어떻게 보일까?'가 아니라 '어떻게 작동할까?'에 초점을 맞춥니다. 컴포넌트의 **기능적 로직**은 완전히 추상화된 형태로 구현하되, **UI에 대한 제어**는 사용자가 원하는 대로 설정할 수 있도록 합니다.

UI와 로직이 분리됨에 따라 다음과 같은 이점이 생깁니다.

1. **유연성**: UI가 로직과 분리되어 있어 필요에 따라 UI를 자유롭게 변경할 수 있습니다.
2. **재사용성**: 로직만을 가진 Headless Component를 여러 곳에서 사용하고, 각기 다른 UI를 조합하여 다양한 요구 사항에 대응할 수 있습니다.
3. **확장성**: 새로운 기능이 추가되거나 요구사항이 변경되어도 로직과 UI를 독립적으로 수정할 수 있어 코드의 변

이러한 구조 덕분에 컴포넌트의 크기가 커지거나 가독성이 떨어지는 문제를 방지할 수 있으며, 유지보수 역시 용이해집니다.

Dropdown 컴포넌트의 예시를 보면 Headless Component가 무엇인지 더욱 잘 이해할 수 있을 것입니다.

![image](https://github.com/user-attachments/assets/ddd8e866-5219-4480-8589-673b475dcd4c)

Dropdown 컴포넌트를 클릭(trigger)하면 DropdownMenu가 나와야 합니다. 이런 동작은 Dropdown의 ‘로직’에 해당한다고 할 수 있습니다. `useSelect`라는 커스텀 훅을 만들어 Dropdown 로직을 구현할 수 있습니다.

```jsx
function useSelect(items) {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedItem, setSelectedItem] = useState(null);

  const toggleOpen = () => setIsOpen(!isOpen);
  const selectItem = (item) => {
    setSelectedItem(item);
    setIsOpen(false);
  };

  return {
    isOpen,
    selectedItem,
    toggleOpen,
    selectItem,
  };
}
```

위의 `useSelect` 훅은 드롭다운의 상태 관리를 담당합니다. 이 훅은 열림/닫힘 상태, 선택된 항목, 항목 선택 및 열림 토글 기능을 제공합니다. 하지만 이 훅은 UI를 렌더링하지 않습니다. 따라서 UI는 별도의 컴포넌트에서 정의됩니다.

```jsx
function Dropdown() {
  const options = ["Option 1", "Option 2", "Option 3"];
  const { isOpen, selectedItem, toggleOpen, selectItem } = useSelect(options);

  return (
    <div>
      <button onClick={toggleOpen}>{selectedItem || "Select an option"}</button>
      {isOpen && (
        <ul>
          {options.map((option) => (
            <li key={option} onClick={() => selectItem(option)}>
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

Dropdown 컴포넌트에서 `useSelect` 훅을 사용하여 드롭다운을 렌더링합니다. 여기서 중요한 점은 UI의 구현 방식은 자유롭지만, 선택 로직은 훅으로 분리되어 재사용할 수 있다는 것입니다.

### Headless Component의 주요 특징

1. 로직과 UI의 완전한 분리: Headless Component는 순수한 기능적 로직만을 포함하고 UI 렌더링과 관련된 요소를 제외합니다. 이로 인해 동일한 로직을 다양한 UI에 적용할 수 있는 유연성이 크게 향상됩니다.
2. 높은 재사용성: UI와 분리된 로직은 다양한 프로젝트나 UI 디자인에 쉽게 적용될 수 있습니다. 이는 코드의 재사용성을 극대화하고 장기적으로 프로젝트의 유지보수를 용이하게 만듭니다.
3. 커스터마이징의 자유: 개발자는 각 프로젝트의 요구사항에 맞춰 UI를 자유롭게 설계할 수 있습니다. 이는 디자인 시스템의 변경이나 새로운 플랫폼 도입 시 특히 유용합니다.
4. 테스트 용이성 향상: UI와 분리된 로직은 독립적으로 테스트하기가 훨씬 쉬워집니다. 이는 코드의 신뢰성을 높이고 버그 발생 가능성을 줄이는 데 도움이 됩니다.

### Headless Component의 한계점

Headless Component가 가진 많은 장점에도 불구하고 몇 가지 주의해야 할 점이 있습니다

1. 초기 설계의 복잡성: 단순한 애플리케이션의 경우 Headless Component 방식이 불필요한 복잡성을 초래할 수 있습니다.
2. 학습 곡선: 이 개념에 익숙하지 않은 개발자들에게는 초기 이해와 적용이 어려울 수 있습니다.
3. 과도한 사용 위험: 모든 컴포넌트를 Headless로 만들려는 시도는 코드베이스를 불필요하게 복잡하게 만들 수 있습니다.
4. 성능 고려사항: 공통 로직을 사용하여 여러 컴포넌트를 렌더링할 때 성능 최적화에 주의를 기울여야 합니다.

Headless Component는 좋은 도구이지만 프로젝트의 규모와 요구사항을 고려하여 적절히 사용해야 합니다. 복잡한 UI 로직을 다루거나 높은 수준의 커스터마이징이 필요한 경우에 특히 유용하며, 이를 통해 코드의 재사용성과 유지보수성을 크게 향상시킬 수 있습니다.

---

참고 자료

https://soobing.github.io/react/decoupling-ui-and-logic-in-react-a-clean-code-approach-with-headless-components/

https://martinfowler.com/articles/headless-component.html

https://www.howdy-mj.me/design/headless-components
