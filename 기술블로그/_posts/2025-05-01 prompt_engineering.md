---
layout: post  
title: "prompt engineering"
author: "김현호"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["기술블로그"]
---

# AI 기반 제어 가능한 데이터셋 생성을 위한 프롬프트 작성 방법

## 들어가며
바야흐로 바이브코딩의 시대가 도래했습니다. 프로그래밍 실력과 더불어 AI를 잘 활용하는 방법 그리고 AI를 잘 활용하기 위해서 **프롬프트**를 잘 작성하는 것이 중요해졌죠. 이 글에서는 좋은 프롬프트를 작성하는 방식에 대해 알아보도록 하겠습니다.

좋은 프롬프트를 작성하기 위해 가장 먼저 고려해야 할 것은 바로 **명확성**입니다. 자연어는 사람에게는 편리하지만, AI에게는 자칫 **모호함과 혼동**을 일으킬 수 있습니다. 아래 예시를 보죠.

[문장 예시]
마트에 가서 사과 6개 사오고 만약 수박 있으면 3개 사와

이 문장은 사람에겐 당연하게 들릴 수 있지만, AI나 프로그래밍 언어의 관점에서는 해석의 여지가 생깁니다. 수박이 있으면 ‘수박을 3개 사는 건지’, ‘사과를 3개만 사는 건지’ 불명확하죠. 실제로 사람도 수박이 있어서 사과를 3개만 사 오는 경우가 생길 수 있습니다.
반면 프로그래밍 언어에서는 다음과 같이 로직을 분기하여 조건을 명확히 할 수 있습니다.

```markdown
def make_husband_buy_fruits():
    go_to_market()
    is_watermelon = find_watermelon()
    if is_watermelon:
        buy_fruits(
            fruit_type="apple",
            quantity=3
        )
    else:
        buy_fruits(
            fruit_type="apple",
            quantity=6
        )

    return 0
```

이처럼 **AI와의 대화도 프로그래밍처럼 구조화**되어야 더 효과적입니다. 조건, 명령, 예외 처리를 명확히 나누면 AI는 더 정확하고 일관된 결과를 만들어냅니다.
즉, 좋은 프롬프트는 단순히 "자연스럽게 말하는 것"이 아니라, **컴퓨터가 오해 없이 처리할 수 있도록 설계된 명확한 구조와 논리**가 담겨 있어야 합니다.
이제 본격적으로, 프롬프트를 어떻게 작성하면 더 똑똑하게 AI를 쓸 수 있을지 알아볼까요?

## 페르소나형 Single-turn dialogue 생성 프롬프트 작성 방법
AI에 기반하여 데이터셋을 생성하는 상황에 있어 양질의 데이터셋을 생성하기 위한 방법으로 AI에게 페르소나를 주입하는 방법이 있습니다.
좀 더 사실적인 데이터를 생성하기 위해 몇가지 역할을 설정합니다.


```markdown
role_name = {
    "newlyweds": "신혼부부",
    "dinks": "딩크족",
    "college_student": "대학생",
    "solo_worker": "사회초년생",
    "investor": "투자자",
    "retiree": "은퇴자",
    "young_family": "젊은 부부",
    "high_earner": "전문직",
    "corporate_buyer": "법인 구매자",
    "foreigner": "외국인",
    "relocator": "이주민",
    "single_mom_or_dad": "싱글맘",
    "digital_nomad": "디지털 노마드",
    "military_family": "군인가족",
    "job_seeker": "취준생",
    "startup_founder": "스타트업 창업자",
    "creative_worker": "프리랜서",
    "weekend_commuter_dad": "기러기 아빠"
}
```
```markdown
tone = {
    "newlyweds": "첫 집을 신중하게 알아보는 듯한 말투",
    "dinks": "프라이버시 좋고 세련된 구조에 관심",
    "college_student": "월세 예산 안에서 가성비 좋은 곳을 찾는 느낌",
    "solo_worker": "출퇴근 거리, 주변 편의시설에 민감",
    "investor": "수익률, 입지, 시세 상승 여력이 중요한 요소",
    "retiree": "안정적이고 조용한 주거 환경을 찾는 느낌",
    "young_family": "학군, 공원, 육아 환경 고려",
    "high_earner": "위치와 브랜드를 중요시하는 고급 수요",
    "corporate_buyer": "수익 구조와 절세 전략 중심",
    "foreigner": "계약 조건과 생활환경에 대한 세부 정보 요청",
    "relocator": "입지 적응과 초기 생활 안정성 고려",
    "single_mom_or_dad": "안전한 동네와 실용적인 구조 중시",
    "digital_nomad": "가볍게 머물 수 있고 교통 좋은 매물",
    "military_family": "이사 편의성과 전세 기간이 유리한 조건 선호",
    "job_seeker": "가격 민감하고 교통 좋은 원룸 선호",
    "startup_founder": "사무/주거 겸용이 가능한 구조, 유연한 조건 선호",
    "creative_worker": "조용하고 창의적인 작업이 가능한 공간 선호",
    "weekend_commuter_dad": "교통 접근성, 특히 기차역이나 터미널 접근성을 중시하는 말투"
}
```
```markdown
amenity_options = ['엘리베이터', '테라스', '무인택배함', '화재경보기', '베란다', '마당']
amenity_importance = {
    "newlyweds": 0.4,           # 신혼부부는 편의시설 관심
    "dinks": 0.6,               # 맞벌이 부부는 편의시설 중시
    "college_student": 0.2,     # 비용 대비 기본적인 것만
    "solo_worker": 0.4,         # 실용적인 편의시설 선호
    "investor": 0.3,            # 임대가치 기준
    "retiree": 0.5,             # 편안한 생활 중시
    "young_family": 0.6,        # 가족 편의 매우 중시
    "high_earner": 0.6,         # 고급 편의시설 선호
    "corporate_buyer": 0.3,     # 실용성 위주
    "foreigner": 0.5,           # 편의시설 중시
    "relocator": 0.4,           # 새로운 환경 적응 고려
    "single_mom_or_dad": 0.5,   # 자녀 편의 중시
    "digital_nomad": 0.3,       # 기본적인 편의시설 중시
    "military_family": 0.4,     # 기본 편의사항 고려
    "job_seeker": 0.2,          # 비용 대비 기본적인 것만
    "startup_founder": 0.5,     # 업무환경 고려
    "creative_worker": 0.4,     # 작업 환경 고려
    "weekend_commuter_dad": 0.4 # 기본적인 편의사항 중시
}
```

