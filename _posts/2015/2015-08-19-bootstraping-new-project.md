---
layout: post
title: "Step By Step MEAN Stack Dev Environment"
description: ""
category: AngularJs
tags: yeoman yo AngularJs
---

## Directory
```sh
#create base directory
mkdir dev-mean
cd dev-mean
mkdir server
mkdir client
mkdir e2e
```

## Install
필요한 프로그램, 라이브러리 모듈 전부 설치

```sh
#install global modules
brew install node
brew install mongodb
npm install -g grunt-cli
npm install -g nodemon
npm install -g protractor
npm install -g karma

#create package.json
npm init

#bower and frontend libraries
npm install --save-dev -g bower 
bower init
bower install angular bootstrap jquery --save
bower install angular-mocks --save-dev
bower install require --save
bower install angular-ui-router --save

#grunt and plugins
npm install --save-dev load-grunt-tasks
npm install --save-dev grunt
npm install --save-dev -g grunt-cli
npm install --save-dev grunt-contrib-jshint
npm install --save-dev grunt-concurrent
npm install --save-dev grunt-contrib-watch
npm install --save-dev grunt-node-inspector
npm install --save-dev grunt-nodemon
npm install --save-dev grunt-open
npm install --save-dev grunt-bower-install
npm install --save-dev grunt-wiredep
npm install --save-dev grunt-express-server


#open
npm install --save-dev open

#mongoose
npm install --save mongoose

#express
npm install --save express

#test tools
npm install --save-dev supertest
npm install --save-dev mocha
npm install --save-dev chai
npm install --save-dev -g karma-cli
npm install --save-dev jasmine
npm install --save-dev phantomjs
npm install --save-dev karma-jasmine@2_0
npm install --save-dev karma-phantomjs-launcher
```

## Setting
설치한 라이브러리에 대한 기본 셋팅

```sh
#bower
bower init
cat > .bowerrc #{"directory" : "client/lib"} #client

#git
git init
cat > .gitignore #https://www.gitignore.io/api/node,bower,grunt, bower경로 수정
```

## Hello World
server/app.js

```js
var express = require('express');
var app = express();
var server = require('http').createServer(app);
var port = process.argv[process.argv.length-1]; //get port number from argument

//error message
function onError(error) {
  switch (error.code) {
    case 'EACCES':
      console.error('port ' + port + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error('port ' + port + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
}

//success message
function onListening() {
  var addr = server.address();
  console.log('Listening on '+ addr.address + addr.port);
}

//serve static file (relative from running path)
app.use(express.static('client'));

server.on('error', onError);
server.on('listening', onListening);
server.listen(port);

module.exports = app;
```

client/index.html

```html
<!DOCTYPE html>
<html lang="ko">
<head>
	<meta charset="UTF-8">
	<title>Hello World!</title>
</head>
<body>
	<h1>Hello World!</h1>
</body>
</html>
```

package.js

```js
...
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "node ./server/app.js"
},
...
```

프로젝트 루트에서 다음과 같이 실행

```sh
npm start 5000
```
	
http://localhost:5000

## NODE_ENV
developement 모드인지 product 모드인지를 나눠서 실행.

app.js

```js
process.env.NODE_ENV = ( process.env.NODE_ENV && ( process.env.NODE_ENV ).trim().toLowerCase() == 'production' ) ? 'production' : 'development';
```


## Grunt 기본 작업

```sh
cat Gruntfile.js
```

### 실행할 작업
 - static html에 의존성 라이브러리를 삽입 -> wiredep
 - js 문법체크 -> jshint
 - 브라우저 오픈 -> open
 - 클라이언트 코드 변경시 새로고침 -> watch
 - 서버쪽 코드 변경시 재시작 및 새로고침 -> nodemon + watch
 - 디버그시 breakpoint & inspect -> node-inspector

### grunt-concurrent

nodemon, node-inspector, watch 등은 동시에 프로세스가 살아있어야 하므로 일반적인 grunt task로는 제대로 실행시킬 수 없다. 이 task들을 동시에 실행하기 위해서 grunt-concurrent 가 필요하다.

```js
concurrent: {
  options: {
    logConcurrentOutput: true
  },
  dev: {
    tasks: ['nodemon:dev', 'watch']
  },
  debug: {
    tasks: ['nodemon:debug', 'node-inspector', 'watch']
  },
  "debug-brk": {
    tasks: ['nodemon:debug-brk', 'node-inspector', 'watch']
  }
},

   ...

function nodemonReload(nodemon) {
  nodemon.on('log', function (event) {
    console.log(event.colour);
  });

  nodemon.on('config:update', function () {
    setTimeout(function() {
      require('open')('http://'+serverConf.express.host + ':' + serverConf.express.serverPort);
    }, 1000);
  });

  nodemon.on('restart', function () {
    setTimeout(function() {
      require('fs').writeFileSync('.rebooted', 'rebooted');
    }, 1000);
  });
}

function nodemonReloadWithInspector(nodemon) {
  nodemonReload(nodemon);
  nodemon.on('config:update', function () {
    setTimeout(function() {
      require('open')('http://'+serverConf.express.host + ':' + serverConf.express.inspectorPort);
    }, 1000);
  });
}
```

