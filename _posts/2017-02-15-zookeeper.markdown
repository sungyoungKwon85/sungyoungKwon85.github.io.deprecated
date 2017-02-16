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
`대용량`의 데이터를 다루다도면 `MySQL 샤딩만으로는 버겁다.`  
Database를 증설할 경우, 다수의 MySQL 연관 장비를 투입/ 애플리케이션 서버의 샤딩 로직을 수정해 서버를 다시 배포해야 한다.  
대용량의 데이터를 저장하기보다 높은 트래픽을 처리하는 것이 중요하다면 Memory DB인 `Redis`가 좋은 대체 Database가 된다.  

또한 분산 시스템이 필요하며 이 분산 시스템 설계시,  
시스템간의 정보를 어떻게 `공유`할 것인가?  
서버들의 `상태 체크`를 어떻게 할 것인가?  
동기화를 위한 `Lock`을 어떻게 처리할 것인가?  
등을 고려해야 한다.  

이러한 문제를 해결하는 시스템을 `Coordination service`라고 하며, `Apache Zookeeper`가 대표적인 서비스이다.  
https://zookeeper.apache.org  

<br>

ZooKeeper를 한마디로 정의하면?  
`"분산 처리 환경에서 사용가능한 데이터 저장소"` 라고 할 수 있다.  
`분산 서버 간의 정보 공유, 서버 투입/제거 시 이벤트 처리, 서버 모니터링, 시스템 관리, 분산 락 처리, 장애 상황 판단 등` 다양한 분야에서 활용할 수 있다.  
분산 시스템의 중요한 상태/설정 정보 등을 유지하기 때문에, 이 코디네이션 서비스에 장애가 발생하면 전체 시스템의 장애로 유발된다.  
따라서 이중화 등을 통해 고가용성을 제공해야 하는데, ZooKeeper가 잘 제공하고 있다.  

<br><br>
<br><br>


ZooKeeper는 데이터를 디렉터리 구조로 저장한다.  
Persistance를 유지하기 위해 transaction log와 snapshot file이 디스크에 저장된다.  
각 디렉터리 노드를 znode라고 한다.  
- Persistent node : 명시적으로 삭제하지 않는 한 유지됨  
- Ephemeral node : 세션이 유효한 동안 유지됨  
- Sequence node : sequence number가 붙어 분산 락 등을 구현할 때 유용  

<br>

zookeeper서버는 일반적으로 3대 이상, 홀수로 구성된다. 서버 간 데이터 불일치가 발생했을 때 과반수의 룰을 적용하기 때문이다.  
Leader와 Follwer로 구성되고, 자동으로 Leader를 선정한다.  

zookeeper 서버 구성도는 아래와 같다.  
클라이언트에서 하나의 Server로 데이터 저장을 시도하면 Server --> Reader --> Other Followers 의 순서로 데이터가 전달되며, 모든 서버에 전달 된 후에 클라이언트에 성공/실패를 알려주는 동기방식으로 동작한다.  
<br>

![구조](http://d2.naver.com/content/images/2015/06/helloworld-294797-2.png)  

<br><br><br>

Zookeeper와 Redis를 이용해 Redis Cluster를 구성한 아키텍쳐는 아래와 같다.  
Redis Cluster Manager를 구현해야 하고, 또한 Zookeeper 디렉터리 구조를 설계해야 한다.  
(Redis Cluster Manager의 요구사항 및 구현은 Naver D2 블로그 참고)  

<br>

[아키텍처 구성도]  

![아키텍처 구성도](http://d2.naver.com/content/images/2015/06/helloworld-294797-4.png)  

<br><br>
