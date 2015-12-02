---
layout: post
title: jQuery differ
categories: []
tags: []
published: True

---

##Promise pattern
Promise 패턴은 Javascript 에서 연속적인 비동기 처리를 원활히 하기 위해서 제안된 패턴이고, ES6에서부터는 정식으로 채택되었다. 

```js

//return Promise Object
new Promise( function(resolve, reject) {
    //async
    window.setTimeout(function () {
        if (param) {
            resolve("done");
        } else {
            reject(Error("fail"));
        }
    }, 0);
}).then(function (text) {   //set callback with chaining "then(success, error)""
    console.log(text);
}, function (error) {
    console.error(error);
});
```

##Polyfill
오래된 브라우저에서는 native Promise를 지원하지 않으므로 다양한 Library나 Framework에서 polyfill을 지원하고 있다.

###jQuery.deferred
다음은 jQuery.deferred를 이용해서 다중 ajax 요청을 보내고 모든 요청에 대한 응답이 성공했을 때 callback을 연속적으로 수행하는 예제이다.

```js

```

###AngularJs $q
...