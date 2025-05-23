---
layout: post  
title: 가성비 있는 단위테스트
author: 김현지
banner:
  image: https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/costEffective.jpg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [단위테스트]
---

안녕하세요, 김현지입니다. 이번 주제는 가성비 있는 단위 테스트입니다.
제가 이 주제를 선택한 이유는 컨트롤러 단위 테스트를 작성하면서 비용 대비 얻는 효과가 적은데?라는 고민으로부터 시작했습니다.

## 단위테스트
단위 테스트를 왜 작성하는걸까요?
제가 가장 좋아하는 열역학 제2법칙 엔트로피 증가의 법칙 때문입니다.
엔트로피 증가의 법칙은 모든 물질과 에너지는 무질서를 향해 간다는 법칙인데요,
소프트웨어 역시 이 법칙에서 크게 벗어나지 않습니다.
따라서, 무질서를 늦추기 위한 수단이 소프트웨어에서는 단위 테스트입니다.
![1](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/entropy.jpg)
우선, 단위 테스트의 목표가 무엇일까요?
단위 테스트의 목표는 **소프트웨어 프로젝트의 지속 가능한 성장**입니다.
테스트가 없는 일반 프로젝트의  경우 빨리 시작할 수 있다는 장점이 있으나,
시간이 지날수록 소프트웨어의 한 부분을 수정하면 다른 부분들이 고장납니다.
그렇기 때문에 개발 속도가 빠르게 감소합니다.
이런 현상을 소프트웨어 엔트로피라고 합니다.
하지만 테스트 코드를 작성한다면 초반에 노력이 필요하지만,
프로젝트 후반으로 갈수록 안정망 역할을 합니다.

그렇다면 테스트 코드를 작성하는 것만으로도 충분할까요?
테스트가 잘못 작성된 프로젝트 역시 침체 단계에 빠지게 됩니다.
![2](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/sustainableGrowth.png)
좋은 테스트 코드에 대한 필요성이 분명합니다.
그렇다면 좋은 테스트 코드와 나쁜 테스트 코드 구분을 어떻게 할까요?
가장 먼저 떠오르는 것은 성능 지표입니다.

## 좋은 테스트 코드
테스트 코드 측정을 위한 성능 지표로 저희에게 가장 익숙한 것은 테스트 커버리지일 것입니다.
커버리지 지표란 테스트 스위트가 소스 코드를 얼마나 실행하는지를 백분율로 나타낸 지표입니다.
물론 코드 커버리지는 괜찮은 부정 지표지만 좋지 않은 긍정 지표입니다.
다시 말해서, 코드 커버리지가 너무 낮다는 것은 좋지 않지만,
100% 커버리지가 마냥 좋은 지표는 아니라는 뜻입니다.
![3](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/coverage.png)
StringUtil 클래스의 코드는 위줄과 아랫줄 모두 같은 로직을 담고 있습니다.
먼저, 위의 코드에서의 테스트는 5줄 중 return true 줄을 제외한 4줄을 포함하고 있으므로 80퍼센트 커버리지를 갖습니다.
그에 반해, 아래 코드에서는 하나의 return문으로 코드를 바꿨을 때, 로직 상 전혀 변경된 점이 없음에도 불구하고 테스트 커버리지는 100퍼센트로 증가합니다.
따라서, 테스트 커버리지가 테스트 스위트의 품질을 효과적으로 측정하는 데 사용될 수 없다는 것을 알게되었습니다.

## 테스트 코드 품질 측정
그렇다면 테스트 코드 품질을 어떻게 측정해야할까요?
테스트 커버리지가 아닌 다른 지표를 찾아야할까요?
아닙니다.
테스트 코드의 품질은 테스트 스위트의 특정 측면만을 반영하기 때문에 온전하지 않습니다.
테스트 코드의 품질은 대신 다양한 측면을 고려해야합니다.
단위 테스트의 목적을 다시 떠올려봅시다. 단위 테스의 목적은 지속가능성인데,
그러한 측면에서 살펴봤을 때, 불필요한 코드를 버리는 것이 가성비 있습니다.
그러면 어떤 속성이 좋은 테스트의 속성이고, 고로 보존해야할 테스트 코드일까요?
좋은 테스트의 속성은 우선 개발 주기에 통합되어야 합니다. 개발 주기에 통합은
사소한 변경이라도 쉽게 모든 테스트를 실행하고 회귀 버그를 방지할 수 있음을 뜻합니다.
회귀 버그라는 말이 어려운데 뒤에서 살펴보겠습니다.
두 번째로, 코드베이스에서 중요한 부분만을 대상으로 설정해햐 합니다.
이말의 뜻은 주요 비즈니스 로직 테스트에 집중하는 것이 중요하다는 것입니다.
마지막으로 최소 유지비로 최대 가치를 끌어내야합니다.
가치 있는 테스트만 남기기 위해선, 그전에 테스트의 가치를 측정할 수 있어야합니다.
이렇게 하기 위해선, 좋은 테스트를 작성하는 방법을 알고 또 식별할 수 있어야합니다.
좋은 코드가 무엇잍지 작성하기 위해선, 그 전에 좋은 코드가 무엇인지를 알아야합니다.
따라서, 이번 발표에선 좋은 코드를 식별하는 데에 초점을 둘 예정입니다.