위의 설정 외에도 여러가지 설정을 추가해줍니다. 여러 설정값들을 입력받아 문자열 포맷팅 기능을 이용하면 아래와 같이 동적인 프롬프트 작성이 가능합니다.
```markdown
prompt = f"""
    [task]
    single turn conversation generation

    [situation]
    공인중개사에게 부동산 문의를 하는 고객의 질문

    [rule]
    1. 문장은 한국어로 작성하며, 문장 길이는 100자에서 200자 사이로 제한할 것
    2. [options]에 해당하는 요소들을 전부 포함하여 문장을 생성할 것
    3. 사용자 역할에 맞는 말투와 표현 방식을 사용할 것
    4. 고객의 요청은 실제로 문의하는 듯한 자연스러운 표현으로 구성할 것
    5. 반복되는 표현은 피하고, 다양한 어휘를 사용할 것
    6. 명시적으로 언급되지 않더라도 직업 기반으로 일반적으로 요구될 수 있는 요소는 자연스럽게 포함 가능

    [options]
    role : {role_name}

    tone : {tone}

    property_type : {property_type}

    transaction_type : {transaction_type}

    budget : {budget}

    amenity : {amenity_tags}

    room_option : {room_option_tags}

    security : {security_tags}

    [example1]
    안녕하세요. 결혼을 앞두고 함께 살 첫 신혼집을 찾고 있는 예비부부입니다.
    신축 아파트 위주로 보고 있고, 앞으로 자녀도 계획하고 있어서 최소 2.5룸 이상의 공간이면 좋겠습니다.
    조용하고 깨끗한 동네였으면 좋겠고, 단지 내에 CCTV가 설치되어 있거나 상주 경비원이 있는 등 보안 시설이 잘 갖춰진 곳을 우선적으로 고려하고 있어요.
    예산은 전세 기준으로 3억 2천만원에서 4억 1천만원 정도 생각하고 있고, 교통이나 생활 인프라도 어느 정도 갖춰져 있으면 더할 나위 없이 좋을 것 같아요.

    [example2]
    안녕하세요. 현재 취업 준비 중이라 교통 편리한 지역의 저렴한 월세 빌라 원룸을 찾고 있습니다.
    보증금은 550만원에서 770만원, 월세는 36만원에서 43만원 정도로 생각 중입니다.
    싱크대, 침대, 책상, 냉장고, 전자레인지 등 기본 옵션은 꼭 있었으면 해요.
    현관보안, CCTV, 카드키 같은 보안도 중요하고, 엘리베이터나 무인택배함, 베란다, 마당 같은 생활 편의 시설도 잘 갖춰진 곳이면 좋겠습니다.
"""
```
```markdown
[task]
single turn conversation generation

[situation]
공인중개사에게 부동산 문의를 하는 고객의 질문

[rule]
1. 문장은 한국어로 작성하며, 문장 길이는 100자에서 200자 사이로 제한할 것
2. [options]에 해당하는 요소들을 전부 포함하여 문장을 생성할 것
3. 사용자 역할에 맞는 말투와 표현 방식을 사용할 것
4. 고객의 요청은 실제로 문의하는 듯한 자연스러운 표현으로 구성할 것      
5. 반복되는 표현은 피하고, 다양한 어휘를 사용할 것
6. 명시적으로 언급되지 않더라도 직업 기반으로 일반적으로 요구될 수 있는 요소는 자연스럽게 포함 가능

[options]
role : 사회초년생

tone : 출퇴근 거리, 주변 편의시설에 민감

property_type : 아파트

transaction_type : 전세

budget : 전세가 40800만원 이상 58800만원 이하

amenity : ['엘리베이터', '무인택배함', '화재경보기', '마당']

room_option : ['싱크대', '샤워부스', '욕조', '비데', '냉장고', 'TV', '책상', '인덕션레인지', '침대', '소파', '식탁', '식기세척기', '가스오 
븐', '가스레인지', '신발장']

security : ['현관보안', '방범창', '카드키', '인터폰', '사설경비']

[example1]
안녕하세요. 결혼을 앞두고 함께 살 첫 신혼집을 찾고 있는 예비부부입니다.
신축 아파트 위주로 보고 있고, 앞으로 자녀도 계획하고 있어서 최소 2.5룸 이상의 공간이면 좋겠습니다.
조용하고 깨끗한 동네였으면 좋겠고, 단지 내에 CCTV가 설치되어 있거나 상주 경비원이 있는 등 보안 시설이 잘 갖춰진 곳을 우선적으로 고려하고 있
어요.
예산은 전세 기준으로 3억 2천만원에서 4억 1천만원 정도 생각하고 있고, 교통이나 생활 인프라도 어느 정도 갖춰져 있으면 더할 나위 없이 좋을 것 
같아요.

[example2]
안녕하세요. 현재 취업 준비 중이라 교통 편리한 지역의 저렴한 월세 빌라 원룸을 찾고 있습니다.
보증금은 550만원에서 770만원, 월세는 36만원에서 43만원 정도로 생각 중입니다.
싱크대, 침대, 책상, 냉장고, 전자레인지 등 기본 옵션은 꼭 있었으면 해요.
현관보안, CCTV, 카드키 같은 보안도 중요하고, 엘리베이터나 무인택배함, 베란다, 마당 같은 생활 편의 시설도 잘 갖춰진 곳이면 좋겠습니다.
```
```markdown
[GPT 출력 결과]
안녕하세요. 직장 다니기 시작한 지 얼마 안 된 사회초년생입니다.
출퇴근 시간이 너무 길면 힘들 것 같아서 지하철역이 가까운 곳이면 좋겠고,
주변에 편의점이나 마트 같은 생활 인프라도 잘 갖춰져 있었으면 해요.
엘리베이터, 무인택배함 같은 기본 시설은 물론이고, 화재경보기나 마당 같은 요소도 있으면 좋겠어요.
전세 예산은 4억 800만원에서 5억 8800만원 사이로 생각 중이고, 아파트 위주로 보고 있습니다.
싱크대, 인덕션, 침대, 책상 같은 옵션도 중요하고, 현관보안이나 카드키, 사설경비 같은 보안시설도 꼼꼼히 챙기고 싶어요.

이 조건에 맞는 전세 아파트 있을까요?
```

