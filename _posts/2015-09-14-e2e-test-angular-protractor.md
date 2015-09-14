---
layout: post
title: "e2e Testing AngularJS <small>with</small> Protractor + Jasmine"
description: ""
category: [test, angularjs]
tags: [test, angularjs, protractor, jasmine, e2e]
---
{% include JB/setup %}

앞의 [Testing AngularJS + Require.js with Karma + Jasmine](/test/angularjs/2015/09/03/testing-angularjs--requirejs-smallwithsmall-karma--jasmine/) 에 이어서 e2e 테스트 하는 방법을 기록한다.

테스트 툴은 [Protractor](https://angular.github.io/protractor)를 사용한다.  
e2e test를 위한 전체 구조는 다음과 같다.

```
[테스트 서버](angular앱 구동)
   ↑
____________protractor________
[브라우저]
   ↑
[브라우저 드라이버]
   ↑
[셀레니움 서버]
   ↑
[테스트코드](~.spec.js)
______________________________
   ↑
[설정파일](protractor.conf.js)
```

##Quick Start

###Protractor & Wabdriver Install

```sh
npm install -g protractor
```

위 명령어는 툴을 두개 설치하는데 하나는 물론 **Protractor** 이고, 추가로 **webdriver-manager**를 설치한다.

####webdriver-manager
webdriver-manager는 브라우저 자동화를 지원하는 [Selenium](http://www.seleniumhq.org/) 서버를 쉽게 실행시켜주는 매니저다. 다음과 같이 필요한 binary를 다운받아야 한다.

```sh
webdriver-manager update
```

서버를 시작해본다.

```sh
webdriver-manager start
```

엔터를 치면 서버가 종료되므로 가만히 두고, http://localhost:4444/wd/hub 로 접속하면 현재 가동되고 있는 Selenium 서버의 상태를 알 수 있다.


###Test Code 작성
[Testing AngularJS + Require.js with Karma + Jasmine](/test/angularjs/2015/09/03/testing-angularjs--requirejs-smallwithsmall-karma--jasmine/) 를 수행했다면 angularJS를 사용한 간단한 어플리케이션이 존재할 것이다.

프로젝트 루트 디렉토리에 e2e 테스트 파일을 저장할 디렉토리를 생성하고, `app.spec.js` 파일을 작성한다.

```js
describe('Protractor Demo App', function() {
  it('should have a title', function() {
    browser.get('http://localhost:5000');  //테스트할 앱이 구동됳 주소
    expect(browser.getTitle()).toEqual('Hello World!'); //browser title test
  });
});
```

###config 파일 작성
karma와 마찬가지로 config가 필요하다.

protractor.conf.js

```js
exports.config = {
  framework: 'jasmine2',
  seleniumAddress: 'http://localhost:4444/wd/hub', //위 selenium 서버 주소를 적어준다.
  specs: ['e2e/*.spec.js']
}
```

e2e 디렉토리의 모든 *.spec.js 파일들에 대해서 테스트를 진행한다.

###run test

여기까지 했으면 테스트를 진행할 수 있다. 테스트를 진행하기 위해서는 테스트할 앱이 서버에서 돌아가고 있어야 한다. express app이면 node로 구동시켜도 되고 아무튼 test code에 설정한 주소인 http://localhost:5000로 테스트할 angular app에 접속 가능하도록 한다.

```sh
python -m SimpleHTTPServer 5000
```

다음과 같이 config 파일을 이용해서 protractor를 구동시키면 seleniumAddress에 설정한 주소로 selenium server를 자동으로 띄우고, test를 실행한다.

```sh
protractor protractor.conf.js
```

test가 진행되면 chrome 브라우저가 잠깐 떴다가 테스트를 마치고 종료되고, 콘솔에 테스트 결과가 출력된다.

```sh
1 spec, 0 failures
Finished in 1.543 seconds
```

만약에 index.html이 다음과 같이 제대로 설정 되어 있다면 test는 무사히 통과할 것이다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    ...
```

##좀더 간편하게..

위의 예제에서는 일단 서버를 띄우고, config 파일 경로를 맞춰서 protractor를 실행해야 하는 번거로움이 있다. grunt를 통해 과정을 자동화 해 보자. 이미 기본적으로 앱을 서버에 올리기 위한 Gruntfile이 어느정도 셋팅 되어있다는 가정하에 진행한다.

일단 필요한 protractor 를 구동하는데 필요한 grunt 모듈을 설치한다.

```sh
npm install grunt-protractor-runner --save-dev
```

```js
/*Gruntfile.js*/
grunt.initConfig({
    ...
    protractor: {
      dev : {
        options: {
          configFile: "protractor.conf.js",
          keepAlive: true, //test가 실패해도 grunt task를 중단시키지 않는다.
          noColor: false //console에 color를 출력한다.
        }
      }
    },
    ...
});
...
grunt.loadNpmTasks('grunt-protractor-runner');
...
```

protractor는 실제 앱 구동과 동시에 실행시켜야 하기때문에 서버를 구동시킬 `grunt-nodemon`(서버측 프로그램이 node가 아니면 다른 모듈로 서버를 구동해야 한다), `grunt-concurrent`가 필요하고, 코드 수정시에 자동으로 다시 테스트 하려면 `grunt-watch`도 필요하다. 다음은 이런 모듈들을 연동해서 구성한 Gruntfile.js 예시이다.

```js
...
grunt.initConfig({
  ...
  //tasks에 등록된 task들을 동시에 병렬적으로 실행시킨다. (실행 순서는 array 순서대로)
  concurrent: {
    "test-e2e":{
      tasks: ['nodemon:test', 'protractor', 'watch:test-e2e']
    }
  },

  //node script를 실행한다. 소스에 변화가 생기면 재시작한다.
  nodemon: {
    test: {
      script: config.express.serverPath,
      options: {
        callback: function (nodemon) {
          //nodemon 재시작시 임의의 파일(.rebooted)을 변경해서 grunt-watch에 알려준다.
          nodemon.on('restart', function () {
            setTimeout(function() {
              require('fs').writeFileSync('.rebooted', 'rebooted');
            }, 500);
          });
        }
      }
    }
  },

  //protractor test 구동 task
  protractor: {
    dev : {
      options: {
        configFile: "protractor.conf.js",
        keepAlive: true,
        noColor: false
      }
    }
  },

  //위에서 nodemon이 restart될 때마다 변경하는 파일(.rebooted)을 watch하다가 test를 재시작해줌.
  watch: {
    "test-e2e": {
      files: ['.rebooted', front-end-file-Path + '/**/*'],
      tasks: ['protractor']
    }
  },
  ...  
})

...
grunt.loadNpmTasks('grunt-contrib-watch');
grunt.loadNpmTasks('grunt-concurrent');
grunt.loadNpmTasks('grunt-protractor-runner');
grunt.loadNpmTasks('grunt-nodemon');
...

grunt.registerTask('test-e2e', ['concurrent:test-e2e']);
```

위와 같이 구성하면 concurrent가 nodemon을 띄우고나서 protractor 테스트를 실행하고, watch까지 동시에 띄워준다. 서버 코드가 변경돼서, nodemon의 callback함수가 .rebooted파일을 다시 쓰거나, front-end 파일이 변경되면 watch가 protractor를 다시 실행해준다.

즉, 다음과 같이 실행하면,

```sh
grunt test-e2e
```

서버를 띄운 후 최초 한 번 e2e 테스트를 실행하고, 백엔드나 프론트엔드 코드가 변경될 때마다 테스트를 자동으로 다시 실행시켜준다.


##브라우저 다루기
[Protractor api](http://angular.github.io/protractor/#/api)를 이용해서 다양한 유저 동작 시나리오를 만들고 결과를 테스트할 수 있다. 앞에서의 test code는 title만 확인하는 간단한 테스트였다. 좀더 확실한 테스트를 하기 위해서 [Testing AngularJS + Require.js with Karma + Jasmine](/test/angularjs/2015/09/03/testing-angularjs--requirejs-smallwithsmall-karma--jasmine/)에서 작성한 앱에 기반해서 route와, controller와 directive가 잘 작동하는지 확인해보겠다.

Protractor api를 사용해서 일련의 테스트 코드를 작성한 예

```js

/*e2e/app.spec.js*/

describe('Protractor Demo App', function() {
  
  beforeEach(function(){
    browser.get('http://localhost:5000');
  });


  it('should have a title', function() {
    expect(browser.getTitle()).toEqual('Hello World!');
  });


  it('should route paths', function() {
    browser.setLocation('user');
    expect(browser.getCurrentUrl())
    .toBe('http://localhost:5000/user');

    expect($("main > span").getText())
    .toBe('User');    
  });

  it('should present default value of name input', function() {
    var nameModel = element(by.model("sampleController.name"));
  
    expect(nameModel.isPresent())
    .toBe(true);

    expect(nameModel.getAttribute('value'))
    .toBe('World');
  });

  it('present name and greets correctly when name is entered.', function() {
    var nameModel = element(by.model("sampleController.name"));
    nameModel.clear().sendKeys("BoB");

    expect(nameModel.getAttribute('value'))
    .toBe('BoB');

    expect($('#name').getText())
    .toBe('BoB');

    expect($('Sample-directive').getText())
    .toBe('Hello, BoB!');
  });
});
```

위의 테스트를 통과하려면 기존 소스를 다음과 같이 고쳐야 한다.

partial/main.html

```html
<input type="text" ng-model="sampleController.name"><br>
<span id="name">{{sampleController.name}}</span><br>
<sample-directive person="sampleController" />
```

directives/sampleDirective.html

```js
define(['app'],function(app){
  function sampleDirective(){
    return {
      restrict: 'E',
      template: 'Hello, {{person.name}}!',
      scope : {
        person : '='
      }
    };
    
  }

  app.directive('sampleDirective', sampleDirective);
  
  return sampleDirective;
});
```

테스트 결과

```sh
4 specs, 0 failures
    Finished in 3.058 seconds
    [launcher] 0 instance(s) of WebDriver still running
    [launcher] chrome #1 passed
```

ctrl+c를 눌러 테스트를 종료하면 된다.