다음과 같은 4가지 요소를 통해 좋은 단위 테스트를 식별할 수 있습니다.
하나씩 살펴보겠습니다.

첫번 째 요소는 좀전에 언급했던 회귀 방지입니다.
회귀란 코드를 수정한 후 기능이 의도한 대로 작동하지 않는 경우를 뜻합니다.
쉽게 말해서 회귀는 곧 소프트웨어 버그입니다.
따라서, 회귀를 방지하는 테스트 코드가 가치있는 테스트를 의미합니다.
회귀 방지 지표 테스트 점수를 올리려면 다음과 같은 요소들을 고려해야합니다.
우선, 테스트 중에 실행되는 코드의 양을 고려해야 합니다.
사실, 테스트 중에 실행되는 코드의 양이 많을수록 테스트 코드의 버그가 발생할 가능성이 높아질 수 있습니다.
회귀 방지를 위한 테스트는 주로 이전에 발견된 버그를 다시 발생시키지 않도록 하는 것을 목표로 합니다.
따라서 회귀 방지를 위한 테스트는 기존의 코드를 안정적으로 유지하고 새로운 기능 추가나 수정으로 인해 발생할 수 있는 부작용을 방지하기 위해 중요합니다.
이를 감안할 때, 테스트 중에 실행되는 코드의 양이 많을수록 회귀 방지 가능성이 높다고 단언하기는 어렵습니다.
따라서 정확한 표현은 "테스트 중에 실행되는 코드의 양이 증가할수록 버그가 발생할 가능성이 높아진다입니다.
따라서, 코드의 절대적인 양이 아닌 다른 요소들을 고려해야합니다.
효율적인 회귀 테스트는 핵심적인 부분에 초점을 맞추어 해당 부분을 충분히 테스트하는 것이 중요합니다.
핵심적인 요소를 판별하는 다른 요소가 바로 코드의 도메인 유의성입니다.
도메인 유의성이란 비즈니스에 중요한 기능에 발생한 버그의 비용이 가장 크기 때문에 이를 회귀 방지하면 가치가 높다는 것을 의미합니다.
추가적으로, 단순한 코드는 실수할 여지가 많지 않기 때문에 회귀 오류가 발생하지 않고 그렇기 때문에 가치가 낮습니다.

다음 요소는 리팩터링 내성입니다.
리팩터링이란 식별할 수 있는 동작을 수정하지 않고 기존 코드를 변경하는 것을 뜻합니다.
리팩터링 이후 기능에 버그는 없었지만 테스트가 빨간색인 경우를 false negative(거짓 양성)이라고 합니다.
다시 말해서 false positive는 기능은 동작하지만 테스트가 실패하는 케이스를 말합니다.
문제의 원인은 구현 세부 사항에 관계되어 있는 것이 주로 실패한다는 것이고
해결하기 위해선 구현 세부 사항 대신 최종 결과를 목표로 해야합니다.

구현 세부 사항이 관계된 테스트는 다음과 같습니다. 이 테스트에선 머리글, 본문, 바닥글에 대한 렌더링 결과를 확인합니다.
이렇게 되면 특정한 구현만 고집하게 되기 때문에 코드를 변경할 때마다 테스트가 빨간색으로 변하게 됩니다.
따라서 , 테스트를 깨지지 않게 하고 리팩터링 내성을 높이기 위해 SUT의 구현 세부 사항과 테스트 간의 결합도를 낮춥니다.
테스트 코드를 세부 사항이 아닌 html 출력을 반환하는지만 체크하도록 리팩토링한 최종 결과를 목표로 둔 테스트 코드가 더 좋은 코드입니다.

