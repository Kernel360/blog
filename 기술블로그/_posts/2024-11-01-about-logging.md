---
layout: post
title: 로깅을 통해 서비스 관리하기
author: 윤해진
banner:
  image: https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/0223/spring-batch-tutorial.jpeg
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [서비스 운영, 로깅]
---

# 들어가는 말

프로젝트를 진행하며 기능 개발에 많은 시간을 할애합니다. 그러나 만들고 나서가 끝일까요? 지속적인 유지 보수와 함께 안정적인 서비스 제공 및 신규 유저 유입, 유저 이탈을 막기 위해 사용성과 관련된 다양한 요소를 파악하고 더 나은 방향으로 나가야할 것입니다. 프로젝트를 진행하며, 단순한 개발에서 끝이 아닌 사용자의 사용성과 관련된 요소를 가설을 세우고 검증하는 과정에 대해서 이야기해 보고자 합니다.

<br/>

# 로깅이란?

제품을 개발하며 사용자의 움직임 서비스 사용 흐름 등을 예상하고 가설을 세웁니다. 이런 가설은 기획 당시의 서비스 이용 대상, 핵심 기능, 타서비스와의 차별점 등 다양한 요소에 전반적으로 세워져있을 것입니다. 그렇다면 이런 가설을 실제로 검증하는 과정이 필수적일 것입니다. 사용자의 실제 움직임 혹은 서비스가 안정적으로 제공되는지 등을 가설을 검증하기 위해서는 다양한 데이터를 모으고 확인해야합니다. 위와 같은 데이터는 단순한 가설 확인을 넘어서 마케팅, 이후 서비스 발전 등 다양한 분야에서 유의미하게 활용 가능합니다.

이런 유의미한 데이터를 모으는 과정을 “로깅”이라고 합니다.

로깅에는 수많은 종류가 있습니다.

- 사용자 행동 로깅
- 에러 로깅
- 성능 로깅
- 서비스 로깅
- 이벤트 로깅

등등 상용되는 서비스가 이상 없이 정상적으로 동작하는지 수시로 점검하고 확인하기 위해 다양한 측면에서 데이터를 기록하고 확인하는 과정을 거칩니다. 수많은 로깅 중에서 이번에 우리가 프로젝트를 진행하면서 `사용자의 행동`을 로깅하고 `에러`를 로깅하는 측면에서 이야기를 해보려고 합니다.

<br/>

# 로깅은 왜 필요할까

애플리케이션에는 **핵심 기능**이 존재합니다. 이 핵심 기능은 서비스의 정체성이자 이용자들을 모으는 가장 큰 축일 것입니다. 핵심 기능을 설계하는 시점에 “사용자가 어떤 방식으로 어떻게 사용할 것이다.”라는 가설을 세우게 됩니다. 이런 가설을 로깅을 통해 실제로 검증하는 과정을 거칠 수 있습니다. 추가적으로 배포한 서비스가 사용자에게 어떤 이슈 없이 안정적으로 제공이 되고 있는지 실시간으로 모니터링이 가능합니다.

`유저 행동 로깅`을 통해서 우리는 사용자가 서비스를 어떻게 사용하는지 전반적인 행동 흐름 파악이 가능합니다. 사용자가 **메인 페이지**에 들어와서 어느 버튼을 **클릭**하고 어디로 가장 많이 **이동**하는지 등을 데이터로 쌓아나가 **전반적인 유저의 흐름**을 파악합니다. 이는 비즈니스에서 매우 중요한 지표로 활용됩니다. 유저의 경험을 기반으로 서비스를 개선하거나 광고, 마케팅의 효과를 측정하는 등의 다양한 비즈니스 지표로 활용할 수 있습니다. 유저가 애플리케이션에서 취하는 다양한 행동을 트래킹하고 데이터로 관리하는 것이 `유저 행동 로깅`입니다.

`에러 로깅`을 통해서는 클라이언트 단에서 예기치 못한 오류를 효과적으로 파악 가능합니다. 상용 애플리케이션에는 개발자가 예상하지 못한 오류가 발생할 수 있습니다. 서버가 아닌 클라이언트 단에서 발생한 오류를 파악하기 위해서는 오류가 발생한 브라우저의 개발자 도구의 콘솔을 보는 것이 가장 빠르고 확실한 방법입니다. 그러나 오류를 마주한 사용자에게 “에러가 발생하신 상황에서 F12를 눌러…”이렇게 요구를 하기는 쉽지 않습니다. 따라서 클라이언트 단의 오류가 발생하였을 때 디버깅과 이슈 해결과 관련된 정보를 트래킹하고 파악하는 것이 `에러 로깅`입니다.

