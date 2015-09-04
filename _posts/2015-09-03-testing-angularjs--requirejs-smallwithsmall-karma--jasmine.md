---
layout: post
title: "Testing AngularJS + Require.js <small>with</small> Karma + Jasmine
"
description: ""
category: 
tags: []
---
{% include JB/setup %}

AngularJS를 rquire.js 로 구성하는 방법은 여러 가지가 나와 있고, 이를 테스트 가능하게 하는 방법도 여러가지이다. 구글링을 해보면 관련 주제에 대해 다양한 포스팅이 있는데, 대체로 상당히 복잡하다.

가능한 심플한 방법으로 테스트 가능한 AngularJS + require.js 앱을 구성해 봤다. 코드가 길어보여도 막상 설명을 위한 주석을 다 지우고 나면 그리 길거나 복잡하진 않다.

##AngularJS + require.js
먼저 테스트 이전에 기본 구성을 시작한다.

###folders & files
```
 - js
     + controllers
         * SampleController.js .....테스트할 컨트롤러
     + lib .........................라이브러리 디렉토리
         * angular
         * angular-ui-router
         * requirejs
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
(function(){
    'use strict';

    requirejs.config({
        //다른 상대경로들의 베이스가 될 경로. 
        //'js' 디렉토리 안에 js파일들이 다 들어있으므로 'js'로 설정.
        baseUrl: "js",

        //모듈 이름 : 상대 경로
        paths: {
            app         : "app",
            angular     : "./lib/angular/angular",
            jquery      : "./lib/jquery/dist/jquery",
            "angular-ui-router" : "./lib/angular-ui-router/release/angular-ui-router"
        },
        packages: [],

        //AMD를 따르지 않는 모듈들에 대한 정의
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
        },
        //main.js 자체의 의존성. app.js를 등록해두었으므로
        //자동으로 html에 삽입된다.
        deps: ["app"] 
    });
})();
```

###app.js
여기서 angular의 모듈 정의와 config을 해줄 수 있다. require.js의 deps 설정에 의해서 로드된다. 
require.js에서 사용되는 모듈들은 AMD(Asynchronous Module Definition) 표준을 따라 작성되어야 한다. AMD에 관해서는 http://d2.naver.com/helloworld/12864, http://d2.naver.com/helloworld/591319 를 참고.

```js
(function(){
    'use strict'
    
    //필요한 의존성을 주입 받는다. main.js에서 설정한 모듈 이름을 사용한다.
    //AMD 지원 모듈이 아닐 경우 해당 모듈 설정에서 exports에 설정한 객체를 파라미터로 받는다.
    define(['angular', 'jquery', 'angular-ui-router'], function(angular, jQuery){
        //ui-router 사용을 위해 define에서 의존성을 설정하면,
        //ui.router의존성을 module에 주입할 수 있다.
        var app = angular.module('app',['ui.router']);
        app.config(function($urlRouterProvider, $locationProvider, $stateProvider) {

            //ui-router 관련 설정
            $locationProvider.html5Mode({
                enabled: true,
                requireBase: false
            });

            $urlRouterProvider.otherwise('/');
            
            //접속시 기본 화면으로 partials/main.html을 보여준다.
            $stateProvider
            .state('main', {
                url: '/',
                templateUrl: 'partials/main.html',
                //controller는 잠시 후에 AMD 모듈로 따로 뺄 것이다.
                controller: function($scope){
                    $scope.name = "World";
                }
            });
        });

        jQuery(document).ready(function(){
            angular.bootstrap(document, ['app']);
        })
    });

    
})();
```

###partials/main.html

`ui-view`에 들어갈 html 템플릿

```html
this is main Page.<br />
Hello, {{name}}!
```

일단 이와 같이 완료하고 간단한 서버를 구동시키면 다음과 같이 작동하는 angularJS app을 확인할 수 있다.

```sh
$ python -m SimpleHTTPServer
```

