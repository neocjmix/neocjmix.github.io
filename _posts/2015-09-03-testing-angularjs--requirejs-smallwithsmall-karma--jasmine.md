---
layout: post
title: "Testing AngularJS + Require.js <small>with</small> Karma + Jasmine
"
description: ""
category: [test, angularjs]
tags: [test, angularjs, require.js, karma, jasmine]
---
{% include JB/setup %}

AngularJS를 rquire.js 로 구성하는 방법은 여러 가지가 나와 있고, 이를 테스트 가능하게 하는 방법도 여러가지이다. 가능한 심플한 방법으로 테스트 가능한 AngularJS + require.js 앱을 구성해 봤다. 코드가 길어보여도 막상 설명을 위한 주석을 다 지우고 나면 그리 길거나 복잡하진 않다.

##AngularJS + require.js
먼저 테스트 이전에 기본 구성을 시작한다.

###folders & files
```
 - js
     + controllers
         * SampleController.js .....테스트할 컨트롤러
     + lib .........................라이브러리 디렉토리
         * angular/...
         * angular-ui-router/...
         * requirejs/...
     + app.js ......................angular 모듈
     + main.js .....................require.js 설정
 - partials ........................ng-route 파트
     + sample.html .................메인페이지
 - index.html
```

###index.html
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>AngularJS+require.js</title>
    <script src="js/lib/requirejs/require.js" data-main="js/main.js"></script>
</head>
<body>
    <header><h1>AngularJS+require.js</h1></header>
    <main ui-view></main>
