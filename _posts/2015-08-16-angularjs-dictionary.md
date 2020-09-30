---
layout: post
title: "AngularJS Components for Test"
description: ""
category: [test, AngularJs]
tags: [AngularJS, test]
---
{% include JB/setup %}

##ngMock

[Mocha](/test/2015/08/06/mocha/) 나 [Jasmine](/test/2015/08/14/karma/#jasmine) 등의 테스트 프레임웍 자체에는 앵귤러 모듈을 로딩하거나 서비스를 주입받는 기능이 없으므로 ngMock을 사용해서 모듈을 로딩하고 의존성을 주입받는다.

Mocha, Jasmine 테스트시에만 사용사능하며, 전역변수에 등록되므로 다음과 같이 Test Hook에서 간단하게 사용할 수 있다.

###module
```js
beforeEach(module('myApplicationModule'));
```

###inject

```js
beforeEach( inject( function(_myService_){
  myService = _myService_;
}));

it('use Service', function() {
  myService.doStuff();
});
```

위의 예에서 보듯이, 나중에 서비스를 할당할때의 변수이름 짓기의 편리함을 위해서, 서비스를 주입받을 파라미터 이름은 `_` 로 감싸서 받을 수도 있다.

아래처럼 $injector 서비스를 주입받아서 다른 서비스를 주입하는 패턴으로, 파라미터 수를 줄일 수도 있다.

```js
var scope;

beforeEach(inject(function($injector, $sniffer) {
    scope = $injector.get('$rootScope');
});
```

##$controller

controller 인스턴스를 생성한다.

##angular.element & $compile

###element
jqLite를 사용할 수 있는 jQuery객체를 리턴하는 **함수**다. 테스트에서는 임의의 DOM을 만드는 용도로 사용할 수 있다.

```js
var doc, elem;
describe('form input element',function(){
    beforeEach(function() {
        doc = angular.element(
            '<form name="form">' +
            '<input type="text" ng-model="item" name="item" />' +
            '</form>'
            );

        elm = doc.find('input').eq(0);
    
        //...아래 코드에 계속
```

###$compile
주어진 DOM이나 HTML String을 Angular Template 으로 컴파일 해주는 **서비스**.
서비스이므로 DI를 받아서 사용한다.

```js
        //위의 코드에 이어서...
        scope = $injector.get('$rootScope');
        $compile = $injector.get('$compile');
        $compile(doc)(scope);
    });

    //...아래 코드에 계속
```

###test

```js
    it('linked with',function(){
        var testString = "item1";
        scope.item = testString;
        expect(elm).toBe(testString);
    });
});
```

DOM을 compile 한 후에는 위와 같이 scope에 연결시킬 수 있다.



browserTrigger
$sniffer
changeInputValue










