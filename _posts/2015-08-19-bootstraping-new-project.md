---
layout: post
title: "Step By Step MEAN Stack Dev Environment"
description: ""
category: "tools"
tags: [yeoman, yo, generator]
---
{% include JB/setup %}
##Directory
```sh
#create base directory
mkdir dev-mean
cd dev-mean
mkdir server
mkdir client
mkdir e2e
```

##Install
필요한 프로그램, 라이브러리 모듈 전부 설치

```sh
#install global programs
brew install node
brew install mongodb
npm install -g grunt-cli
npm install -g nodemon
npm install -g protractor

#create package.json
npm init

#bower and frontend libraries
npm install --save-dev bower 
bower init
bower install angular bootstrap jquery --save
bower install angular-mocks --save-dev

#grunt and plugins
npm install --save-dev grunt
npm install --save-dev grunt-bower-install
npm install --save-dev grunt-contrib-jshint
npm install --save-dev grunt-contrib-watch
npm install --save-dev grunt-node-inspector

#mongoose
npm install --save mongoose

#express
npm install express

#test tools
npm install --save-dev supertest
npm install --save-dev mocha
npm install --save-dev chai
npm install --save-dev karma-cli
npm install --save-dev jasmine
npm install --save-dev phantomjs
npm install --save-dev karma-jasmine@2_0
npm install --save-dev karma-phantomjs-launcher
```

##Setting
설치한 라이브러리에 대한 기본 셋팅

```sh
#git
git init
cat > .gitignore #https://www.gitignore.io/api/node,bower,grunt

#karma
karma init karma.conf.js #(**/*.spec.js)
```

##Hello World
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

##Basic Grunt Task for Running and Debugging

```sh
#(re)start server with watch/livereload
npm install grunt-express-server --save-dev
#open web browser
npm install --save-dev grunt-open

touch Gruntfile.js
```

Gruntfile.js

```js

```