</body>
</html>
```


`<script src="js/lib/requirejs/require.js" data-main="js/main.js"></script>`에 주의. 특히 data-main 속성이 중요하다.  
`index.html` 에서는 `require.js` 만 로드하고, 메인 파일을 지정해준다. 다른 라이브러리는 모두 `main.js`에 의존성 모듈로 설정해서 동적으로 로드한다.

###main.js
require.js 작동을 위한 기본 설정 파일.

```js
requirejs.config({
    //다른 상대경로들의 베이스가 될 경로. 
    //'js' 디렉토리 안에 js파일들이 다 들어있으므로 'js'로 설정.
    baseUrl: "js",

    //사용할 모듈들
    //이름 : 상대 경로
    paths: {
        app         : "app",
        angular     : "./lib/angular/angular",
        jquery      : "./lib/jquery/dist/jquery",
        "angular-ui-router" : "./lib/angular-ui-router/release/angular-ui-router"
    },
    packages: [],

    //AMD를 따르지 않는 모듈들을 AMD 형식으로 노멀라이징 해준다.
    shim: {
        angular: {
            exports: "angular" //파라메터에 주입될 객체나 변수
        },
        jquery :{
            exports: "jQuery"   
        },
        "angular-ui-router": {
            deps: ["angular"] //해당 모듈의 의존성
        },
    }
});
```

###app.js
여기서 angular의 기본 모듈을 정의하는데, app.js역시 require에서 사용될 모듈이므로 AMD(Asynchronous Module Definition) 형식을 따라 작성되어야 한다. AMD에 관해서는 http://d2.naver.com/helloworld/12864, http://d2.naver.com/helloworld/591319 를 참고.

```js
//ui-route를 사용할 것이므로 main.js에서 미리 정의해둔 'angular-ui-router'를 의존성에 추가해준다.
define([
    'angular',             //의존성 모듈 목록
    'angular-ui-router'
], function(angular){      //팩토리 함수. 의존성 모듈을 파라키터로 순서대로 받는다.
    return angular.module('app',['ui.router']); //외부에 노출될 모듈을 리턴한다.
});
```

###controllers/SampleController.js

앞에서 모듈로 만든 app을 의존성으로 주입받아서 app에서 사용할 controller를 등록한다.

```js
define(['app'],function(app){   
    function Controller($scope){
        $scope.name = "World";
    }

    app.controller('SampleController', ['$scope', Controller]);
})
```

###route.js

ng-route 설정.  
역시 app을 의존성으로 받아와서 app.config을 통해서 route 설정을 한다.

```js
define(['app'],function(app){

    //ui-route 관련 설정
    app.config(function($urlRouterProvider, $locationProvider, $stateProvider) {

        $locationProvider.html5Mode({
            enabled: true,
            requireBase: false
        });

        $urlRouterProvider.otherwise('/');
        
        $stateProvider
        .state('main', {
            url: '/',
            templateUrl: 'partials/main.html', //`ui-view`에 들어갈 html 템플릿. 템플릿 파일은 뒤에서 작성한다.
            controller: 'SampleController' 
            //여기서 controller를 지정하려면, SampleController가 app에 등록되어 있어야 한다. 이 다음에 나오는 SampleController.js에서 등록한다.
        });
    });
})
```



###partials/main.html

`ui-view`에 들어갈 html 템플릿

```html
this is main Page.<br />
Hello, {{name}}!
```

###Bootstraping

일반적으로는 angualr.js파일과 app.js를 로딩하면 `body`에  `ng-app` 디렉티브를 걸고, 이걸 통해서 AngularJS가 모듈을 부트스트랩 하고 템플릿을 컴파일 하는데, 지금 의 경우, 모듈들을 동적으로 로딩하므로 `ng-app`을 사용하면 로딩시점이 맞지 않아 에러가 난다. 따라서 다음 코드를 이용해서 수동으로 bootstraping을 해줘야 한다.

```js
angular.bootstrap(document, ['app']);
```

이 코드를 언제, 어디에서 실행할 지 정하기 전에 잠깐 정리를 해보면 js 파일을 4개 만들었는데, 각각 역할은 다음과 같다.
 
 - main       : require 설정. 라이브러리등 의존성 모듈들을 등록.
 - app        : 앵귤러 모듈정의
 - controller : 컨트롤러를 모듈에 등록
 - route      : 모듈에 route 설정. 등록된 컨트롤러를 route에 할당함.
 
여기서 한가지 짚고 넘어갈 것은, route.js에서 `app.config`에 넘긴 설정을 담당하는 콜백 함수는 bootstrap 시점에서 실행된다는 것이다. 그러므로 bootstrap 하기 이전에 미리 app.config을 실행해 두어야 bootstrap할 때 제대로 실행된다. 그리고 방금 말한 callback 함수에서 controller를 url에 매핑하므로 controller등록도 bootstrap 하기 이전에만 하면 된다. (route.js보다는 빠르든 늦든 상관 없다. route.js 실행시점에 controller들을 바로 참조해서 매핑하는것이 아니라 콜백함수만 넘기기 때문이다.) 이를 바탕으로 위 코드들의 올바른 실행 순서를 살펴보면 다음과 같이 실행되어야 맞을 것이다.

```
main -> app -> (controller, route) -> bootstrap
```
현재 상태를 보면 실제로 실행되는 것은 main.js뿐이다. require.js가 로드하도록 `data-main`에 지정해두었으므로 가장 먼저 실행이 될 것이다. 그리고 app은 controller와 route에 의존성 설정을 해두었으므로 controller나 route가 실행되면 실행 되도록 되어 있다. bootstraping을 하려면 이 모든 모듈이 로딩 되어 있어야 하므로, 필요한 의존성 모듈을 주입 받아서 다음과 같이 실행하면 된다.

```js
requirejs([
    'angular',
    'app',
    'route',
    './controllers/SampleController'
], function (angular) {
    angular.bootstrap(document, ['app']);
});
```

위의 내용은, require.js가 main.js를 동적으로 로딩해서 config를 실행한 이후에 실행되어야 하므로 `main.js`의 config 뒤쪽에 넣어준다.

```js
/*main.js*/
requirejs.config({
    baseUrl: "js",
    paths: {
        app         : "app",
        route       : "route",
        angular     : "./lib/angular/angular",
        jquery      : "./lib/jquery/dist/jquery",
        "angular-ui-router" : "./lib/angular-ui-router/release/angular-ui-router"
    },
    packages: [],
    shim: {
        angular: {
            exports: "angular"
        },
        jquery :{
            exports: "jQuery"   
        },
        "angular-ui-router": {
            deps: ["angular"]
        },
    }
});

requirejs([
    'angular',
    'app',
    'route',
    './controllers/SampleController'
], function (angular) {
    angular.bootstrap(document, ['app']);
});
```


일단 이와 같이 완료하고 간단한 서버를 구동시키면 다음과 같이 작동하는 angularJS app을 확인할 수 있다.

```sh
$ python -m SimpleHTTPServer
```

![Imgur](http://i.imgur.com/N0AuNrG.png)


##Setting Karma
angularJS + require.js 를 사용한 앱 구성은 정리가 되었다. 이제 이걸 바탕으로 karma test를 설정해 보자.

###설치

```sh
$ npm init #enter를 연속으로 쳐서 설정 완료

$ npm install -g jasmine
$ npm install -g karma-cli
$ npm install -g phantomjs
$ npm install karma --save-dev
$ npm install karma-jasmine@2_0 --save-dev
$ npm install karma-phantomjs-launcher --save-dev

