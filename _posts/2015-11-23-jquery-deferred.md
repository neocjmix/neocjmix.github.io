---
layout: post
title: jQuery deferred
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

 - resolve, reject, notify 를 이용하면 done, fail, progress 시점에 예약된 callback을 실행시킬 수 있다.
 - resolveWith, rejectWith, notifyWith 를 이용해서 각 시점에 메시지를 전달할 수 있다.


```js
var promise = $.Deferred();

//resolve 되었을 때의 callback을 등록한다. Chaining을 할 수 있다.
promise.done(function(arg1, arg2){
    console.log("done", this, arg1, arg2);
}).fail(function(arg1, arg2){
    console.log("fail", this, arg1, arg2);
}).progress(function(arg1, arg2){
    console.log("progress", this, arg1, arg2);
}).always(function(arg1, arg2){
    console.log("always", this, arg1, arg2);
});


switch(prompt("1=resolve, 2=reject, 3=notify")){
    case "1":
        setTimeout(function(){
            promise.notifyWith(window, ["progress", "resolving in progress..."]);
        },1000);

        setTimeout(function(){
            promise.resolveWith(window, ["done"]);
        },3000);
        break;
    case "2":

        setTimeout(function(){
            promise.notifyWith(window, ["progress", "rejecting in progress..."]);
        },1000);

        setTimeout(function(){
            promise.rejectWith(window, ["fail", "Custom error"]);
        },3000);
        break;
}
```

jQuery의 $.ajax 를 실행하면, 해당 요청에 대한 promise가 리턴된다.
다음은 jQuery.deferred를 이용해서 다중 ajax 요청을 보내고 모든 요청에 대한 모든응답이 성공했을 때 callback을 연속적으로 수행하는 예제이다.

```js
//각각의 요청에 대한 promise들을 저장할 배열
var promises = [];

for(var i=0; i < 10; i++){
  //i를 위한 closure scope
  (function(i){

    //배열에 promise를 밀어넣는다.
    promises.push($.ajax({
      url: "/echo/json/",
      method: 'POST',
      data: {
       json:'{"key":'+i+'}'
      }
    }));

  })(i);   
  //closure 끝
}


//원래 when은 promise들을 연속된 parameter로 받도록 되어 있으므로,  --> $.when(promise1, promise2, ...)
//배열 형태의 promise들을 parameter로 넣어주기 위해서 Function.prototype.apply를 사용한다.
$.when.apply($,promises).done(function(){

    //모든 요청이 성공해서 응답을 받은 후에 실행된다.
    //각 promise에 대한 resolve 파라미터가 배열형태로 정리되어 arguments 에 다음 형태로 들어온다.
    //[[arg1, arg2, arg3], [arg1, arg2, arg3], [arg1, arg2, arg3], [arg1, arg2, arg3]]
    for(var i = 0; i < arguments.length; i++)
		document.write(arguments[i][1] + " : " + arguments[i][0].key + "<br />");

}).fail(function(){

    //하나라도 실패했을 시
    document.write("failed");
});
```

실패하든 성공하든, 모든 비동기 요청이 종료되었을 때 callback을 실행하기 위해서는 promise를 한겹 더 사용해야 한다.

```js
var AlldonePromises = [];

for(var i=0; i < 10; i++){

  (function(i){
    var promise = $.Deferred();

    //일부를 실패하도록 한다.
    var url = (i > 5) ? "/echo/json/" : "/failURL/";

    $.ajax({
      url: url,
      method: 'POST',
      data: {
       json:'{"key":'+i+'}'
      }
    }).always(function(){
      //성공하든 실패하든 resolve 한다.
      promise.resolveWith(this,arguments);
    });

    AlldonePromises.push(promise);
  })(i);   
}

$.when.apply($,AlldonePromises).done(function(){
    for(var i = 0; i < arguments.length; i++)
		document.write(arguments[i][1] + " : " + arguments[i][0].key + "<br />");
});
```


###AngularJs $q
...
