---
layout: post
title:  "ZooKeeper"
date:   2017-02-15 13:10:53 +0900
categories: zookeeper
---

참조 :  
http://bcho.tistory.com/1016  
http://d2.naver.com/helloworld/294797  

<br><br>

분산 시스템 설계시,  
시스템간의 정보를 어떻게 `공유`할 것인가?  
서버들의 `상태 체크`를 어떻게 할 것인가?  
동기화를 위한 `Lock`을 어떻게 처리할 것인가?  
등을 고려해야 한다.  

이러한 문제를 해결하는 시스템을 `Coordination service`라고 하며, `Apache Zookeeper`가 대표적인 서비스이다.  
https://zookeeper.apache.org  
<br><br>

분산 시스템의 중요한 상태/설정 정보 등을 유지하기 때문에, 이 코디네이션 서비스에 장애가 발생하면 전체 시스템의 장애로 유발된다.  
따라서 이중화 등을 통해 고가용성을 제공해야 하는데, ZooKeeper가 잘 제공하고 있다.  
* 자체적으로 `클러스터링`을 제공한다.  
* 장애에도 데이터 유실 없이 `fail over/fail back` 등이 가능하다.


<br><br>
<br><br>
<br><br>
<br><br>
<br><br>
`대용량`의 데이터를 다루다도면 `MySQL 샤딩만으로는 버겁다.`  
Database를 증설할 경우, 다수의 MySQL 연관 장비를 투입/ 애플리ㄱ케이션 서버의 샤딩 로직을 수정해 서버를 다시 배포해야 한다.  
대용량의 데이터를 저장하기보다 높은 트래픽을 처리하는 것이 중요하다면 Memory DB인 `Redis`가 좋은 대체 Database가 된다.  
그리고 `클러스터를 구성하기 위해(샤딩, Failover 등) ZooKeeper를 사용`한다.  
Zookeeper는 Redis뿐만 아니라 분산 서버 환경에서 효율이 좋은 도구이다.  
<br><br>
ZooKeeper를 한마디로 정의하면?  
`"분산 처리 환경에서 사용가능한 데이터 저장소"` 라고 할 수 있다.  
`분산 서버 간의 정보 공유, 서버 투입/제거 시 이벤트 처리, 서버 모니터링, 시스템 관리, 분산 락 처리, 장애 상황 판단 등` 다양한 분야에서 활용할 수 있다.  
<br><br>
ZooKeeper는 데이터를 디렉터리 구조로 저장한다.  
데이터가 변경되면 클라이언트에게 어떤 노드가 변경됐는지 콜백해준다.  
<br><br>
**사용/운영 시 몇가지 주의 사항**  
캐시 용도로 사용했을 때, 장애가 발생한 사례가 종종 있다.  
데이터 변경이 자주 발생하는 서비스에서 ZooKeeper를 데이터 저장소로 사용하는 것은 비추이다.  
Read:Write 비율은 10:1 이상이 좋다.  
<br><br>
ZooKeeper 서버가 제대로 실행되지 않을 때는, 대부분 서버간의 데이터 불일치로 인한 데이터 동기화 실패가 원인이다.  
이때는 데이터를 초기화한 후 서버를 실행해야 한다.  
<br><br>
zoo.cfg 설정파일의 ZooKeeper 서버 목록을 꼼꼼히 확인해야 한다.  
<br><br><br><br>
[아키텍처 구성도]  
![아키텍처 구성도](http://d2.naver.com/content/images/2015/06/helloworld-294797-4.png)  
