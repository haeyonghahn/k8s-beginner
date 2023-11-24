## 1. Container
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/8ad63412-906c-4bec-9d0f-6ea2374743c9)

### 1-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container1
    image: kubetm/p8000
    ports:
    - containerPort: 8000
  - name: container2
    image: kubetm/p8080
    ports:
    - containerPort: 8080
```

## 2. Label
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/b971b299-b79f-4f2d-b7f5-9530ca42da1b)

### 2-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    type: web
    lo: dev
spec:
  containers:
  - name: container
    image: kubetm/init
```
### 2-2. Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: web
  ports:
  - port: 8080
```
## 3. Node Schedule
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/afaa8913-f7f3-41c1-bc8d-355e8dabaff8)

### 3-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
```
### 3-2) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: kubetm/init
    resources:
      requests:
        memory: 2Gi
      limits:
        memory: 3Gi
```