## 마치며
지금까지 AI 시대에 효과적인 프롬프트 작성 방법에 대해 알아보았습니다. 특히 페르소나 기반의 데이터셋 생성을 위한 구조화된 프롬프트 작성 방법을 살펴보았는데요, 이를 통해 우리는 다음과 같은 중요한 점들을 배웠습니다:

1. **명확성의 중요성**: AI와의 소통에서는 자연어의 모호함을 피하고, 프로그래밍적 사고방식으로 접근하는 것이 효과적합니다.
2. **구조화된 설계**: 역할(role), 톤(tone), 규칙(rule) 등을 명확히 정의하여 일관된 결과물을 얻을 수 있습니다.
3. **세부 설정의 활용**: 다양한 변수와 옵션을 통해 더욱 풍부하고 현실적인 데이터를 생성할 수 있습니다.

앞으로 AI 기술은 더욱 발전할 것이며, 이에 따라 프롬프트 엔지니어링의 중요성도 커질 것입니다. 이 글에서 소개한 방법들을 기반으로 여러분만의 효과적인 프롬프트 작성 방법을 발전시켜 나가시기 바랍니다.
좋은 프롬프트는 단순히 AI에게 명령하는 것이 아니라, AI와 효과적으로 협업하는 방법을 설계하는 것임을 기억해주세요. 이를 통해 우리는 AI의 잠재력을 최대한 끌어내어 더 나은 결과물을 만들어낼 수 있을 것입니다.