![Imgur](http://i.imgur.com/N0AuNrG.png)

###SampleController as AMD Module
앞에서 메인 페이지에 사용되었던 controller 함수를 require.js로 동적로딩할 수 있도록 AMD 표준을 구현한 모듈로 만들어보자.

js/controllers/SampleController.js

```js
//id, 의존성 부분은 비워두고, 팩토리 함수를 작성한다.
define(function(){
    //리턴하는 함수에서 angular module을 DI 받아서 controller를 등록해준다.
    return function(app){
        //controller 등록시 이름은 나중에 app.js에서 사용되므로 혼동되지 않도록 파일명과 일치시키는 것이 좋다.
        app.controller('SampleController', ['$scope', function($scope){
            $scope.name = "World";
        }]);
    }
})
```

app.js

수정된 부분만 적는다.

```js
    ...    
    define([
        'angular',
        'jquery',
        'controllers/SampleController', //Samplecontroller 의존성 추가
        'angular-ui-router'
    ], function(angular, jQuery, SampleController){ //파라미터에 순서대로 주입
        var app = angular.module('app',['ui.router']);
        
        //Samplecontroller 에 angular module을 주입하면 controller로 등록된다.
        SampleController(app);
        
        app.config(function($urlRouterProvider, $locationProvider, $stateProvider) {
            
            ...

            $stateProvider
            .state('main', {
                url: '/',
                templateUrl: 'partials/main.html',
                controller: 'SampleController as sampleController'  //등록된 controller를 route에 매핑
            });
        });
    ...
```

controller에서 this를 사용하지 않고, $scope만 사용할 경우, `'SampleController as sampleController'` 에서 as 이하는 생락할 수도 있다.  
require.js를 이용해서 controller를 모듈로 만드는 방법에는 이 방법 외에도 여러가지가 있는데, 위와 같이 하는 것이 간단한 구조를 만드는데 제일 도움이 되었다.

다음은 다른 코드를 참고해서 시도했다가 실패한 방법들이다.

 - 팩토리 메쏘드에서 controller 함수만 리턴해서 app.js에서 controller를 직접 등록하는 방법

    ```js
    /*controllers/Samplecontroller.js*/
    define(function(){
        return function($scope){
            ...
        }
    });
    
    /*app.js*/
    define(['Samplecontroller'],function(Samplecontroller){
        ...
        app.controller('SampleController',['$scope'], SampleController)
    });
    ```

    이런 방법은 return이 깔끔하게 controller 함수만 나와서 깔끔하긴 하지만, 컨트롤러 등록하는 app.js 의 코드에서 주입하는 의존성을 실제 컨트롤러 함수의 파라미터와 맞추어 줘야 하는 문제가 있다.

 - Sample Controller의 define에서 'app'을 의존성으로 주입 받아서 SampleController를 DI만 받아도 controller가 등록되게 하는 패턴
 
    ```js
    /*controllers/Samplecontroller.js*/
    define(['app', function(app){
        function controller($scope){
            ...
        }

        app.controller(['$scope', controller]);
        return controller;
    });

    /*app.js*/
    //의존성 순환 -> error!!
    define(['angular', 'controllers/Samplecontroller', function(app){

    ```

    controller 함수를 리턴할 수도 있고, controller 등록을 바로 해주므로 편할 것 같긴 하지만, app.js에서 의존성 주입을 통해서 사용하려 할 경우 app -> SampleController -> app 으로 의존성이 순환되므로 에러가 나므로, 해당 컨트롤러는 다른 방법으로 사용할 수 밖에 없다.

###route.js
위 app.js의 다음 부분을 다시 보자.

```js
    ...    
    define([
        'angular',
        'jquery',
        'controllers/SampleController',
        'angular-ui-router'
    ], function(angular, jQuery, SampleController){
    ...
```

라이브러리 의존성과 controller 모듈이 섞여 있어 구분이 어렵고, 나중에 controller나 directive등이 많아질 경우에도 순서를 맞추어서 파라미터에 넣어줘야 하는 등, 문제가 남아있는 코드이다.

각 url을 routing 하는 부분을 별도의 모듈로 분리하면서 이 문제를 해결할 수 있다.

먼저 js 디렉토리에 route.js를 생성한다.

```sh
touch js/route.js
```

main.js에서 모듈로 등록하고, app.js의 routing 관련 부분을 옮겨준다.

```js
/*main.js*/
    ...
    paths: {
        app         : "app",
        route       : "route",  //새로운 모듈로 등록
       ...
    }
    ...

/*app.js*/

(function(){
    'use strict'
    
    define([
        'angular',
        'jquery',
        'route',    //새로운 모듈을 주입받는다.
        'angular-ui-router'
    ], function(angular, jQuery, route){
        var app = angular.module('app',['ui.router']);

        route(app); //기존 로직을 삭제하고 대신 route 모듈로 app을 넘긴다.

        jQuery(document).ready(function(){
            angular.bootstrap(document, ['app']);
        })
    });
})();


/*route.js*/

(function(){
    'use strict'

    define([
        //route에서는 controller들만 깔끔하게 주입받을 수 있다.
        'controllers/SampleController',
        'controllers/SampleController2',
        'controllers/SampleController3',
        ], function(
            //파라미터로 한번 더 가져온다.
            SampleController1,
            SampleController2,
            SampleController3
        ){
        return function(app){

            //주입받은 controller 등록 함수에 app을 넘겨준다.
            SampleController1(app);
            SampleController2(app);
            SampleController3(app);

            app.config(function($urlRouterProvider, $locationProvider, $stateProvider) {
                
                ...
                
                //route에 매핑시켜준다.
                $stateProvider
                .state('main', {
                    url: '/',
                    templateUrl: 'partials/main.html',
                    controller: 'SampleController'
                })
                .state('sub2', {
                    url: '/sub2',
                    templateUrl: 'partials/sub2.html',
                    controller: 'SampleController2'
                })
                .state('sub3', {
                    url: '/sub3',
                    templateUrl: 'partials/sub3.html',
                    controller: 'SampleController3'
                })
                
                ...
```

위 route.js 에는 의존성 부분에 주입받은 controller들만 깔끔하게 넣을 수는 있지만 여전히 parameter를 중복적으로 적어주어야 하고 일일이 app을 넘겨서 실행해야 하는 불편함이 있다. `arguments`를 이용해서 이 부분을 리팩토링하면 좀더 간결해진다.

```js
/*route.js*/

(function(){
    'use strict'

    define([
        //여기에서 한번만 의존성 모듈의 경로를 적어준다.
        'controllers/SampleController',
        'controllers/SampleController2',
        'controllers/SampleController3',
        
        ], function(){
            //따로 파라메터를 적지 않아도 의존성 모듈들은 arguments 로 차례대로 들어온다. 
            //리턴할 내부 함수에서 사용가능하도록 arguments를 참조하는 변수를 만들어둔다.(클로저)
            var controllers = arguments;

            return function(app){
                
                //모든 controller를 순회하면서 app에 등록한다.
                controllers.forEach(function(controller){
                    controller(app);
                });
        ...
```

##Setting Karma








