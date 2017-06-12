---
layout: post
title:  "container-orchestration"
date:   2017-05-30 12:10:53 +0900
categories: pinpoint
---

<br><br><br>

아래 참조의 링크들의 내용을 정리했습니다.  
- https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html
- https://www.slideshare.net/seungyongoh3/ndc17-kubernetes  

<br><br><br>

## Container Orchestration  


<br><br><br>

큰 회사와는 달리 작은 회사들은 많은 서버개발자가 DevOps Engineer의 역할을 할 필요가 많습니다.  
아키텍쳐 설계부터 스택 선택, 개발부터 프로덕션까지 서버 배포, 모니터링, 장애 대응, 새로운 시스템의 개발을 진행하곤 합니다.  

<br><br>

여태 경험해왔던 서버 구조는 대부분 아래와 비슷한 프로세스로 진행해 왔습니다.  
- 웹서버 > 웹 애플리케이션 서버 > DB  
- DEV > QA > STG > PRD

<br><br>

보통 개발 서버는 하나만 두고 그곳에 웹 애플리케이션을 배포해서 개발/테스트를 하곤 합니다.  
API서버, 어드민서버, 배치 서버 등 용도에 맞춰서 개발 서버는 여러대가 있을 수 있습니다.  
이런 방식은 어느정도 효율적이고 정리가 잘 된 것처럼 보입니다.  

하지만 용도에 맞춰 나눠놓았더라고 특정 서버에만 컨테이너가 몰리는 상황이 점차 발생하게 되고(API서버나 WEB 서버일듯?) 역할이 모호한 애플리케이션의 경우 어디에 설치할지가 모호해집니다.  
컨테이너가 몰린 서버에서 여유가 있는 서버로 옮기려고 해도 의존성 문제때문에 쉽사리 옮길수가 없게 됩니다.  

`Docker가 생겨서 애플리케이션 설치가 매우 편리해졌습니만 더 나아가 컨테이너를 실행할 서버를 고르는 것부터 관리하는 것 까지 자동화 하는 것이 가능해졌습니다.`     
이러한 작업을 `Orchestration`이라고 합니다.  

<br><br><br>


서버 Orchestration을 간단하게 정의하면 `여러 대의 서버와 여러 개의 서비스를 편리하게 관리`해주는 작업입니다.  
간단하게는 아래와 같은 역할들을 합니다.  
- 스케쥴링 : 컨테이너를 적당한 서버에 배포
- 클러스터링 : 여러 개의 서버를 하나의 서버처럼 사용
- 서비스 디스커버리 : 서비스를 찾아주는 기능
- 로킹, 모니터링 : 로그와 서버 상태를 한곳에서 관리

<br><br><br>

### Container Orchestration Tool  
다양한 툴들이 있지만 대표적인 것 몇개만 나열합니다.  
- Docker swarm // 적당한 규모에 좋음
- Kebernetes // 대규모 클러스트에 좋음
- ECS // AWS에서 단순하게..
- Nomad
- Cloud Foundry

<br><br><br>

아주 간략하게 Container Orchestration을 알아보았습니다.  
다음에는 Docker swarm, Kebernetes, Could Foundry 등을 스터디해서 정리할 예정 입니다.  




<br><br><br>



{% highlight ruby %}
{% endhighlight %}
