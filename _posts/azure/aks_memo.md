http://wiki.skplanet.com/pages/viewpage.action?pageId=232602408



#### **AKS**

```
az login
az account set --subscription 0b15d13d-80a3-4a82-aebd-3b9dd05d18b3
az aks get-credentials --resource-group skp-muffin-dev3 --name skp-muffin-aks
# 모든 네임스페이스의 모든 배포 나열
kubectl get deployments --all-namespaces=``true
```



#### **ACR**

```
docker login acr0muffin.azurecr.io
az acr repository list -n acr0muffin
id : acr0muffin
password : 2Oo2o1FipgjhFwA``//wZg8MLdAHxvfFOO
az acr repository list -n acr0muffin
```



###### 노드 풀의 상태를 확인

```
az aks nodepool list --resource-group skp-muffin-dev3 --cluster-name skp-muffin-aks
```



###### 배포

```

./gradlew clean build -x test
az acr build --registry acr0muffin --image common-api:20210412 -f Dockerfile.skp .
kubectl apply -f manifests/deployment_skp.yml
kubectl apply -f manifests/service_skp.yml
<pod log 추출>
kubectl get pods
kubectl logs pod_name
<pod shell 에 접속>
kubectl exec --stdin --tty pod_name -- /bin/bash
```

http://52.231.73.15:8080/truckuser/swagger-ui.html





###### deployment.yml

```yacas
apiVersion : apps/v1
kind: Deployment
metadata:
  name: truckuser-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: truckuser-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: truckuser-api
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
        runAsNonRoot: true
      containers:
        - name: truckuser-api
          imagePullPolicy: Always
          image: acr0muffin.azurecr.io/truckuser-api:20210412
          securityContext:
            allowPrivilegeEscalation: false
          resources:
            limits:
              cpu: "1"
              memory: "2Gi"
            requests:
              cpu: "1"
              memory: "2Gi"
```



###### service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ske-cvp-truckuser-api
spec:
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  selector:
    app: truckuser-api
```

