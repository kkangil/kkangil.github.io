---
title: Next.js with pm2 ecosystem
date: 2019-04-18 00:29:19
categories:
  - Deploy
tags:
  - Next.js
  - deploy Next.js
  - pm2
  - pm2-ecosystem
---

`Next.js`는 Client-Side-Rendering 을 사용하는 react가 아닌 `SSR(Server-Side-Rendering)` 방식을 사용하는 react framework 이다.
해당 글은 `Next.js`에서 배포시 참고하면 좋을 내용으로 `Next.js`가 뿐만 아니라 react, Node 에서도 사용 가능하다.

<!-- more -->

## pm2 ecosystem
pm2 ecosystem이란, 실행할 인스턴스의 설정을 JSON 형식으로 관리할 수 있고, pm2 에서 제공해 주는 option 을 보다 쉽게 관리 할수 있도록 도와준다. pm2 명령어로 직접 실행시킬 수 있지만, ecosystem을 사용함으로써 어떤 서버나 환경에서도 같은 설정을 사용해서 서버를 실행할 수 있다는 점이 장점이다.

1. ecosystem.config.js 를 최상위 폴더에 생성한다.
2. 작성방법

```
module.exports = {
  apps: [
    {
      name: "chungeoram",
      script: "./server.js",
      watch: true,
      interpreter: '/home/ubuntu/.nvm/versions/node/v8.11.3/bin/node',
      "env_public-develop": {
        NODE_ENV: "public-develop",
        PORT: 1111,
        API_END_POINT: 'http://endpoint/api'
      },
      env_production: {
        NODE_ENV: "production"
      }
    }
  ]
};

```

- name: pm2 에서 관리하는 이름
- script: 앱을 구동할 경로
- watch: 파일이 변경되면 자동으로 재시작 유무
- ignore_watch: 배열안에 있는 리스트는 watch 대상에서 무시한다.
- exec_mode: 클러스터 모드로 구동instances: 클러스터로 구동될시 몇개까지 구동할것인지 선택(0 : cpu갯수)
- merge_logs: 클러스터로 구동할시 로그를 한파일에 기록
- interpreter: 해석기 절대 경로(default: node)
- log_date_format: 로그에 출력될 날짜와 시간값의 형식
- error_file: 에러 파일 위치
- out_file: 기본 출력 로그 위치
- env_{value}
  - value는 `process.env.NODE_ENV` 값과 매칭된다. 예를 들어 현재 `process.env.NODE_ENV`가 `public-develop` 일때, `env_public-develop` 내부의 값이 사용된다. 내부의 값들은 `process.env` 객체 내로 값이 할당되며, `process.env.PORT` , `process.env.API_END_POINT` 로 값을 배포 환경의 따라 다르게 사용가능하다.

3. pm2 구동방법
package.json script에 명령어를 추가해준다.
```
"scripts": {
    "precommit": "lint-staged",
    "build": "next build",
    "start": "pm2 start ecosystem.config.js --env production",
    "start-public": "pm2 start ecosystem.config.js --env public-develop",
    "dev": "nodemon server.js"
  }
```
- $ yarn build → $ yarn start
- app 빌드 후 명령어를 사용해서, pm2를 구동시켜 준다. pm2 start {파일이름} --env {value} 와 같은 형식으로 명령어를 추가한다. 여기서 환경변수는 ecosystem.config.js 의 env_{value}와 매칭 된다.

## Next.js 에서 process.env 변수 사용 주의사항
`Next.config.js` 에서 `sass`, `webpack` 등의 처리 후 export 하는 과정에서, process.env 객체가 비어있는 문제를 발견했다. 하지만, api.js config.js 등 환경 변수에 따라 값을 변경해줘야 하기 때문에 process.env 객체가 비어있으면 안된다. 해당 문제를 처리하기 위해 Next.js 에서 `publicRuntimeConfig` 라는 옵션을 제공해준다.

```
const withSass = require('@zeit/next-sass')
const withCSS = require('@zeit/next-css')

const publicRuntimeConfig = {
  API_END_POINT: process.env.API_END_POINT,
  NODE_ENV: process.env.NODE_ENV,
}

module.exports = withCSS(withSass({
  publicRuntimeConfig,
  webpack: config => {
    // Fixes npm packages that depend on `fs` module
    config.node = {
      fs: 'empty',
      module: 'empty'
    }

    return config
  }
}))
```

- 필요한 process.env 의 값을 `publicRuntimeConfig` 객체에 담아주고 Next.js 의 옵션 값에 할당해준다.할당해 주게되면, Next.js 의 config 값으로 저장되게 된다.이후 Next.js의 config 값을 불러오는 함수를 추가해준다.

```
import getConfig from 'next/config';

export const getNodeEnv = () => {
  const { publicRuntimeConfig } = getConfig();

  const realNodeEnv = publicRuntimeConfig.NODE_ENV || process.env.NODE_ENV;
  const apiEndPoint = publicRuntimeConfig.API_END_POINT;

  return { realNodeEnv, apiEndPoint }
};
```
- 해당 함수는 Next.js의 config에서 publicRuntimeConfig 를 가져와 return 해준다.이제 process.env 환경변수를 사용하고 싶다면, process.env.API_END_POINT 와 같이 직접적으로 사용하는것이 아닌, 해당 함수를 사용해준다.

```
import config from './config'
import { getNodeEnv } from '@/utils/env'

const env = getNodeEnv()
const endPoint = env.apiEndPoint || config.apiEndPoint

export default {
  AUTH_TOKEN: {
    method: 'POST',
    path: () => `${endPoint}/auth/authenticate-token`
  }
}
```