<br/>

# 어떻게 로깅을 할 수 있을까

로깅을 구현하는 방식은 매우 다양합니다. 사용하는 기술 스택에 따라 다양한 선택지가 있습니다.

`유저 행동 로깅`의 경우에는 주로 [Google Analytics](https://developers.google.com/analytics?hl=ko), [Amplitude](https://amplitude.com/), [Mixpanel](https://mixpanel.com/contact-us/ps-sem-demo-request-apac?utm_source=google&utm_medium=cpc&utm_campaign=APAC-Korea-Brand-Search-EN-Exact-All-Devices&utm_content=Mixpanel-Exact&utm_ad=708060443734&utm_term=mixpanel&matchtype=b&campaign_id=21535059339&ad_id=708060443734&gclid=Cj0KCQiAire5BhCNARIsAM53K1hT6aQ07IvwD_PRMyaKUe4Vj2GpN4JenKHRSOzOdlHcHpDSU8yLbIsaArjHEALw_wcB&gad_source=1) 과 같은 툴을 사용합니다.

- 클릭
- 페이지 이동
- 특정 이벤트 트래킹

등을 손쉽게 설정하고 분석할 수 있습니다. 이를 통해 우리는 사용자가 어떤 페이지에 가장 많이 방문하였는지 어디서 이탈을 하는지 등의 정보를 얻고 추후 개선 방향을 찾아나갈 수 있습니다.

최근에는 단순히 로그 데이터로만 분석하는 것을 넘어, 사용자 화면을 그대로 녹화하여 기록하는 방식으로도 유저 행동을 트래킹합니다. [FullStory](https://www.fullstory.com/), [LogRocket](https://logrocket.com/) 같은 도구는 사용자의 행동을 실시간으로 녹화해, 사용자가 어떤 순서로 페이지를 탐색하고 어떤 요소와 상호작용하는지를 영상 형태로 확인할 수 있게 해 줍니다.

`에러 로깅`은 애플리케이션의 오류를 실시간으로 기록하고, 발생한 문제를 빠르게 해결하는 데 매우 중요한 역할을 합니다. 서버의 오류 외에도 클라이언트 측의 에러의 경우에는 주로 [Sentry](https://sentry.io/welcome/?utm_source=google&utm_medium=cpc&utm_id=%7B21427619193%7D&utm_campaign=Google_Search_Brand_SentryKW_APAC_Alpha&utm_content=g&utm_term=sentry&gad_source=1&gclid=Cj0KCQiAire5BhCNARIsAM53K1jJfl1IuQsTzjrJNtqSZ8SzGG3VjCCdcWbSN-uYhkSt8SG811xJ2icaAvr0EALw_wcB) , [Rollbar](https://rollbar.com/) , [Bugsnag](https://www.bugsnag.com/?utm_source=aw&utm_medium=ppcg&utm_campaign=SEM_Bugsnag_PR_APAC_ENG_EXT_Prospecting&utm_term=bugsnag&utm_content=700967338246&campaignid=21334626909&adgroupid=166052154874&adid=700967338246&gad_source=1&gclid=Cj0KCQiAire5BhCNARIsAM53K1jv_w0tXGdO2xuhf4SHJO_lPuToSWEOJ5Fs3GYTa_O6s3j4bMBnV20aAnlvEALw_wcB&gclsrc=aw.ds) 와 같은 툴을 사용합니다.

위와 같은 서비스는 클라이언트에서 발생하는 오류 정보를 자동으로 수집하고, 발생 당시의 사용자의 디바이스 정보, 브라우저 정보 등 오류 상황과 오류를 디버깅하는데 도움이 되는 자료들을 수집합니다.

특히, Sentry와 같은 도구는 에러가 발생할 때마다 이메일로 알림을 보내주는 기능을 제공하여 개발자가 문제를 빠르게 인지하고 대응할 수 있게 합니다.

<br/>

# 마무리 하는 말

로깅은 서비스 기획 당시의 가설을 검증할 수 있게 할뿐더러 유지 보수 및 안정적인 서비스 제공 측면에서도 매우 중요한 요소입니다. 기능 개발에서 끝나는 것이 아닌 서비스를 운영하고 유지보수하는 경험도 매우 중요하다고 생각합니다. 프로젝트를 진행하며 글에서 설명한 다양한 로깅을 심어봄으로 기획 당시 세운 다양한 가설을 검증하고 더 나은 서비스 제공을 위해 고민해 보는 과정을 모두 경험해 보면 좋을 것 같습니다.

<br/>
