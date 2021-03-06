---
layout: post
title: "AngularJs Two way Binding"
description: ""
category: "AngularJs"
tags: []
---
{% include JB/setup %}
AngularJs 의 Scope Object는 양방향 바인딩을 구현하기 위해 대표적으로 다음의 세 가지 메소드를 가지고 있다.
 
 - $watch
 - $apply
 - $digest

이 메쏘드들이 어떻게 작동하는지 알아보자.

## Event-Loop
  
Angular는 브라우저의 기본 Event Loop 이외에 angular context를 따로 만든다.
이 angular context 안에서 angular 자체의 이벤트 루프를 돌리면서 변경된 값을 찾아서 DOM과 Scope를 동기화 하게 된다.

이것을 **Dirty Checking**이라 한다. 즉 값이 변경되었지만, 동기화가 이루어지지 않은 경우를 'dirty' 하다고 표현한다.
  
### $scope.$watch

AngularJs가 템플릿이 될 HTML String이나 DOM을 컴파일 할 때, 템플릿 내부의 directive들을 읽어들이면서 연결이 필요한 부분들을 Scope의 해당 부분과 link 작업을 진행한다.   
([$compile 서비스](/test/angular/2015/08/16/angularjs-dictionary/#angular-element-amp-compile)로 이 작업을 수동으로 할 수 있다.)

이 linking 작업은 `$scope.$watch` 를 통해서 이루어진다.

```js
$watch(watchExpression, listener, [objectEquality]);
```

`watchExpression`은 String으로 된 표현식이나 함수를 넘길 수 있다.

Angular는 watchExpression이 리턴하는 값이 변경되면 listener를 호출하게 된다.
이때 비교는 `!==` 연산자를 통해서 이루어지는데, `objectEquality` 옵션을 true로 변경하면 [angular.equals](https://docs.angularjs.org/api/ng/function/angular.equals)를 통해서 비교하게 된다.(object의 type과 properties 를 재귀적으로 검사한다.)

이렇게 linking된 데이터는 watch 목록에 들어가는데, `$scope.$$watchers` 를 통해 확인할 수 있다.

### $scope.$digest

```js
scope.$digest();
```

실제 Dirty Checking을 하는 메쏘드이다. 해당 스코프의 $scope.$$watchers에 등록된 watcher들을 순회하면서, 이전번 $digest가 실행되었을 때의 값과 다르면 `$watch` 에서 등록했던 listener 함수를 실행한다. 

**이때  함수 실행으로 인해서 watcher의 값이 변경되면 다시 처음부터 루프를 돈다.**

따라서 listener 함수가 바인드된 데이터를 다시 변경할 경우 무한루프를 돌 수도 있기 때문에 반복횟수에 limit가 걸려있으며 기본적으로 최대 10회이다.

또 해당 scope의 하위scope가 있으면 하위scope의 watcher도 모두 검사한다.
위 내용에 따라서 다음의 요인은 성능에 나쁜 영향을 칠 수 있다.
 
 - watchExpression 이 복잡한 경우
 - scope hirarchy가 복잡한 경우
 - listener 함수가 바인딩 된 데이터를 다시 변경할 경우

### $scope.$apply

```js
scope.$apply();
```

`$digest` 를 rootScope에서 실행한다. 즉 `$rootScope.$digest()` 와 같다.


## Linking, Dirty checking 시점

위 내용을 살펴보면 $watch를 통해서 바인딩을 목록에 추가하고 $digest가 실행되면 Dirty Checking을 해서 양방향 동기화 한다는 것을 알 수 있다. 그렇다면 $watch와 $digest는 어떤 시점에 실행되는지 알 필요가 있다.

### Linking

일단 $watch의 경우, template에 directive로 명시한 바인딩의 경우, 위에 적었듯이 compile할 때, scope에다가 연결 시켜준다. 이 역할을 수동으로 해주는 $compile 서비스의 usage를 살펴보면 다음과 같다.

```js
$compile(template)(scope);
```

템플릿을 컴파일 하면 scope를 link할 수 있는 함수가 리턴되고 이 함수의 파라미터로 연결할 scope을 넣어서 실행시키면 해당 scope의 $$whatchers 에 등록이 된다.

### Dirty checking

가장 중요한 것은 사실 Dirty checking 시점인데, 일반적인 JS Event Loop처럼 지속적으로 루프를 돌면서 체크해주면 참 좋겠지만, 이 동작을 경제적으로 하기 위해서 Angular는 몇가지 특정 시점에서만 체크한다.

#### 자동체크

 - ng-events (ng-click, ng-mousedown, ng-change, ng-checked, …) 발생시
 - $timeout
 - $http
 - $resource
 - $location

위 경우에 해당할 때 Angular Context로 들어가며 $scope.$apply()가 실행된다.
 
#### 수동체크

 - $scope.$apply()
 - $scope.$digest()

Angular를 사용해서 어떤 데이터가 바뀔 수 있는 거의 모든 경우에 Dirty Checking을 해주고 있다. 하지만 Angular events를 사용하지 않고 Native JS나 jQuery등을 이용해서 DOM이나 값을 변경해준 경우 Dirty Checking을 자동으로 해주지 않으므로, 바인딩이 끊겨 양방향 데이터가 동기화가 안된 상태로 남아있게 된다. 따라서 이런 경우에 $scope.$apply()나 $scope.$digest()를 통해서 수동으로 Dirty Checking을 해주어야 한다.

공식적으로 digest 보다는 apply를 권장한다고 하는데, 케이스가 명확하고 성능상 이점을 취하려면 digest를 사용하는것도 좋다.

(앞으로 Angular2 에서는 Observer를 사용하여 항상 자동으로 체크해준다고 한다.)





참고문서 : [AngularJs Dirty Checking 방법](http://sculove.pe.kr/wp/angularjs-dirty-checking-%EB%B0%A9%EB%B2%95/), [AngularJs: Scope와 데이터 바인딩[ $apply, $watch ]](http://www.nextree.co.kr/p8890/), [[AngularJs] $watch, $apply, $digest 개념잡기](http://mobicon.tistory.com/328)






























