---
layout: post
title:  "Webpack dev server 기동 시 config-yargs오류 발생"
categories:
- 트러블슈팅
tags:
- webpack
---

### 1. 개요
웹팩 데브 서버를 기동하는데 하단 에러가 계속 발생했다.

```bash
$ webpack-dev-server

> example@1.0.0 start
> webpack-dev-server

internal/modules/cjs/loader.js:905
  throw err;
  ^

Error: Cannot find module 'webpack-cli/bin/config-yargs'
Require stack:
- D:\HK\git\TIL\til.frontend\lecture-frontend-dev-env\example\node_modules\webpack-dev-server\bin\webpack-dev-se
rver.js
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:902:15)
    at Function.Module._load (internal/modules/cjs/loader.js:746:27)
    at Module.require (internal/modules/cjs/loader.js:974:19)
    at require (internal/modules/cjs/helpers.js:92:18)
    at Object.<anonymous> (D:\HK\git\TIL\til.frontend\lecture-frontend-dev-env\example\node_modules\webpack-dev-
server\bin\webpack-dev-server.js:65:1)
    at Module._compile (internal/modules/cjs/loader.js:1085:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1114:10)
    at Module.load (internal/modules/cjs/loader.js:950:32)
    at Function.Module._load (internal/modules/cjs/loader.js:790:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:76:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    'D:\\HK\\git\\TIL\\til.frontend\\lecture-frontend-dev-env\\example\\node_modules\\webpack-dev-server\\bin\\w
ebpack-dev-server.js'
  ]
}
```

<br/>

### 2. 해결
찾아봐도 webpack, webpack-cli, webpack-dev-server를 최신화하라는 말뿐이어서 고생했는데,
알고보니 webpack dev server가 4에서 5로 버전이 올라가면서 기동방법이 변경되어 발생한 이슈였다.

webpack5부터는 기존 처럼 <var>webpack-dev-server</var>가 아닌, <var>npx webpack serve</var>으로 실행해야
정상 동작한다.
```bash
$ npx webpack serve
```
