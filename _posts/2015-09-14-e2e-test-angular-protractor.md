---
layout: post
title: "e2e Testing AngularJS <small>with</small> Protractor + Jasmine"
description: ""
category: [test, angularjs]
tags: [test, angularjs, protractor, jasmine, e2e]
---
{% include JB/setup %}

앞의 [Testing AngularJS + Require.js with Karma + Jasmine](/test/angularjs/2015/09/03/testing-angularjs--requirejs-smallwithsmall-karma--jasmine/) 에 이어서 e2e 테스트 하는 방법을 기록한다.

AngularJS 를 위한 Protractor를 사용한다.

##Protractor & Wabdriver Install

```sh
npm install -g protractor
```

위 명령어는 툴을 두개 설치하는데 하나는 물론 **Protractor** 이고, 추가로 **webdriver-manager**를 설치한다.

###webdriver-manager
webdriver-manager는 브라우저 자동화를 지원하는 [Selenium](http://www.seleniumhq.org/) 서버를 쉽게 실행시켜주는 매니저다. 다음과 같이 필요한 binary를 다운받아야 한다.

```sh
webdriver-manager update
```

서버를 시작해본다.

```sh
webdriver-manager start
```

엔터를 치면 서버가 종료되므로 가만히 두고, http://localhost:4444/wd/hub 로 접속하면 현재 가동되고 있는 Selenium 서버의 상태를 알 수 있다.


##Test Code 작성
[Testing AngularJS + Require.js with Karma + Jasmine](/test/angularjs/2015/09/03/testing-angularjs--requirejs-smallwithsmall-karma--jasmine/) 를 수행했다면 angularJS를 사용한 간단한 어플리케이션이 존재할 것이다.

프로젝트 루트 디렉토리에 e2e 테스트 파일을 저장할 디렉토리를 생성하고, `app.spec.js` 파일을 작성한다.

```js



```
