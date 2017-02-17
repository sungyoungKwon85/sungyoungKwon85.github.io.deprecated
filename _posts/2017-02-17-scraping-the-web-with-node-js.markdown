---
layout: post
title:  "scraping-the-web-with-node-js"
date:   2017-02-17 13:10:53 +0900
categories: scraping-the-web-with-node-js
---


Refer to..  
https://scotch.io/tutorials/scraping-the-web-with-node-js  
http://pyrasis.com/nodejs/nodejs-HOWTO  

<br><br>

### 1.  
크롤링에 앞서 NodeJS에 대해 간략히 스터디 한다.

<br><br>

### NPM  
npm은 Node.js로 만들어진 모듈을 인터넷에서 받아서 설치해주는 패키지 매니저.  
수 많은 Node.js 모듈들이 이미 만들어져있어 npm으로 설치해서 쓰면 된다.  

<br><br>

### Single Thread Model and Non-blocking I/O  
NodeJS의 가장 큰 특징은 Single thread model과 Non-blocking I/O 라는 것이다.  
Multi thread model의 경우, 동기화 문제로 인해 학습벽이 높아 생산성 저하로 이어지기 마련이고 이는 곧 비용 증가로 이어진다.  
NodeJS가 Single thread model과 Non-blocking I/O을 채택한 이유는 위의 이유 뿐만 아니라 Javascript의 언어적 특성 때문이기도 하다.  

무슨말이냐하면.. 아래의 코드처럼 Javascript는 1을 실행 후 기다리지 않고 바로 2를 실행한 뒤 3을 실행한다.
시간이 오래 걸리는 작업은 worker thread로 보내버리고 main thread는 코드를 계속 실행하는 Non-blocking 방식으로 동작한다.  
(함수 호출해서 리턴값을 받거나 그냥 함수만 호출하는 경우는 Blocking 방식으로 실행된다.)  
{% highlight ruby %}
var fs = require('fs');

fs.readFile('./컴배콤.txt' /* 1 */, function (err, data) {
  console.log(data); // 3
});

console.log('Hello JavaScript'); // 2
{% endhighlight %}

<br><br>

### NodeJS install for mac
https://nodejs.org/en/  

{% highlight ruby %}
Node.js was installed at

   /usr/local/bin/node

npm was installed at

   /usr/local/bin/npm

Make sure that /usr/local/bin is in your $PATH.
{% endhighlight %}

<br><br>*

### helloworld  

[app.js]  
console.log('hello world');  

$ node app.js  
> Hello world  

<br><br>

### Simple Web server  

[app.js]  
{% highlight ruby %}
var http = require('http');

var server = http.createServer(function (req, res) {
  res.writeHead(200, { 'Content-Type' : 'text/plain' });
  res.end('Hello World');
});

server.listen(8000);
{% endhighlight %}

http://127.0.0.1:8000 접속하면 Hello world를 확인할 수 있다.  
참고로 윈도우8이 80포트를 사용해서 8000으로 했다(윈도우8을 사용하는 건 아니지만..)  

<br><br>

### Web server by express  

express는 NodeJS에서 가장 유명한 framework module이다.  
이것을 이용하면 더 간단하게 웹서버를 만들 수 있고, 다양한 template engine과 기능들을 사용할 수 있다.  

$ mkdir HelloWebServer  

[app.js]  
{% highlight ruby %}
var express = require('express')
  , http = require('http')
  , app = express()
  , server = http.createServer(app);

app.get('/', function (req, res) {
  res.send('Hello /');
});

app.get('/world.html', function (req, res) {
  res.send('Hello World');
});

server.listen(8000, function() {
  console.log('Express server listening on port ' + server.address().port);
});
{% endhighlight %}

~/HelloWebServer$ npm install express  

**주의점** - app.js가 있는 directory에서 npm install express 명령을 실행할 것.  
app.js 가 있는 directory에 node_modules directory가 생성되고 express 모듈이 설치된다.  


~/HelloWebServer$ node app.js  

http://127.0.0.1:8000/    
http://127.0.0.1:8000/world.html  

<br><br>

### Template Engine  

express로 웹 서버를 만들어도 많은 경로/파일을 일일이 정의하기는 힘들다. Template Engine을 사용하면 PHP. ASP, JSP처럼 서버에서 HTML을 동적으로 생성할 수가 있다. 게다가 간단한 문법으로 웹페이지를 만들 수 있다.  
아래는 Template Engine의 대표적인 녀석들이다. 자세한 내용은 참조 링크를 참고하자.  
- EJS  
- Jade  

<br><br>

### 실시간 통신하기  

NodeJS에서 사용할 수 있는 대표적인 실시간 통신기술은 아래와 같다.  
- TCP  
- WebSocket  
- socket.io  
자세한 내용은 참조 링크를 참고하자.  































.