### live reload
모든 파일에 livereload.js 를 로딩하는 코드를 넣었다가 배포시에는 제거하는 작업은 성가실 수 있다. 이걸 자동화 하기 위해 서버 쪽 코드, express에 미들웨어 'connect-livereload'를 추가한다. 개발모드일 때만 livereload 코드를 로딩한다.

**주의** dynamic routing을 하기 전에 삽입해야 한다.

app.js

```js
if( process.env.NODE_ENV == 'development' ) {
  app.use(require('connect-livereload')({
    port: liveReloadPort
  }));
}
```

### Basic Grunt Tasks for dev & debug

Gruntfile.js

```js
grunt.registerTask('dev', ['wiredep', 'jshint', 'concurrent:dev']);
grunt.registerTask('debug', ['wiredep', 'jshint', 'concurrent:debug']);
grunt.registerTask('debug-brk', ['wiredep', 'jshint', 'concurrent:debug-brk']);
```

### 주의할 점
 - wiredep은 bootstrap 3.3.5 버전부터 css를 제대로 삽입 못하는 버그가 있다. bootstrap 3.3.4 버전을 사용하면 일단 해결은 된다.


## require.js + AngularJs + ng-ui-Router

### 계획

main.js --> app.js --> route.js --> controllers

### 라이브러리

```sh
bower install require --save
bower install angular-ui-router --save
```

### 코드

index.html

```html
<!DOCTYPE html>

<!--ng-app 을 넣지 않고 angualr.bootstrap() 을 이용-->
<html lang="ko"> 
<head>
  <meta charset="UTF-8">
  <title>Hello World!</title>

  <!--grunt-wiredep 으로 의존성을 로드한다.-->
  <!-- bower:css --><!-- endbower -->
  <!-- bower:js --><!-- endbower -->

  <!--require setup & angualr.bootstrap()-->
  <script src="js/main.js"></script> 

</script>
</head>
<body>
  <header>
    <h1>The Wall</h1>
    <nav>
      <ul>
        <li><a href="/">home</a></li>
        <li><a href="/user">user</a></li>
      </ul>
    </nav>
  </header>
  <!--ng-ui-router 작동을 위한 ng-view directive-->
  <main ui-view></main>
  <footer></footer>
</body>
</html>
```

js/main.js

```js
(function(){
  'use strict';

  requirejs.config({
    baseUrl:'js'
  });
  
  //app.js 로딩 && 문서 로딩후 bootstrap 수행
  requirejs(['app'], function () { 
    $(document).ready(function () { 
      angular.bootstrap(document, ['theWall']);
    });
  });
})();
```

js/app.js

```js
//route.js 로딩, 의존성 주입
define(['route'],function(route){
  var theWall = angular.module('theWall', ['ui.router']);
  route(theWall);
  return theWall;
});
```

js/route.js

```js
define(function() {
  function route(app){
    app.config(function($urlRouterProvider, $locationProvider, $stateProvider) {
      $locationProvider.html5Mode({
        enabled: true,
          requireBase: false
      });

      $urlRouterProvider.otherwise('/');
      
      $stateProvider
      .state('main', {
        url: '/',
        templateUrl: 'html/main.html'
      })
      .state('user', {
        url: '/user',
        templateUrl: 'html/user.html'
      });
    });
  }

  return route;
});
```

## Settings for Karma test with AngularJs + require.js

```sh
karma init

Which testing framework do you want to use ?
> jasmine

Do you want to use Require.js ?
> yes

Do you want to capture any browsers automatically ?
> PhantomJS

What is the location of your source and test files ?
> **/*spec.js

Should any of the files included by the previous patterns be excluded ?
>

Do you wanna generate a bootstrap file for) RequireJS?
> yes

Do you want Karma to watch all the files and run the tests on change ?
> yes
```

server/config/express.js

```js
module.exports = {
    ...
    testMain        : "test-main.js",
    ...
};
```

Gruntfile.js

```js
  bowerRequirejs: {
    app : {
      rjsConfig: serverConf.clientPath + '/js/main.js'
    },
    test : {
      rjsConfig: serverConf.testMain
    }
  },
```

karma.conf.js

```js
var config = require("server/config").express;

module.exports = function(config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', 'requirejs'],
    files: [
      config.testMain,
      {pattern: config.clientPath + '/**/*.js', included: false},
      {pattern: '**/*spec.js', included: false}
    ],
    exclude: [],
    preprocessors: {},
    reporters: ['progress'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_DEBUG,
    autoWatch: true,
    browsers: ['PhantomJS'],
    singleRun: false
  })
}
```






























## Bower componenets -> require.js 자동 등록

```sh
$ npm install grunt-bower-requirejs --save-dev
```

```js
/*Gruntfile.js*/
grunt.initConfig({
  bower: {
    target: {
      rjsConfig: 'app/config.js'
    }
  }
});
 
grunt.loadNpmTasks('grunt-bower-requirejs');
 
grunt.registerTask('default', ['bower']);
```
