빠른 피드백은 단위 테스트의 필수 속성으로 테스트 속도가 빠를수록 코드에 결함이 생기자마자 버그를 수정하는 비용을 0으로 줄일 수 있습니다. 다시 말하자면, 느린 테스트는 피드백을 느리게 하고 잠재적으로 버그를 뒤늦게 눈에 띄게 해서 버그 수정 비용을 증가시킵니다.

유지 보수성은 테스트의 유지비를 평가하는 요소로, 크게 두가지가 있습니다.
첫 번째는 테스트가 얼마나 이해하기 어려운가?입니다.
테스트 코드의 가독성은 테스트 코드의 라인이 적을수록 올라갑니다.
두 번째는 테스트가 얼마나 실행하기 어려운가?입니다.
테스트가 프로세스 외부 종속성으로 작동하면, 데이터베이스 서버를 재부팅하고 네트워크 연결 문제를 해결하는 등 의존성을 상시 운영하는 데 시간을 들여야합니다.

이상적인 테스트란 어떤 것일까요?
![4](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/idealTest.png)
이상적인 테스트란 앞서 언급한 4가지 특성, 회귀 방지, 리팩터링 내성, 빠른 피드백, 유지 보수성을 모두 만족시키는 테스트일 것입니다.
하지만 이는 불가능한데요, 유지보수성을 제외한 3가지 특성이 상호 배타적이기 때문입니다.
셋 중 하나를 희생해야 두 가지를 최대화할 수 있습니다.
하지만 실제로는 리팩터링 내성을 포기할 수 없습니다.
그 이유는 리팩터링 내성은 이진 특성을 가지기 때문입니다.
리팩터링 내성을 완전히 포기할 수는 없으므로 리팩터링 내성을 가져가되,
회귀 방지, 즉 테스트가 얼마나 버그를 잘 찾아내는지, 빠른 피드백, 즉 얼마나 빠른지를 조절해야합니다.
분산 데이터 저장소에서 일관성, 가용성, 분할 내성을 모두 가질 수 없다는 CAP 정리와 비슷하다고 생각하면 됩니다.

## 가성비 좋은 단위테스트
여태 좋은 단위테스트가 무엇인지 알아보았습니다.
즉, 단위 테스트의 효과 측면을 집중하여 살펴보았습니다.
이제는 단위 테스트의 비용 대비 효과를 살펴보면서, 어떤 단위테스트를 남길지에 대한 판단을 실제로 해봅니다.
단위 테스트의 가성비란, 비용 대비 효과를 고려해야 합니다.
테스트의 효과는 코드의 복잡도, 도메인 관련성을 보고 판단할 수 있고,
테스트의 비용은 외부 의존성 갯수로 가늠할 수 있습니다.

이 그래프는 테스트 코드 가성비 그래프입니다.
y축은 복잡도 및 도메인 유의성, 다시 말해서 가치를 뜻하고, 높을 수록 좋은 단위테스트입니다.
x축은 협력자의 수, 즉, 외부 의존성, 즉 비용을 뜻하고, 낮을수록 좋습니다.
따라서 y축은 높고 x축은 낮은 도메인 모델 및 알고리즘 관련된 부분은 단위테스트를 진행하는 것이 좋습니다.
x축과 y축이 낮은 간단한 테스트는 가치가 없기 때문에 작성할 필요 없습니다.
드디어 컨트롤러 단위 테스트가 등장했는데요, 컨트롤러 테스트는 큰노력대비 효과가 적으므로, 즉 가성비가 떨어지므로 작성하지 않는것을 추천드립니다.
마지막으로, 가치도 높고, 비용도 높은 아주 복잡한 코드의 경우는 테스트를 진행해야할까요? 하지 말아야할까요?

![5](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0220/refactoring.png)
이때가 바로 리팩토링을 해야할 때입니다.
리팩토링을 통해 도메인 유의한 코드와 컨트롤러로 나눈 후 테스트를 진행하는 것을 추천드립니다.
저희가 MVC 구조로 나누어서 코드를 작성하는 것도 테스트를 하기 더 용이한 형태로 나누는 과정이라는 관점으로도 볼 수 있습니다.

과연 컨트롤러 단위 테스트를 작성해야할까?하는 의문에서부터 시작하여
가성비있는 단위 테스트가 무엇인지에 대해 알게 되었습니다. 감사합니다.
