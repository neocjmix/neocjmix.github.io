---
layout: post
title: "AngularJs Controller : <small>$scope VS Controller as</small>"
description: ""
category: "angularjs"
tags: [angularjs]
---
{% include JB/setup %}

##Controller

>Controllers are "classes" or "constructor functions" that are responsible for providing the application behavior that supports the declarative markup in the template.   
 - https://docs.angularjs.org/guide/di

위 인용문에서 나타나듯이 Controller는 **생성자 함수**이다. 따라서 AngularJS 내부 어딘가에서 Controller를 실행하는 스코프에 인스턴스객체를 리턴한다.

AngularJs 코드를 보기 전에 먼저 전통적인 객체 생성자의 사용을 보자.

```js
function MyClass(){
    this.name = "Simple Object";
}

var instance = new MyClass();
document.getElementById("output").innerHTML = instance.name; //Simple Object
```

보통 생성자를 이용해서 인스턴스를 만들고, 생성자에서 this의 property로 선언 변수를 인스턴스에서 `.` 연산자로 참조해서 사용한다.

이 사실을 염두에 두고 먼저 가장 기본적인 Controller를 작성해본다.

다음은 모듈상에 기본적인 Controller를 정의하고 변수 하나를 정의하는 최소한의 코드이다.

```js
app.controller('MyController', function() {
    this.greeting = "Hello World!";
});
```
Controller는 생성자 함수라고 했으니까 convention에 맞도록 첫글자를 대문자로 지어준다.  
그리고... 왠지 느낌상 Controller 안에 변수를 정의하면 view에서 바인딩이 될 것 같지만...

```html
<section ng-controller="MyController">{​{MyController.greeting}}</section>
```

해보면 알겠지만 view에는 아무것도 나타나지 않는다. 왜냐하면 MyController는 생성자이지 인스턴스가 아니기 때문이다. MyController는 angularJS 내부 컨텍스트에서 실행되기 떄문에 현재로써는 서비스코드에서 생성된 인스턴스에 직접 접근할 방법이 없다.

일반적으로 view와 controller 내부 로직에서 정의한 변수들을 binding하기 위해서 `$scope` 객체 를 injection 한다.  
다음과 같이 수정하면 view에서 Hello World! 를 확인할 수 있다.

```js
app.controller('MyController', function($scope) {
    $scope.greeting = "Hello World!";
});
```

