## 1. ClusterIP
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/becaaca2-ee2f-460e-a64f-1a6ad5836083)

### 1-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```
### 1-2) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```
```
curl 10.104.103.107:9000/hostname
```

## 2. NodePort
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/4482b0f1-437f-4f32-95d8-6d0e8ee0d388)

### 2-1) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
  externalTrafficPolicy: Local
```

## 3. Load Balancer
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/714528c9-3f6c-44ae-abf2-dadbfee4e7c9)

### 3-1) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
  type: LoadBalancer
```
```
kubectl get service svc-3
```

## Sample Yaml
### Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:             # Pod의 Label과 매칭
    app: pod
  ports:
  - port: 9000          # Service 자체 Port
    targetPort: 8080    # Pod의 Container Port
  type: ClusterIP, NodePort, LoadBalancer  # 생략시 ClusterIP
  externalTrafficPolicy: Local, Cluster    # 트래픽 분배 역할
```

## kubectl
### Get
defalut 이름의 Namespace에서 svc-3 이름의 Service 조회
```
kubectl get service svc-3 -n defalut
```

## Tips
### NodePort
- Node Port의 범위 : 30000~32767
