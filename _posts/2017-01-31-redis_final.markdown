---
layout: post
title:  "redis 정리"
date:   2017-01-31 12:10:53 +0900
categories: redis
---

참조 :  
http://bcho.tistory.com/654  
http://knight76.tistory.com/attachment/cfile4.uf@220DF53954F84EA32C99B0.pdf  
<br>
Spring + redis cache : http://www.baeldung.com/spring-data-redis-tutorial
<br>

## 레디스란

**REmote DIctionary System**  
**Memory 기반의 Key/Value Store**  
<br>
<br>
Cassandra/HBase와 같이 **NoSQL DBMS**로 분류, 또는 memcached같은 **In memory Solution**으로 분류된다.  
<br>
<br>
memcached에 버금가는 성능 + 다양한 데이터 구조체를 지원함으로써 다음과 같은 용도로 사용될 수 있다.
* Message Queue
* Shared Memory
* Remote Dictionary
<br>
<br>

Redis를 적용한 서비스들은 아래와 같다.
* Line
* StackOverflow
* Blizzard
* digg
<br>
<br>

# 데이터 타입
* String
* Set
* Sorted Set
* Hashes
* List

<br>
<br>
데이터 저장 구조  
![저장구조](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/images/Image_%255B7%255D.png?raw=true)  

<br>
<br>
# Persistance
**1. snapshot(RDB) 방식**  
특정 시점에서 메모리에 있는 전체 내용을 DISK에 옮긴다.  
SAVE(blocking 방식)/BGSAVE(non-blocking 방식)이 있다.  
<br>
장점 : 서버 restart시 snapshot만 load하면 되므로 restart가 빠르다.  
단점 : snapshot 추출시간이 오래 걸린다. snapshot 이후 서버가 다운되면 snapshot이후의 데이터는 유실된다.  
<br>
<br>
**2. AOF(Append On File) 방식**  
모든 write/update 연산 자체를 모두 log파일에 기록하는 형태이다.  
서버 restart시 write/update operation을 순차적으로 재 실행해서 데이터를 복구한다.  
non-blocking방식  
<br>
장점 : Log file에 append하므로 server가 다운되도 데이터 유실이 없다.  
단점 : 로그 데이터 양이 RDB방식에 비해 크고, 복구시 write/update operation때문에 restart가 느리다.  
<br>
<br>
**권장 사항**  
<span style="color:blue">RDB와 AOF 방식을 혼용해서 쓰는 것이 바람직하다.</span>  
주기적으로 snapshot으로 백업하고, 다음 snapshot까지 AOF방식으로 log를 쌓는다.  
서버가 restart시 snapshot을 읽어들이고, 소량의 log에서 operation하므로 restart 시간이 절약되고 데이터 유실 또한 막을 수 있다.  
<br>
<br>

<br>
# Replication Topology
**Master/Slave**  
1개의 master node는 n개의 slave node를 가질 수 있고, 각 slave node 또한 n개의 slave node를 가질 수 있다.  
<br>
Query Off Loading 기법을 통해 master node는 write only, slave node는 read only로 사용하면 성능향상에 도움이 된다.  
Redis는 특히 value에 대한 여러가지 연산(ex.합집합, 교집합, Range query등)을 지원하므로 더욱 성능향상에 유리하다.  
<br>
master/slave간의 복제는 non-blocking 상태로 이뤄진다.  
즉, 데이터 불일치성을 유발할 수 있으므로 일치성(Consistency)가 매우 중요하다면 Redis 사용을 다시 한번 고려헤 봐야 한다.  


<br>
<br>
**용량 확장**  
클러스터링을 통한 확장성을 제공하지 않기 때문에 데이터 용량이 클 경우 redis는 Sharding 아키텍쳐를 이용하고 있다.  
ex. Redis 서버별로 저장하는 key 대역폭을 정해놓은 후, 나눠서 저장  

<br>
<br>
**장애 조치(Failover)**  
Master의 데이터가 장애로 인해 손실 될 경우, 묶여 있는 Slave의 데이터도 동시에 손실 될 수 있다.  
- 이런 경우에는 "slaveof no one"이라는 명령어를 사용해 Slave가 Master가 되게할 수 있다.  
- "info" 명령어로 master 설정을 확인해 본다  
- 애플리케이션 서버에서 Redis연결 부분을 장애가 발생한 Master에서 Slave(New Master)로 변경한다.  
- New Master가 된 서버에서 "info" 명령어를 통해 connected_clients가 접속되는지/키 값이 올라가는지 확인한다.  
- "monitor" 명령어를 통해 커맨드가 제대로 들어오는지 확인한다.  

<br>
<br><br>
위에 나열한 장애 조치를 자동으로 시스템에서 할 수 있다.  
- Redis-management 서버를 둬서 Redis 장비 모니터링을 해 status를 확인한다.  
- 각 Redis 장비의 Master/Slave 정보를 Zookeeper에 저장한다.  
- 애플리케이션 서버는 Zookeeper의 정보를 보고 Redis Master의 장비 이름을 확인하고 사용한다.  
- 애플리케이션 서버는 Zookeeper 주소만 알면 된다.  


![구조](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/images/redis_zookeeper_archit.png?raw=true)
<br><br>
다음은 Redis 서버 장애시 새로운 Master로 구조가 변경됨을 보여준다.  

![구조](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/images/redis%20failover%20archit.png?raw=true)
<br><br>
<br>
