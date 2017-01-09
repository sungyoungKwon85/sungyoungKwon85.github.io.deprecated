---
layout: post
title:  "redis #1"
date:   2017-01-09 17:10:53 +0900
categories: redis
---


## Redis 핵심정리 책 Study

.

# 시작하기

.

REmote Dictionary Server  
레디스는 고성능 `키-값 데이터 저장소인 NoSQL`이다.  
문자열, 해시, 리스트, 셋, 정렬된 셋, 비트맵, 하이퍼로그로그와 같은 강력한 데이터 타입 때문에 데이터 구조 서버라고도 불린다.  

.

레디스는 기본적으로 `모든 데이터를 메모리에 저장`하기 때문에 읽기와 쓰기가 매우 빠르다.  
(디스크에도 저장할 수 있다.)

.

영속성
* 바이너리 스냅샷으로 생성 -> **스냅샷**
* 시간에 따른 커맨드를 순서대로 저장, 파일로 생성 -> **저널링**

.

key expiration, transaction, publish/subscribe 기능 설정이 가능하다.

.

루아(lua) script 기능을 제공한다.

.

Redis 문서는 http://redis.io 에서 볼 수 있다.

.

**설치**
{% highlight ruby %}
$ curl -O http://download.redis.io/releases/redis-3.0.2.tar.gz
$ tar xzvf redis-3.0.2.tar.gz
$ cd redis-3.0.2
$ sudo make install
{% endhighlight %}

.  

**Hello Redis with Node**  
Redis는 여러 실행파일을 제공하는데, 우선 redis-server와 redis-cli를 살펴본다.
* redis-server : 실제 데이터 저장소
* redis-cli : command line interface

.

레디스는 기본적으로 6379 포트를 바인드하고, 독립 실행형 모드로 실행한다.

.

다음은 redis-cli를 사용해 redis-server에 접속하는 방법이다.
{% highlight ruby %}
$ redis-cli
> SET philosopher "socrates"  \\ 키 생성
OK
> GET philosopher  \\ 키값 읽기
"socrates"
{% endhighlight %}

.

패턴과 일치하는 키 리턴
{% highlight ruby %}
> KEYS [pattern]
{% endhighlight %}

.

node를 사용. redis 라이브러리를 설치한다.
{% highlight ruby %}
$ cd redis-essentials
$ npm install redis
{% endhighlight %}

.

hello.js  
{% highlight ruby %}
var redis = require("redis");
var client = redis.createClient();
client.set("my_key", "Hello World");
client.get("my_key", redis.print);
client.quit();  

$ node hello.js
{% endhighlight %}

.

**레디스 데이터 타입**
문자열
* 많은 커맨드를 가지고 여러 목적으로 사용된다.
* 텍스트(XML, JSON, HTML etc), 정수, 부동소수점, binary data(video, audio, image) 등이 저장된다.
* 512MB 초과 X  

* 예제
  {% highlight ruby %}
  $ redis-cli
  > MSET first "First key value" second "Second key value"
  OK
  > EXPIRE first 10
  (Integer) 1
  > MGET first second
  "First key value"
  "Second key value"
  > TTL first
  (Integer) 3 // remaining seconds
  {% endhighlight %}
