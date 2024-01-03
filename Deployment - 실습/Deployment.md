## 1. ReCreate
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/4665fa88-d73e-4b01-95c0-3bfadb692fe4)

### 1-1) Deployment
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  selector:
    matchLabels:
      type: app
  replicas: 2
  strategy:
    type: Recreate
  revisionHistoryLimit: 1
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 10
```
- revisionHistoryLimit : 해당 옵션은 버전이 업그레이드 되면서 기존에 존재하던 버전의 히스토리를 몇 개까지 남길 것인지 설정하는 옵션이다. 만약 해당 옵션을 1로 설정하고 v1 -> v2 -> v3으로 버전이 업그레이드되었을 때
v1은 ReplicaSet에서 삭제되며 v2는 남아있다. 만약 옵션을 0로 설정했을 경우 v1까지 히스토리가 삭제되지 않고 남아있을 것이다.

### 1-2) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: app
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
### Command
Recreate 배포 진행을 보기 위해 1초마다 버전을 확인할 수 있는 명령어
```
while true; do curl {서비스클러스터IP:서비스클러스터Port}/version; sleep 1; done
```
### Kubectl
아래의 명령어들은 배포되었던 대상을 롤백하기 위한 명령어이다. 

해당 명령어는 배포되었던 히스토리의 버전을 확인할 수 있다.
```
kubectl rollout history deployment deployment-1
```
명령어를 통해서 확인된 버전을 `--to-revision={버전}` 해당 옵션에서 지정하고 명령어를 통해 이전 버전으로 롤백시킬 수 있다.
```
kubectl rollout undo deployment deployment-1 --to-revision=2
```

## 2. RollingUpdate
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/5ddae267-b366-4aed-a8d3-19f787a5f001)

RollingUpdate는 ReCreate와 다르게 DownTime이 발생하지 않는다.

### 2-1) Deployment
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  selector:
    matchLabels:
      type: app2
  replicas: 2
  strategy:
    type: RollingUpdate
  minReadySeconds: 10
  template:
    metadata:
      labels:
        type: app2
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```

### 2-2) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    type: app2
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
### Command
RollingUpdate 배포 진행을 보기 위해 1초마다 버전을 확인할 수 있는 명령어
```
while true; do curl {서비스클러스터IP:서비스클러스터Port}/version; sleep 1; done
```
## 3. Blue/Green
### ReplicaSet
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v1
  template:
    metadata:
      name: pod1
      labels:
        ver: v1
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
### Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    ver: v1
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
### ReplicaSet
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica2
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v2
  template:
    metadata:
      name: pod1
      labels:
        ver: v2
    spec:
      containers:
      - name: container
        image: kubetm/app:v2
      terminationGracePeriodSeconds: 0
```
### Service
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/85bff3f8-a14e-4039-916e-c81bc6eeb8e3)

Service의 selector를 v1 -> v2 로 변경
