---
layout: post
title: "Super Test"
description: ""
category: "test"
tags: [test, http]
---
{% include JB/setup %}

super-agent 를 이용해서 HTTP통신을 통한 테스트를 쉽게 해준다.

> The motivation with this module is to provide a high-level abstraction for testing HTTP, while still allowing you to drop down to the lower-level API provided by super-agent.

## install

    npm install supertest --save-dev

## Basic example

supertest를 require 한 후에 express 를 인자로 넘겨준다. 그다음에는 체이닝으로 [Super Agent](#super-agent)의 API와 [exptect](#expect)를 마음대로 사용할 수 있다.

```js
var request = require('supertest')
  , express = require('express');

var app = express();

app.get('/user', function(req, res){
  res.send(200, { name: 'tobi' });
});

request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '20')
  .expect(200)
  .end(function(err, res){
    if (err) throw err;
  });
```

## [Super Agent](http://visionmedia.github.io/superagent/)

HTTP Request Library

Github link : https://github.com/visionmedia/superagent



### Some examples
```js
  .get('/user')
  .set('Accept', 'application/json')
```

```js
request
  .post('/api/pet')
  .send({ name: 'Manny', species: 'cat' })
  .set('X-API-Key', 'foobar')
  .set('Accept', 'application/json')
  .end(function(err, res){
    // Calling the end function will send the request
  });
```


## Expect

.end() 메쏘드를 사용하면 .expect() 가 실패해도 에러를 던지지 않고, end 메쏘드의 err 인자로 넘겨준다.

```js
.expect(status[, fn]) //Assert response status code.

.expect(status, body[, fn]) //Assert response status code and body.

.expect(body[, fn]) //Assert response body text with a string, regular expression, or parsed body object.

.expect(field, value[, fn]) //Assert header field value with a string or regular expression.

.expect(function(res) {})
```

## Usage with Test Tool & Assertion Engine

 - [Mocha](/test/2015/08/06/mocha/)
 - [Chai](/test/2015/08/06/mocha/#chai-api)

.end의 callback에서 테스트를 실행한다. 비동기이므로 [done을 이용해야 한다.](/test/2015/08/06/mocha/#Async Test)

```js
var request = require('supertest'),
    express = require('express');

var app = express();

describe('some api',function(){
    it('works asyncronously', function(done){
      request(app)
        .get('/api')
        .expect(200)
        .end(function(err, res){
          if (err) throw err;
          expect(res.body).to.be.true;
          done();
        });
    });
});
```
