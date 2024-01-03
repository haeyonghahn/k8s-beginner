## 1. Template, Replicas
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/42b955b8-9972-4acb-b0e6-95c41adf82a9)

### 1-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    type: web
spec:
  containers:
  - name: container
    image: kubetm/app:v1
  terminationGracePeriodSeconds: 0
```
- terminationGracePeriodSeconds : 파드가 삭제 시 기본적으로 30초 후에 삭제가 되도록 설정이 되어 있다. 해당 옵션을 설정해 놓으면 삭제 시 바로 삭제가 된다.

### 1-2) ReplicaSet
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
  template:
    metadata:
      name: pod1
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```

## 2. Updating Controller
### ReplicationController
```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: replication1
spec:
  replicas: 2
  selector:
    cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
```
### kubectl
replicationcontroller는 대시보드에서 삭제할 수 없고 master에서 kubectl 명령어로 삭제해야 한다.
```
kubectl delete replicationcontrollers replication1 --cascade=false
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
      cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
```
## 3. Selector
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/d5fa8482-8686-45fd-9f14-a65434da9e97)

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
      ver: v1
    matchExpressions:
    - {key: type, operator: In, values: [web]}
    - {key: ver, operator: Exists}
  template:
    metadata:
      labels:
        type: web
        ver: v1
        location: dev
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
- ReplicaSet의 Selector 사용 시 주의 사항
1. selector에 있는 내용이 template에 있는 label의 내용에 포함이 되어야 한다.
2. selector의 matchLabels와 matchExpressions를 동시에 사용할 수 있고 여러가지 조건을 넣을 수 있는데, 마찬가지로 labels에 포함이 되어야 한다.
