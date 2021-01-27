## 마이크로서비스 배포 

마이크로서비스 패턴(길벗) 책의 12장, 마이크로서비스 배포 챕터내용.



---

- 과거에는 WAR/JAR 등 언어에 맞게 패키징해서 배포했었음 

  - 배포가 빠름  

  - 인스턴스가 소비하는 리소스를 제한할 방법이 없음  
  - 여러 서비스 인스턴스가 동일 머신에서 실행되면 서로 격리할 수 없음  
  - 자동으로 인스턴스를 두기가 어려움  
  - 기술 스택을 캡슐화 할 수 없음  



---

- AWS EC2 + AMI(Amazon Machine Image)로 묶어 배포하기
  - 가상 머신 패턴이라고 보면 됨  
  - VM이미지로 기술 스택을 캡슐화 가능  

  - 서비스 인스턴스가 격리된다
  - 리소스는 효율적으로 활용 못함
  - 배포가 비교적 느림
  - 시스템 관리 오버헤드



---

- AWS Lambda등 서버리스 배포 방법도 있음. 

---

- 컨테이너 패턴
  - 더 가벼운 배포 수단

  - 다른 컨테이너들과 격리된 샌드박스에서 하나의 프로세스로 실행됨

  - 고유한 IP 주소를 갖고 있으므로 포트 충돌 가능성 없음
  - 컨테이너마다 자체 루트 파일 시스템 갖고 있음

  - 컨테이너 런타임은 `docker`가 유명

  - 기술 스택의 캡슐화
  - 서비스 인스턴스가 격리되고 리소스를 제할할 수 있음



- `run` 명령어는 단일 머신에서 실행되는 컨테이너를 생성하므로 배포용은 아님

- `docker compose` 역시 개발/테스트할 때 아우 유용하지만 프로덕션용이라고 보긴 어려움

- 서비스를 확실하게 배포하려면 `쿠버네티스`같은 `도커 오케스트레이션 프레임워크`가 필요



---

#### 쿠버네티스

- 리스소 관리
- 스케줄링
- 서비스 관리





- master
- node
- pod
- container





- master
  - api server
  - etcd
  - scheduler
  - controller manager



- node
  - kubelet	
  - kube-proxy
  - pod





- 쿠버네티스 핵심 개념
  - pod
  - deployment
  - service
  - configMap



---

`쿠버네티스`에 `서비스`를 배포하려면  먼저 `deployment`를 정의해야 한다  

`ftgo-restaurant-service.yml`의 쿠버네티스 `deployment` 정의 부분  

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: ftgo-restaurant-service
  labels:
    application: ftgo
    svc: ftgo-restaurant-service
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        svc: ftgo-restaurant-service
        application: ftgo
    spec:
      containers:
      - name: ftgo-restaurant-service
        image: msapatterns/ftgo-restaurant-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: httpport
        env:
          - name: JAVA_OPTS
            value: "-Dsun.net.inetaddr.ttl=30"
          - name: SPRING_DATASOURCE_URL
            value: jdbc:mysql://ftgo-mysql/eventuate
          - name: SPRING_DATASOURCE_USERNAME
            valueFrom:
              secretKeyRef:
                name: ftgo-db-secret
                key: username
          - name: SPRING_DATASOURCE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ftgo-db-secret
                key: password
          - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
            value: com.mysql.jdbc.Driver
          - name: EVENTUATELOCAL_KAFKA_BOOTSTRAP_SERVERS
            value: ftgo-kafka:9092
          - name: EVENTUATELOCAL_ZOOKEEPER_CONNECTION_STRING
            value: ftgo-zookeeper:2181
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 20
```

핼스체크 end point를 호출해서 인스턴스 상태를 파악함   



---

yml이 준비되었으면 kubectl apply 커맨드로 deployment를 생성/수정하면 된다.    

`kubectl apply -f ftgo-restaurant-service/src/deployment/kubernetes/ftgo-restaurant-service.yml `   

위 deployment를 사용하려면 쿠버네티스 시크릿을 생성해야 하는데 자세한 내용은 공식 문서를 볼 것    



---

`pod` ip 주소는 동적할당 되므로 서비스 `디스커버리` 메커니즘이 필요함

쿠버네티스 기본 내장 되어 있음  

IP 주소 및 DNS명이 할당된 서비스는 하나 이상의 pod 클라이언트에 안정된 end point를 제공하는 쿠버네티스 오브젝트이다. ??  

서비스는 자신의 IP주소로 향하는 트래픽 부하를 여러 pod에 고루 분산한다.   

ftgo-restaurant-service.yml의 쿠버네티스 Service 정의 부분  

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ftgo-restaurant-service
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    svc: ftgo-restaurant-service
```



---

음식점 서비스는 클러스터 내부에서만 접근가능  

API GW는 외부의 트래픽을 라우팅하므로 정의해줘야 한다.   

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ftgo-api-gateway
spec:
  type: NodePort
  ports:
  - nodePort: 30000
    port: 80
    targetPort: 8080
  selector:
    svc: ftgo-api-gateway
```

`NodePort` 로 지정해주면 모든 노드에 접근할 수 있다.   

광역 클러스터 포트여서 30000~32767 중에 택일해야 한다   





부하 분산은 AWS ELB를 구성하면 된다   

type에서 LoadBalancer는 부하 분산기를 자동 구성하는 서비스이다   

AWS에서 쿠버네티스를 실행하면 부하 분산기가 ELB가 되는 것  





---

#### 무중단 배포?  



변경분을 프로덕션에 배포한다면   

- 이미지 빌드

- 레지스트리에 푸쉬/태그 변경

- 새 이미지를 참조하도록 `deployment yaml `변경

- `kubectl apply -f` 로 `deployment`를 업데이트





쿠버네티스는 pod롤 `롤링 업데이트` 한다    

(신 버전이 요청 처리가 완료되지 전에는 구버전을 종료 안함)  

`deployment`는 `rollout`이라는 이력을 관리함  

아래 커멘드로 롤백할 수 있음   

`kebectl rollout undo deployment ftgo-restaurant-service`   



---

하지만 신버전이 배포가 된 이후 버그를 발견하면? 영향을 받아버린다.   

`배포`와 `릴리즈`를 분리하는게 좋다 -> `서비스 메시`가 이런일을 해준다  





- 새 버전의 서비스를 프로덕션에 배포함(라우팅은 안함)  
- 프로덕션에서 새 버전을 테스트함
- 소수의 트래픽을 새 버전에 라우팅 함
- 점점 늘려감
- 어딘가 문제가 생기면 구 버전으로 돌림



서비스메시는 아래 기능을 가짐  

- 마이크로서비스 섀시 프레임워크  기능(회로차단, 분산 추적, 디스커버리 등) 

- 규칙 기반의 부하 분산 및 트래픽 라우팅 기능 제공



---

[`Istios`](https://istio.io/latest/docs/setup/getting-started/)는 `서비스 메시` 제품중 하나임  

마이크로서비스를 연결, 관리, 보안하는 오픈 플랫폼이라고 소개함  



- 트래픽 관리()

- 보안(TLS)

- 텔레메트리(지표수집, 분산추적)

- 정책 집행(쿼터 및 사용률 제한)





