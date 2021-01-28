# [Azure AKS 활용법 101 정리](https://www.youtube.com/watch?v=fCVPp-g9Ti4&list=PLDZRZwFT9WkuMBsjmzk5E7GOG8DIh49Ur&index=2&ab_channel=MicrosoftDeveloperKorea)

Azure AKS 활용법에 대한 유튜브 영상을 정리합니다. 



---

### ep2. Kubernetes

- Kubernetes 사용자: API서버와 통신을 하여 state를 가짐
- Master node: worker node들이 해당 state를 가지도록 설정/보장  
- Worker node: 컨테이너를 직접적으로 관리  

 ![k8s1](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/_posts/azure/images/k8s1.png)



#

---

### ep3. Kubernetes Service

Container만으로 부족해서 `Kubernetes`를 쓰는데, 그것만으로도 **충분하지 않다?**  

- k8s를 위한 인프라 자동화: 프로비저닝, 패치, 업그레이드 단순화
- 컨테이너화된 앱 개발, 그리고 `CI/CD` 워크 플로우에 적합한 도구 및 툴
- `보안`, `거버넌스`, `인증`, `액세스 관리`를 지원하는 서비스

그래서 나온게 `Azure K8s Service (AKS)`  

`k8s` 사용/운영을 Microsoft에서 관리해주는 것.   

![AKS](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/_posts/azure/images/k8s2.png)



#

---

### ep4. Azure Container Registry 만들기

공식 [문서](https://docs.microsoft.com/ko-kr/azure/aks/tutorial-kubernetes-prepare-app)를 따라해보면 된다.   

- git 에서 소스를 가져온다

  - `$ git clone https://github.com/Azure-Samples/azure-voting-app-redis.git`

- 해당 샘플은 이미지를 두개 사용한다 

- `docker-compose.yaml` 

  - ```dockerfile
    version: '3'
    services:
      azure-vote-back:
        image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
        container_name: azure-vote-back
        environment:
          ALLOW_EMPTY_PASSWORD: "yes"
        ports:
            - "6379:6379"
    
      azure-vote-front:
        build: ./azure-vote
        image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
        container_name: azure-vote-front
        environment:
          REDIS: azure-vote-back
        ports:
            - "8080:80"	
    ```

  - `$ docker-compose up -d` // 이미지도 받아지고 실행도 된다

- Azure ->` 컨테이너 레지스트리 만들기` 를 통해 레지스트리 생성

- 컨테이너 레지스트리에 로그인하기

  - `$ docker login demoazurereg.azurecr.io`
  - 생성된 레지스트리 화면에서 id, pw를 볼 수 있음

- 샘플 이미지를 생성한 레지스트리에 푸시하기

  - `$ docker tag azure-vote-front demoazurereg.azurecr.io/azure-vote-front`
  - `$ docker push demoazurereg.azurecr.io/azure-vote-front`
  - `$ docker tag redis demoazurereg.azurecr.io/redis`
  - `$ docker push demoazurereg.azurecr.io/redis`

-  생성한 레지스트리에 올라갔는지 확인

- `azure-vote-all-in-one-redis.yaml` ??



---

### ep5. Azure k8s Cluster 만들기

- 포털 > k8s 클러스터 만들기 클릭
- 주 노드 풀 3
- VM 확장 집합 기능 사용(자동으로 scale in, out 함)
- 인증 > 클러스터 인프라 > 시스템에서 할당한 관리 ID
- 네트워킹 > 네트워크 구성 > 고급..
- 통합 > Azure Container Registry > 생성한거 선택하기
- ...



---

### ep6. Application 배포하기

- 포털 > Kubernetes 서비스 클릭

- 생성했던 클러스터 정보가 나온다

- '연결' 탭 클릭

- Cloud Shell 열기 또는 Azure CLI 부분에서 두번째를 복사

  - Kubernetes 클러스터에 연결하도록 `kubectl`을 구성하려면

  - `az aks get-credentials --resource-group demo-rg --name demo-demo-cluster`

  - ```
    $ kubectl get nodes
    
    NAME                       STATUS   ROLES   AGE   VERSION
    aks-nodepool1-12345678-0   Ready    agent   32m   v1.14.8
    ```

- 내가 갖고 있는 pod를 배포할 차례

  - yaml 파일을 통해서 배포가 된다 

  - `azure-vote-all-in-one-redis.yaml` 

  - ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-back
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-back
      template:
        metadata:
          labels:
            app: azure-vote-back
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-back
            image: demoazurereg.azurecr.io/redis:latest // 생성했던 레지스트리의 주소를 써야한다 
            env:
            - name: ALLOW_EMPTY_PASSWORD
              value: "yes"
            ports:
            - containerPort: 6379
              name: redis
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-back
    spec:
      ports:
      - port: 6379
      selector:
        app: azure-vote-back
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-front
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-front
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
      minReadySeconds: 5
      template:
        metadata:
          labels:
            app: azure-vote-front
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-front
            image: demoazurereg.azurecr.io/azure-vote-front:latest // 생성했던 레지스트리의 주소를 써야한다 
            ports:
            - containerPort: 80
            resources:
              requests:
                cpu: 250m
              limits:
                cpu: 500m
            env:
            - name: REDIS
              value: "azure-vote-back"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-front
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: azure-vote-front
    ```

  - 클러스터에 배포하기

    - ```
      $ kubectl apply -f azure-vote-all-in-one-redis.yaml
      
      deployment "azure-vote-back" created
      service "azure-vote-back" created
      deployment "azure-vote-front" created
      service "azure-vote-front" created
      ```

    - `$ kubectl get pods`  // pod 확인

    - `$ kubectl get svc` // 서비스 확인

- 컨테이너 확장을 확인해보자

  - `$ kubectl autoscale deployment --max=10 azure-vote-front --min=3`
  - `$ kubectl get pods`
    - pods가 늘어난걸 볼 수 있다 

- 영상에서 부하 테스트도 함. 



---

### ep7. 배포된 Application 모니터링하기

대시보드 > Kubernetes 서비스 > 생성한 클러스터 > 모니터링















