---
layout: post
title:  "websocket"
date:   2017-05-17 15:10:53 +0900
categories: websocket
---

# WEBSOCKET이란?  

WebSocket은 HTTP의 단점을 보안하기 위해 등장했다.  
browser 버전에 따라 지원하지 않으므로 유의해야 한다.  

<br>

### HTTP의 단점이 뭔데?  
- HTTP는 `Servie-Client간 접속을 유지하지 않고`, 한 번에 `한 방향으로만 통신`하는 Half-duplex방식이다.  
- 또한 `Header에 많은 데이터를 내포`하고 있다.  
- Client(ex.browser)가 요청을 보내지 않아도 Server가 데이터를 보내주는 기능 구현에 한계가 있다.  

<br>

WebSocket이 등장하면서  
- ActiveX를 사용하지 않고도 TCP/IP 소켓 통신을 구현할 수 있게 되었다.  
- Network의 과부하를 줄인다.(ex.HTTP Header 800Kb~N000Kb -> byte수준)  

<br>

물론 HTTP를 대체하는 건 아니다.  
다만 Messaging, Transaction 및 High Traffic, low time limit 환경에서 유리하단다.  
ex. JMS(Java Messaging Service), RMI(Remote Method Invocation), XMPP(Extensible Messaging and Presence Protocol)  

<br>

WebSocket은 접속확립에 HTTP를 사용하고, 그 후의 통신은 WebSocket 독자의 프로토콜로 이뤄진다.  
장시간 접속을 전제로 하므로 접속된 상태기만 하면 Server/Client로부터의 송신이 가능하다.  
통신시 지정되는 URL은 ws://www.sample.com 과 같은 형식이다.  

<br>

### WebSocket이 필요한 경우?
- `실시간 양방향 데이터 통신`이 필요한 경우. ex)Gmail
- `많은 수의 동시 접속자`를 수용해야 하는 경우
- `브라우저에서 TCP 기반의 통신으로 확장`해야 하는 경우
- Cloud환경이나 Web을 넘어 `SOA`(Service Oriented Architecture)로 확장해야 하는 경우

<br>

`Spring4에서 WebSocket 사용이 가능`해졌다.  

{% highlight ruby %}
# pom.xml에 dependency 추가
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-websocket</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

# org.springframework.web.socket.handler.TextWebSocketHandler 클래스를 상속받아 websocket handler를 생성
# Servlet context에 websocket handler 등록


# view.html
websocket = new WebSocket(wsUri);
websocket.onopen = function(evt) {
    onOpen(evt)
};
websocket.onmessage = function(evt) {
    onMessage(evt)
};
websocket.onerror = function(evt) {
    onError(evt)
};
{% endhighlight %}



<br><br><br>



### 기초..  
**프로토콜 계층 별, 주로 사용되는 데이터 단위 명칭**  
- 응용 계층 : 메시지, 데이터  
- 표현 계층 : 메시지, 데이터  
- 세션 계층 : 메시지, 데이터  
- 전송 계층 : 세그먼트  
- 네트워크 계층 : 패킷, 데이터그램  
- 데이터링크 계층 : 프레임  
- 물리 계층 : 비트  

<br>

**TCP/IP**  
- TCP : Transmission Control Protocol, 정보 패킷 차원에서 다른 인터넷 노드와 메시지를 상호 교환하는데 필요한 규칙 사용    
- IP : Internet Protocol, 인터넷 주소 차원에서 메시지를 보내고 받는데 규칙을 사용  

<br>

**HTTP(Hyper Text Transfer Protocol)**  
Server가 html 문서를 User에게 전달하는 목적  
TCP/UDP를 사용  
80 port  
request/response  

<br>

**플래시(플렉스), 자바애플릿(자바FX), ActiveX , 실버라이트 그리고 Web Socket**  
갈수록 동적인 표현과 뛰어난 상호작용이 요구되었고 이로 인해 여러 새로운 기술이 탄생되었다.  
하지만 이들은 웹에서 화려한 동작과 뛰어난 상호작용을 보장하지만 순수 웹 환경이 아니라 별도의 런타임을 플러그 인 형태로 브라우저에 설치해야 사용 가능하다.  
HTML5는 그 주요 목적 중 하나인, 플러그 인 없는 일관되고 표준화된 웹 응용 환경이라는 기치하에 많은 참신한 스펙들이 개발되었다.   
그 중 순수 웹 환경에서 실시간 양방향 통신을 위한 스펙이 바로 '웹 소켓(Web Socket)' 이다.  
`웹 소켓은 웹 서버와 웹 브라우저가 지속적으로 연결된 TCP 라인을 통해 실시간으로 데이터를 주고 받을 수 있도록 하는 HTML5의 새로운 사양이다. 따라서 웹 소켓을 이용하면 일반적인 TCP소켓과 같이 연결지향 양방향 전이중 통신이 가능하다`.  

Web Socket과 Ajax(XMLHttpRequest)의 [속도 비교]를 한 곳이 있다.  
여기에 따르면 50배 이상 웹 소켓의 성능이 좋은 것으로 나타난다.  







<br><br><br>
출처 :  
- http://blog.naver.com/PostView.nhn?blogId=beabeak&logNo=220471878778&parentCategoryNo=&categoryNo=82&viewDate=&isShowPopularPosts=true&from=search  
- http://sarc.io/index.php/java/185-html-websocket  
- http://asfirstalways.tistory.com/85
- ***** http://m.mkexdev.net/98  


[속도 비교]:http://bloga.jp/ws/jq/wakachi/mecab/wakachi.html

<br><br><br>


{% highlight ruby %}
{% endhighlight %}