이 `$scope` 는 [$rootScope.Scope](https://docs.angularjs.org/api/ng/type/$rootScope.Scope) 클래스의 인스턴스이다. `$watch`, `$digest`, `$apply` 등의 메소드를 가지고 데이터를 [양방향 바인딩](/angularjs/2015/08/16/angularjs-two-way-binding/) 하는 바로 그 클래스이다.

###Child Scope & Parent Scope
Controller는 부모-자식 관계를 가질 수 있다. 이때 자식 Controller는 부모 Controller의 $scope에 접근할 수 있다.

이 과정이 어떻게 이루어지는지를 확인하기 위해 http://codetunnel.io/angularjs-controller-as-or-scope/ 에서 발견한 예제를 인용하겠다.

다음의 코드를 보자.

js

```js
app.controller('Parent', function($scope) {
    $scope.greeting = "Hello World!";
});

app.controller('Child', function($scope) {
});
```

html

```html
<section ng-controller="Parent">
    Parent <input type="text" ng-model="greeting">
    <div ng-controller="Child">
        Child <input type="text" ng-model="greeting">
    </div>
</section>
```

result
<iframe src="http://embed.plnkr.co/iGWen0fy0XkwcCpw2AmR/preview" frameborder="0"></iframe>

parent쪽에서 값을 입력하면 child쪽의 값도 잘 변경된다. 그런데 **반대로 Child쪽에서 값을 변경하면 binding이 깨지는 것을 볼 수 있다.**

왜 이런 현상이 발생할까?

###$scope의 상속관계

위 Child Controller를 다음과 같이 변경하고 브라우저에서 실행한 다음 console창을 보면,

js

```js
app.controller('Child', function($scope){
    console.log($scope);
});
```

console
![Imgur](http://i.imgur.com/Xmufl6D.png)

상위 `$scope` 객체 `__proto__`로 참조하고 있는 것을 알 수 있다. (`__proto__`는 코드에서 직접 참조할 수 없으므로 코드에서 참조할 수 있도록 $parent로도 연결 되어 있다. 하지만 상위 scope의 변수로의 접근은 `__proto__`에 의한 프로토타입 체인을 통해서 이루어진다.)

childScope에 greeting으로 binding되어 있는 input의 내용을 변경하면 `this.greeting = "변경된 내용"` 과 같이 작동하므로, 프로토타입 체인을 참조하지 않고, childScope에 새로운 greeting이 선언된다. 따라서 현재 스코프에 있는 greeting을 참조하게 됨으로써 프로토타입 체인이 깨지고 더이상 parent scope의 greeting과의 바인딩을 유지할 수 없게 되는 것이다.

###간단한 해결 방법

가장 간단한 해결책 중 하나는 binding할 변수를 객체의 프로퍼티로 설정하는 것이다.
```js
app.controller('Parent', function($scope){
    $scope.greeting = {
        text : "Hello World!"
    };
});

app.controller('Child', function($scope){
});
```

```html
<section ng-controller="Parent">
    Parent <input type="text" ng-model="greeting.text">
    <div ng-controller="Child">
        Child <input type="text" ng-model="greeting.text">
    </div>
</section>
```

이렇게 하면 내용을 변경할 때, `this.greeting.text = "변경된 내용"` 로 실행이 될텐데, `this.greeting` 객체가 없으므로 상속받은 객체의 프로퍼티를 참조하게 되어 문제없이 binding이 유지된다.

##Controller "as"

Anguler 1.2 이후에 나온 예제들를 처음 접하다 보면 대개 다음과 같은 코드를 보게 된다.

```js
app.controller('Controller', function() {
    this.greeting = "Hello World!";
});
```

```html
<div ng-controller="Controller as ctrl">
    {​{ctrl.greeting}}
</div>
```
 
맨 위에 작성했던 코드에 비해서 `Controller as ctrl`, `ctrl.greeting` 이 부분이 다르다. 이런 코드를 보다 보면 다음과 같은 생각이 들게 마련이다.

>"저 as는 왜 사용하는거지? 컨트롤러 이름이 길어서 타이핑하기 귀찮아서인가? 그렇다면 그냥 컨트롤러 이름을 짧게 ... 아니 애초에 그러면 안되는거 아냐?? 변수명을 성실하게 지어야 클린코드지! 구글 이 게으른놈들!!"

이것은 as의 역할을 잘 모르고 단순히 alias(별칭)로만 생각하기 떄문에 드는 착각이다. 맨 처음에 작성한, [인스턴스가 없어서 실패한 코드](#controller) 를 보고 위 코드를 다시한 번 보자. 

alias 하나를 추가했을 뿐인데 아까는 안되던 것이 이번엔 된다. **왜냐하면 `as ctrl`은 단순한 alias가 아니라 Controller를 생성자로 실행해서 리턴된 instance이기 때문이다.**

좀더 자세히 설명하자면 AngularJS 1.2 부터 ng-controller directive에 `as`로 인스턴스 변수명을 적을 수 있는 문법이 추가되었는데, AngularJs가 이 directive를 link할 때, Controller를 생성자로 인스턴스를 만든 후에 `$scope`에 집어넣어준다.

```js
console.log($scope.ctrl)
```

위와 같이 해보면 
```js
{
    greeting : "Hello World!"
}
```
라고 출력되는 것을 알 수 있다.

view template에서는 `$scope` 의 멤버에만(상속받은 멤버 포함) {​{ }} 를 이용해서 직접 접근할 수 있으므로 `{​{ctrl.greeting}}`  로 `"Hello World!"` 를 표시할 수 있는 것이다.

이 방법을 사용하면 아까 [위 코드](child-scope-amp-parent-scope)에서 겪었던 프로토타입 체인에 의한 바인딩 깨짐 현상도 자연히 해결된다. 왜냐면 [위 해결책](#간단한-해결-방법)에서 사용한 방법처럼, object 내의 변수를 참조하기 때문에 하위 스코프에 변수를 새로 만들지 않고 원래의 부모 스코프를 제대로 참조할 수 있게 된다.

이를 이용해서 부모-자식 스코프 간의 상속 구조를 간단히 할 수 있고, 원래 자바스크립트 객체가 가진 프로퍼티 구조를 활용할 수 있어서 더 이해하기 간편한 구조를 만들 수 있다.

다음은 `this` 와 `as`를 이용해서 자식 Controller에서 부모 Controller의 데이터를 binding 한 이다. 부모쪽에서 변경하든 자식쪽에서 변경든 문제없이 binding 된다.
<iframe src="http://embed.plnkr.co/azcQWtj9TbrYSy4TKaLI/preview" frameborder="0"></iframe>

###$scope 와 Controller instance

앞에서 view와 데이터를 [양방향 바인딩](/angularjs/2015/08/16/angularjs-two-way-binding/) 해주는 `$watch`, `$digest`, `$apply` 함수들은 `$scope`의 쏘드라고 했다. 그러면 변수를 $scope에 저장하지 않고 `this`에 저장하는 방식은 바인딩에 문제가 생기는것 아닌가 하는 의문이 들 수 있지만, `this`가 생성될 instance이고 이 인스턴스는 `$scope` 객체의 멤버로 할당되어 view에 노출되는 것이므로 `$scope`를 `$digest`할 때 문제없이 함께 업데이트 된다.메

##요약
 - view에서 사용하는 {​{ }} 문법으로는 `$scope` 의 멤버에만 직접 접근 가능하다.
 - '`Controller`는 `$scope`에 뭔가를 추가하는 어떤 것'이 아니고 **생성자 함수** 이다.
 - `Controller as instance` 에서 as 뒤의 단어는 alias가 아니라 생성된 인스턴스객체의 변수명이다.
 - `ng-controller` 디렉티브를 link 할때 생성된 인스턴스 객체는 `$scope`의 멤버로 등록되어 {​{ }} 로 접근가능하며 `$digest` 시 업데이트 된다.
 - 하위 Controller의 `$scope`객체는 상위 Controller의 `$scope` 객체에 **프로토타입 체인 연결**을 가진다.
 
 `$scope` 와 `this` + `as` 방법의 장단점은 http://odetocode.com/blogs/scott/archive/2014/08/11/thoughts-on-angular-controller-as-syntax.aspx 에 잘 정리되어 있다.

 이글에 달린 코멘트에 따르면 Angular 2.0 에서는 부자연스러운 프로토타입 상속을 가지는 $scope 객체는 사라지고 `Controller as` 에 더 가까운 방식을 사용한다고 한다.








































