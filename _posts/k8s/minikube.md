## Hello minikube



---

[minikube doc](https://minikube.sigs.k8s.io/docs/start/)

macOS

- 설치: `brew install minikube`

- 시작: `minikube start`



---

### [hello minikube](https://kubernetes.io/ko/docs/tutorials/hello-minikube/)

- 대쉬보드 활성화 해보기

  `minikube dashboard`

![k8s dashboard](https://github.com/sungyoungKwon85/sungyoungKwon85.github.io/blob/master/_posts/k8s/images/dashboard%20example.png)



​																																				

---

#### Deployment 만들기

- pod를 관리할 deployment를 만든다. 제공된 docker image를 기반으로 컨테이너를 실행한다. 

  `$ kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.41`  
  `deployment.apps/hello-node created`

- deployment 보기  
  `kubectl get deployments`

  ```
  NAME         READY   UP-TO-DATE   AVAILABLE   AGE
  hello-node   1/1     1            1           31s
  ```

- pod 보기  
  `kubectl get pods`

  ```
  NAME                          READY   STATUS    RESTARTS   AGE
  hello-node-7567d9fdc9-jjbxr   1/1     Running   0          72s
  ```

- 클러스터 이벤트 보기  
  `kubectl get events `   

  ```
  LAST SEEN   TYPE     REASON              OBJECT                             MESSAGE
  101s        Normal   Scheduled           pod/hello-node-7567d9fdc9-jjbxr    Successfully assigned default/hello-node-7567d9fdc9-jjbxr to minikube
  99s         Normal   Pulling             pod/hello-node-7567d9fdc9-jjbxr    Pulling image "k8s.gcr.io/echoserver:1.4"
  88s         Normal   Pulled              pod/hello-node-7567d9fdc9-jjbxr    Successfully pulled image "k8s.gcr.io/echoserver:1.4" in 11.3346621s
  88s         Normal   Created             pod/hello-node-7567d9fdc9-jjbxr    Created container echoserver
  88s         Normal   Started             pod/hello-node-7567d9fdc9-jjbxr    Started container echoserver
  101s        Normal   SuccessfulCreate    replicaset/hello-node-7567d9fdc9   Created pod: hello-node-7567d9fdc9-jjbxr
  101s        Normal   ScalingReplicaSet   deployment/hello-node              Scaled up replica set hello-node-7567d9fdc9 to 1
  ```

- kubectl 환경설정 보기  
  `kubectl config view`

  ```
  apiVersion: v1
  clusters:
  - cluster:
      certificate-authority: /Users/sss/.minikube/ca.crt
      extensions:
      - extension:
          last-update: Wed, 27 Jan 2021 12:24:45 KST
          provider: minikube.sigs.k8s.io
          version: v1.17.0
        name: cluster_info
      server: https://127.0.0.1:55000
    name: minikube
  contexts:
  - context:
      cluster: minikube
      extensions:
      - extension:
          last-update: Wed, 27 Jan 2021 12:24:45 KST
          provider: minikube.sigs.k8s.io
          version: v1.17.0
        name: context_info
      namespace: default
      user: minikube
    name: minikube
  current-context: minikube
  kind: Config
  preferences: {}
  users:
  - name: minikube
    user:
      client-certificate: /Users/sss/.minikube/profiles/minikube/client.crt
      client-key: /Users/sss/.minikube/profiles/minikube/client.key
     
  ```

### 

---

#### 서비스 만들기

기본적으로 파드는 쿠버네티스 클러스터 내부의 IP 주소로만 접근할 수 있다. `hello-node` 컨테이너를 쿠버네티스 가상 네트워크 외부에서 접근하려면 파드를 쿠버네티스 [*서비스*](https://kubernetes.io/ko/docs/concepts/services-networking/service/)로 노출해야 한다.

- `kubectl expose` 명령어로 퍼블릭 인터넷에 파드 노출하기  

  `kubectl expose deployment hello-node --type=LoadBalancer --port=8080`

- 서비스 살펴보기

  ```
  kubectl get services
  NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
  hello-node   LoadBalancer   10.106.217.64   <pending>     8080:30497/TCP   24s
  kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          155m
  ```

- ```
  minikube service hello-node
  |-----------|------------|-------------|---------------------------|
  | NAMESPACE |    NAME    | TARGET PORT |            URL            |
  |-----------|------------|-------------|---------------------------|
  | default   | hello-node |        8080 | http://192.168.49.2:30497 |
  |-----------|------------|-------------|---------------------------|
  🏃  hello-node 서비스의 터널을 시작하는 중
  |-----------|------------|-------------|------------------------|
  | NAMESPACE |    NAME    | TARGET PORT |          URL           |
  |-----------|------------|-------------|------------------------|
  | default   | hello-node |             | http://127.0.0.1:50392 |
  |-----------|------------|-------------|------------------------|
  🎉  Opening service default/hello-node in default browser...
  ❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
  ```

  

---

#### Addons 사용하기

- 현재 지원하는 애드온 목록을 확인

  ```
  minikube addons list
  |-----------------------------|----------|--------------|
  |         ADDON NAME          | PROFILE  |    STATUS    |
  |-----------------------------|----------|--------------|
  | ambassador                  | minikube | disabled     |
  | csi-hostpath-driver         | minikube | disabled     |
  | dashboard                   | minikube | enabled ✅   |
  | default-storageclass        | minikube | enabled ✅   |
  | efk                         | minikube | disabled     |
  | freshpod                    | minikube | disabled     |
  | gcp-auth                    | minikube | disabled     |
  | gvisor                      | minikube | disabled     |
  | helm-tiller                 | minikube | disabled     |
  | ingress                     | minikube | disabled     |
  | ingress-dns                 | minikube | disabled     |
  | istio                       | minikube | disabled     |
  | istio-provisioner           | minikube | disabled     |
  | kubevirt                    | minikube | disabled     |
  | logviewer                   | minikube | disabled     |
  | metallb                     | minikube | disabled     |
  | metrics-server              | minikube | disabled     |
  | nvidia-driver-installer     | minikube | disabled     |
  | nvidia-gpu-device-plugin    | minikube | disabled     |
  | olm                         | minikube | disabled     |
  | pod-security-policy         | minikube | disabled     |
  | registry                    | minikube | disabled     |
  | registry-aliases            | minikube | disabled     |
  | registry-creds              | minikube | disabled     |
  | storage-provisioner         | minikube | enabled ✅   |
  | storage-provisioner-gluster | minikube | disabled     |
  | volumesnapshots             | minikube | disabled     |
  |-----------------------------|----------|--------------|
  ```

- 한 애드온을 활성화 한다. 예를 들어 `metrics-server`

  ```
   minikube addons enable metrics-server
  🌟  'metrics-server' 애드온이 활성화되었습니다
  ```

- 방금 생성한 파드와 서비스를 확인한다.

  ```
  kubectl get pod,svc -n kube-system
  NAME                                   READY   STATUS    RESTARTS   AGE
  pod/coredns-74ff55c5b-czc9h            1/1     Running   0          159m
  pod/etcd-minikube                      1/1     Running   0          159m
  pod/kube-apiserver-minikube            1/1     Running   0          159m
  pod/kube-controller-manager-minikube   1/1     Running   0          159m
  pod/kube-proxy-g2cgl                   1/1     Running   0          159m
  pod/kube-scheduler-minikube            1/1     Running   0          159m
  pod/metrics-server-56c4f8c9d6-ctsm4    1/1     Running   0          21s
  pod/storage-provisioner                1/1     Running   1          159m
  
  NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
  service/kube-dns         ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   159m
  service/metrics-server   ClusterIP   10.103.179.38   <none>        443/TCP                  21s
  ```

- `metrics-server` 비활성화
  `minikube addons disable metrics-server`



---

#### 제거하기

- 리소스 제거하기

  ```
  kubectl delete service hello-node
  kubectl delete deployment hello-node
  service "hello-node" deleted
  deployment.apps "hello-node" deleted
  ```

- 필요하면 Minikube 가상 머신(VM)을 정지
  `minikube stop`

- 필요하면 minikube VM을 삭제
  `minikube delete`





















