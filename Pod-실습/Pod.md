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

## Sample Yaml
### Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4                           # Pod 이름
  labels:                               # Label 
    type: web                           
    lo: dev  
spec:
  nodeSelector:                         # Node 직접 지정시
    kubernetes.io/hostname: k8s-node1   
  containers:
  - name: container                     # Container 이름
    image: kubetm/init                  # 이미지 선택
    ports:
    - containerPort: 8080               
    resources:                          # 자원 사용량 설정
      requests:
        memory: 1Gi
      limits:
        memory: 1Gi
```

## kubectl
### Create
```yml
# 파일이 있을 경우
kubectl create -f ./pod.yaml

# 내용과 함께 바로 작성
kubectl create -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END
```
### Apply
```
kubectl apply -f ./pod.yaml
```
### Get
```
# 기본 Pod 리스트 조회 (Namepsace 포함)
kubectl get pods -n defalut

# 좀더 많은 내용 출력
kubectl get pods -o wide

# Pod 이름 지정
kubectl get pod pod1

# Json 형태로 출력
kubectl get pod pod1 -o json
```
### Describe
```
# 상세 출력
kubectl describe pod pod1
```
### Delete
```
# 파일이 있을 경우 생성한 방법 그대로 삭제
kubectl delete -f ./pod.yaml

# 내용과 함께 바로 작성한 경우 생성한 방법 그대로 삭제
kubectl delete -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END

# Pod 이름 지정
kubectl delete pod pod1
```
### Exec
```
# Pod이름이 pod1인 Container로 들어가기 (나올땐 exit)
kubectl exec pod1 -it /bin/bash

# Container가 두개 이상 있을때 Container이름이 con1인 Container로 들어가기 
kubectl exec pod1 -c con1 -it /bin/bash
```

## Tips
### Apply vs Create
둘다 자원을 생성할때 사용할 수 있지만, `Create`는 기존에 같은 이름의 Pod가 존재하면 생성이 안되고, `Apply`는 기존에 같은 이름의 Pod가 존재하면 업데이트된다.