$ bower install angular-mocks
```

###설정
```sh
$ karma init #아래 옵션 설정에 주의

#Do you want to use Require.js ?
#> yes

#Do you wanna generate a bootstrap file for RequireJS?
#> yes
```

karma.conf.js 파일과  'bootstrap file for RequireJS', 즉 test-main.js 파일이 만들어 졌다. 세부적으로 설정을 조정하자.

###karma.conf.js
주석부분을 삭제하고 나면 아래와 같다. 수정할 부분은 `basePath`, `files` 다.

```js
module.exports = function(config) {
  config.set({
    basePath: 'js', //모든 설정의 상대경로 기준이 될 경로이다.
    frameworks: ['jasmine', 'requirejs'],
    files: [
      'test-main.js', //테스트에 직접 사용될 파일
      {pattern: '**/*.js', included: false} //서버에서 제공만 할 파일
    ],
    exclude: [],
    preprocessors: {},
    reporters: ['progress'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['PhantomJS'],
    singleRun: false
  })
}
```

files 옵션을 자세히 볼 필요가 있는데, 여기에 단순히 추가된 경로는 테스트할때 브라우저에 `<script>` 태그로 직접 코드가 삽입되어 로딩 되지만 `included:false` 옵션을 추가하게 되면 서버에서 serve 하기만 하고 직접 삽입 하지는 않는다. 이렇게 하는 이유는 `test-main.js` 파일만 테스트에 직접 삽입하고, library 등 다른 의존성 파일들을 `test-main.js` 에서 동적으로 삽입할 것이기 때문이다.

`basepath`를 `js`로 설정했으므로 test-main.js를 js 폴더로 이동시킨다.

```sh
mv test-main.js js
```

###js/test-main.js

이 파일을 열어보면 앞서 만들었던, main.js와 흡사한 구성을 가지고 있다. 즉 karma test시에 사용되는 main 파일이다.

```js

//앞부분은 테스트 파일을 모두 가져와서 allTestFiles라는 배열에 넣는 부분이다.
//TEST_REGEXP를 보면 **/*spec.js, **/*test.js 파일들을 로딩하게 되어 있다.

var allTestFiles = [];
var TEST_REGEXP = /(spec|test)\.js$/i; //다른 형식을 원하면 이 부분의 정규표현식을 변경하면 된다.

Object.keys(window.__karma__.files).forEach(function(file) {
  if (TEST_REGEXP.test(file)) {
    var normalizedTestModule = file.replace(/^\/base\/|\.js$/g, '');
    allTestFiles.push(normalizedTestModule);
  }
});


//앞서 main.js에서 했던 설정과 유사하게 require.js 설정을 수행한다.
require.config({
  
  baseUrl: '/base', //karma.conf.js에서 basePath로 설정한 Path의, 서버에서의 주소이다.
  deps: allTestFiles, //앞에서 만든 allTestFiles 를 로딩하는 부분
  callback: window.__karma__.start,

  //테스트에서 사용할 의존성을 설정한다.
  paths: {
    angular: "lib/angular/angular",
    "angular-mocks": "lib/angular-mocks/angular-mocks",
  },
  shim: {
    angular: { exports: 'angular' },
    jquery: { exports: 'jQuery' },
    "angular-mocks": { deps: ['angular'] }
  }
});
```

###js/Controllers/SampleController.spec.js

여기서 테스트 코드를 작성한다. 테스트 코드 역시 require.js 에 의해서 주입되는 모듈이므로, AMD 표준에 맞추어 작성한다.

```js
//필요한 "AMD"의존성을 주입받는다.
define(['./SampleController', 'angular', 'angular-mocks'],
  function(SampleController) {

    //test code 시작
    describe('SampleController', function () {
      var ctrl, scope;
      
      //각각 테스트를 시작하기 전에 필요한 "angular모듈"들을 주입받는다.
      beforeEach(inject(function($controller, $rootScope) {
        scope = $rootScope.$new(); //매번 새로운 scope에서 깔끔하게 테스트
        ctrl = $controller(SampleController, {$scope: scope}); //방금 생성한 scope 기반으로 controller를 mocking한다.
      }));

      it('has default name "World"', function() {
        expect(scope.name).toBe("World"); //beforeEach에서 생성한 scope에 대해서 controller가 제대로 작동했는지 test.
      });
    });
  });
```


###Test 실행
```sh
karma start
```

아래와 같이 success 메시지가 나오면 성공이다.

```sh
04 09 2015 16:08:13.058:WARN [karma]: No captured browser, open http://localhost:9876/
04 09 2015 16:08:13.069:INFO [karma]: Karma v0.13.9 server started at http://localhost:9876/
04 09 2015 16:08:13.076:INFO [launcher]: Starting browser PhantomJS
04 09 2015 16:08:14.332:INFO [PhantomJS 1.9.8 (Mac OS X 0.0.0)]: Connected on socket wUYW1RIdLF6EYQjBAAAA with id 17804098
PhantomJS 1.9.8 (Mac OS X 0.0.0): Executed 1 of 1 SUCCESS (0.004 secs / 0.008 secs)
```



##Adding & Testing Directive

Controller와 마찬가지로 directive도 require.js 로 주입되는 AMD 모듈로 만들 수 있다.
만들 directive는 name 이라는 attribute로 값을 전달받아서 인사를 하는 element directive다.

이번에는 test코드를 먼저 작성해보자. 일단 빈 directive를 등록하는 AMD를 만들자. 앞의 Samplecontroller와 유사하다.

```js
/*js/directives/SampleDirective.js*/
define(['app'],function(app){
  function sampleDirective(){
    return {};
  }

  app.directive('sampleDirective', sampleDirective);
  
  return sampleDirective;
});
```

test code

```js
/*js/directives/SampleDirective.spec.js*/

//directive와 필요한 AMD를 로드
define(['./sampleDirective', 'angular', 'angular-mocks'],
function(sampleDirective) {

  describe('sampleDirective', function () {
    //테스트에 사용할 전역변수
    var $compile, $rootScope, element;
    var name = 'World';

    //directive를 테스트하기 위해서는 html을 compile해야 하므로 module이 필요하다.
    //app.js는 test-main.js에 등록이 되어있고, SampleDirective에 의존성으로 설정했으므로  
    //이미 로드되어있다. ng-mock.module로 mocking할 수 있다.
    beforeEach(module("app")); 

    //inject로 angular 모듈을 주입받을 때, 나중에 집어넣을 변수이름을 짓기 용이하게 하기 위해서
    //앞뒤로 _를 붙여서 주입받을 수 있다.(자동으로 _를 제거해서 모듈을 주입해준다.)
    beforeEach(inject( function(_$compile_, _$rootScope_) {
      //따라서 아래와 같이 저장할 변수명을 좀더 편하게 지정할 수 있다.
      $compile = _$compile_;
      $rootScope = _$rootScope_;

      //매 테스트 하기 전마다 html을 compile하고 scope를 digest해준다.
      element = $compile('<sample-directive name="'+name+'"></sample-directive>')($rootScope);
      $rootScope.$digest();
    }));

    //directive가 잘 작동하는지 테스트한다.
    it('gets attr value', function() {
      expect($rootScope.name).toBe(name);
    });

    it('compile template correctly', function() {
      expect(element.html()).toContain("Hello, "+name);
    });
  });
});
```

`karma start`로 테스트를 실행해보면

```sh
PhantomJS 1.9.8 (Mac OS X 0.0.0) sampleDirective compile template correctly FAILED
  Expected '' to contain 'Hello, World'.
  Error: Expected '' to contain 'Hello, World'.

  ...

PhantomJS 1.9.8 (Mac OS X 0.0.0): Executed 3 of 3 (2 FAILED) (0.005 secs / 0.02 secs)
```

통과하지 못한다. `ctrl-c` 로 테스트를 **종료하지 말고** 별도의 에디터로 테스트에 맞도록 디렉티브 로직을 작성한다.

```js
(function(){
  'use strict';
  define(['app'],function(app){
    function sampleDirective(){
      return {
        restrict: 'E', //element directive
        template: 'Hello, {{name}}!', //scope의 name에게 인사하는 내용을 집어넣는다.
        link: function (scope, element, attr){
          scope.name = attr.name; // name attr을 받아서 scope에 넣는다.
        }
      };
      
    }

    app.directive('sampleDirective', sampleDirective);
    
    return sampleDirective;
  });
})();
```


다 작성하고 파일을 저장하면 karma가 파일을 watch하고 있다가 자동으로 테스트를 재수행 해준다.  
(karma.conf.js의 `autoWatch: true` 설정)   
로직을 제대로 작성했다면, 다음과 같이 success 메시지가 뜬다.

```sh
PhantomJS 1.9.8 (Mac OS X 0.0.0): Executed 3 of 3 SUCCESS (0.001 secs / 0.021 secs)
```





