---
layout: post
title:  "sslv3-poodle-tls"
date:   2017-05-19 10:10:53 +0900
categories: secure
---
<br><br><br>
## SSLv3 poodle 취약점에 따른 변경

{% highlight ruby %}
# apache, httpd.conf  
# httpd-ssl.conf  
# 방법1.
SSLProtocol All -SSLv2 -SSLv3 // SSLv2 와 SSLv3 을 비활성화 (SSLv2와 SSLv3 이외는 모두 사용)**


# 방법2.
SSLProtocol -All +TLSv1 // TLSv1.x 제외하고 모두 사용 안 함


service httpd restart // httpd 를 재시작


# tomcat, server.xml
sslProtocol 속성 변경할 것

{% endhighlight %}

<br><br><br>

## 차이?  
SSL과 TLS는 무엇이 다른가?  
**TLS 1.0은 SSL 3.0을 계승** 한다.  

**SSLv2, v3는 POODLE,DROWN 등의 취약점이 발견되어 암호화 프로토콜로의 기능을 잃은 상태** 이다.  
( Internet Explorer 6이 SSL 3.0 프로토콜까지만 지원한다. )  

**TLS1.0** : SSL 3.0과 큰 차이가 있는 것은 아니나, SSL 3.0이 가지고 있는 대부분의 취약점이 해결되었다.(Internet Explorer 8이 지원하는 가장 높은 TLS 버전인 관계로 **아직도 널리 사용되고 있다.**)  

TLS1.1 : 암호 블록 체인 공격에 대한 방어와 IANA 등록 파라메터의 지원이 추가되었다.  

TLS1.2 : 취약한 SHA1 해싱 알고리즘이 SHA256으로 교체되었다. 나무위키와 네이버 등 포털 사이트 등에서 지원하고 있다.  

<br>

HTTPS : TLS를 사용해 암호화된 연결을 하는 HTTP를 HTTPS라고 하며, 당연히 웹사이트 주소 역시 HTTP가 아닌 HTTPS로 시작된다. 기본 포트는 443번을 쓴다.  
(TLS와 HTTPS를 혼동하는 경우가 많은데, 둘은 유사하긴 하지만 엄연히 다른 개념임)  
(TLS는 다양한 종류의 보안 통신을 하기 위한 프로토콜 - HTTP뿐만이 아니라 FTP, SMTP와 같은 여타 프로토콜까지 포함)  
(HTTPS는 TLS 위에 HTTP 프로토콜을 얹어 보안된 HTTP 통신을 하는 프로토콜)  



<br><br><br>


참조:  
- https://access.redhat.com/ko/node/1258903
- http://kkamagistory.tistory.com/488
- https://namu.wiki/w/TLS
- https://www.kicassl.com/cstmrsuprt/ntc/searchNtcDetail.sg?page=&ntcSeq=620&mode=&searchType=subject&searchWord=&searchRowCnt=10
<br><br><br>

{% highlight ruby %}
{% endhighlight %}
