---
layout: post
title:  "Cache in Java"
date:   2017-02-06 15:10:53 +0900
categories: Cache
---

참조 :  
http://start.goodtime.co.kr/2015/06/자바에서-데이터-캐시-구현하기/  
http://m.blog.daum.net/iamuzooin/102  
<br><br>


# 캐시의 개념  
RDBMS를 백엔드에 두고 데이터 트랜잭션 위주로 운영하는 웹애플리케이션의 경우, 성능을 위해서 애플리케이션 서버와 데이터베이스 서버를 이중화 하곤 한다.  
사용자가 많이 몰리면 결국 병목(bottleneck)이 되는 곳은 데이터베이스인 경우가 많다.  

가장 근본적인 방법은 질의를 줄이는 것이다.  
애플리케이션 입장에서의 캐시는 데이터베이스의 데이터를 정적인 메모리에 보관해뒀다가 필요할 때 메모리에서 바로 쓰는 것이다.  
데이터를 바로 찾을 수 있도록 key를 부여하면 되는데 대표적으로 hash를 사용하면 구현이 가능하다.  

캐시에 사용할 데이터는 자주 사용하면서도 변경은 거의 일어나지 않으며 크기가 작은 것을 택해야 한다.  
예) 카테고리 정보, 게시판 설정 정보, 날씨 정보 등

<br><br>
<br><br>

Spring에서는 표준 Observer패턴으로 이벤트를 전파하는 기능이 있다.  
ApplicationContext, ApplicationEvent 클래스 ApplicationListener 인터페이스를 통해 제공된다.  
ApplicationContext는 Spring에서 IoC 컨테이너 또는 Spring 컨테이너로 볼 수 있다.  
ApplicationContext는 **ApplicationListener** 인터페이스를 구현한 JavaBenas(Observer)가 컨테이너에 등록되어 있을 때, 이벤트 발생 때마다 Observer 이벤트를 통지한다.  
ApplicationListener 인터페이스 구현메서드는 **onApplicationEvent**이고, 이벤트(컨테이너 구동) 발생시마다 이 메서드가 호출된다.  
`바로 여기가 구현한 캐시 init을 호출해주기 좋은 장소`이다.  
컨테이너 구동시마다 캐시에 사용할 데이터를 캐시에 넣어둘 수 있다.  
