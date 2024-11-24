---
layout: post
title: "yarn berry pnp일 때 nextjs docker image 만들기"
author: "김승태"
categories: "프론트엔드 기술블로그"
banner:
  image: assets/images/post/2023-11-05.webp
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["yarn berry", "nextjs", "docker"]
---

docker image를 만드는 과정에서 겪은 트러블 슈팅 과정을 기록했습니다.

## nodejs self-hosting 방식으로 docker image 만들기

[공식 문서](https://nextjs.org/docs/app/building-your-application/deploying#docker-image)를 참고한 방법이며 가장 간단한 방법입니다.

```
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

의존성을 설치하고 `build`를 한 뒤에 `start`를 하면 됩니다.

해당 방식을 적용한 docker file은 다음과 같습니다.

```docker
# 가져올 이미지를 정의
FROM node:20.9.0-alpine
# 작업 디렉토리 설정
WORKDIR /app

# Yarn Berry 활성화 및 설치
RUN corepack enable && corepack prepare yarn@stable --activate
# 종속성 파일 복사
COPY ./package.json ./yarn.lock ./
COPY /techpick/. ./techpick
COPY /techpick-shared/. ./techpick-shared
COPY /schema ./schema

# 루트 폴더 종속성 설치
RUN yarn install
# 다른 패키지의 종속성 설치
RUN yarn all install

# techpick build전에 WORKDIR 변경
WORKDIR /app/techpick

# Build
RUN yarn run build

EXPOSE 3000

# 컨테이너 실행 시 실행될 명령 설정
CMD ["yarn", "start"]
```

다만 해당 방식은 많은 용량을 차지했기에 더 줄여볼수 있는 방법을 찾아보았습니다.
![docker-image size](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1113/docker00.png)

## nextjs 공식문서의 docker file을 이용해서 이미지 만들기.

nextjs에서 docker image를 만들어야할 때 [공식 문서](https://nextjs.org/docs/app/building-your-application/deploying#docker-image)에서는 예제의 docker file을 프로젝트의 루트에 두고 아래와 같은 설정을 하라고 추천하고 있습니다.

```ts
// next.config.js
module.exports = {
  // ... rest of the configuration.
  output: "standalone",
};
```

Next.js의 Standalone 옵션은 웹 애플리케이션을 실행하는 데 필요한 최소한의 코드만을 추출하는 기능입니다.

- 독립적인 빌드: Standalone으로 빌드된 결과물은 독립적으로 실행될 수 있습니다.

- 최소한의 코드: 배포 환경에서 불필요한 코드를 제외시켜 빌드 결과물의 크기를 줄입니다.

- 간편한 배포: .next/standalone 폴더에 생성된 결과물만으로 애플리케이션을 실행할 수 있습니다.

저는 이 docker file을 nextjs 프로젝트의 폴더에 두고 docker image를 만드려고 했으나 실패했습니다. [docker file](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)을 확인해보니 이유는 다음과 같았습니다.

### 1. standalone 설정을 한 뒤에 빌드한 파일이 제대로 동작하지 않는다.

원래라면 standalone 설정을 한 뒤에 빌드를 하면 standalone 폴더 내에서 독립적으로 결과물을 실행할 수 있어야하는데 그렇지 않았습니다. 실행하면 다음과 같은 에러를 마주했습니다.

```
Error: Cannot find module 'next'
```

폴더 내부를 확인해보니 `next` 뿐만 아니라 다른 의존성이 제대로 설치가 되어있지 않음을 확인할 수 있었습니다. 검색을 해보니 저 뿐만 아니라 다른 이들도 같은 문제를 겪는 것을 확인할 수 있었습니다. ([찾아본 링크](https://stackoverflow.com/questions/78681836/error-cannot-find-module-next-in-docker-next-14-node-20-react-18))

저는 이게 global cache를 적용한 yarn berry의 문제인지 확인하기 위해서 `.yarnrc.yml`의 파일을 수정한 뒤에 다시 빌드해보았습니다.

```yml
#before
nodeLinker: pnp
yarnPath: .yarn/releases/yarn-4.3.1.cjs
```

```yml
#after
nodeLinker: node-modules
yarnPath: .yarn/releases/yarn-4.3.1.cjs
```

이렇게 하니 standalone 폴더를 확인해보니 의존성이 올바르게 설치됨을 확인할 수 있었습니다.

그러나 이미지가 깨졌었는데요 이는 standalone 속성을 적용했을 때, `/static`, `/public` 폴더를 가져오지 않기 때문이었습니다. 공식문서를 읽어보니 CDN을 이용하는 것이 이상적이기에 수동으로 넣어야한다라고 명시되어 있었고 ([정보 출처](https://nextjs.org/docs/app/api-reference/next-config-js/output#automatically-copying-traced-files)) 직접 추가해보니 이미지가 제대로 보여짐을 확인할 수 있었습니다.

### 2. nextjs 프로젝트 패키지에 docker file을 두었으나 패키지가 다른 패키지를 의존한다. (docker file에서 가져올 수 없다.)

```
프로젝트 루트
└── 프론트엔드 모노레포
    ├── 웹서비스(nextjs 프로젝트, 공통 테마, 타입 스키마에 의존한다.)
    └── 익스텐션
    └── 공통 테마
    └── 공통 타입 스키마
```

이처럼 웹 서비스에서는 공통 테마와 공통 타입에 의존하고 있었으므로 프론트엔드 모노레포의 루트에서 docker file을 만들어줘야 했습니다.

따라서 이를 적용한 docker file은 다음과 같습니다.

```docker
# 기본 이미지 설정 및 초기 설정
FROM node:20.9.0-alpine AS base
WORKDIR /app

# Yarn Berry 설치
RUN corepack enable && corepack prepare yarn@stable --activate
# 종속성 파일 복사
COPY ./package.json ./yarn.lock ./
COPY /techpick/. ./techpick
COPY /techpick-shared/. ./techpick-shared
COPY /schema/. ./schema

# .yarnrc.yml 파일 및 .yarn 디렉토리 복사
COPY ./.yarnrc.yml ./
COPY ./.yarn/ ./.yarn/

# .yarnrc.yml 파일에서 nodeLinker 값을 pnp에서 node-modules로 변경
RUN sed -i 's/nodeLinker: pnp/nodeLinker: node-modules/' .yarnrc.yml

# 루트 폴더 종속성 설치
RUN yarn install
# 각 패키지 폴더 종속성 설치
RUN yarn all install

################################
# 빌드 단계: Next.js 애플리케이션 빌드
FROM base AS builder
# teckpick build전에 WORKDIR 변경
WORKDIR /app/techpick

# Build
RUN yarn run build

################################
# 실행 단계: 최종 이미지 생성 및 실행 환경 설정
FROM node:20.9.0-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/techpick/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/techpick/.next/standalone/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/techpick/.next/standalone/techpick/. .
COPY --from=builder --chown=nextjs:nodejs /app/techpick/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"
CMD ["node", "server.js"]
```

이를 통해 용량을 줄일 수 있었습니다.
![docker-image size](https://raw.githubusercontent.com/Kernel360/blog-image/main/2024/1113/docker01.png)

## 출처

[공식 문서](https://nextjs.org/docs/app/building-your-application/deploying#docker-image)

[공식 예제](https://github.com/vercel/next.js/tree/canary/examples/with-docker)

[랠릿 standalone 적용기](https://tech.inflab.com/20230918-rallit-standalone/)
