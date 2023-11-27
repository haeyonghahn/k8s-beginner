## 1. Namespace
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/34eae57e-74c4-456a-972a-a433a823d9c5)

### 1-1) Namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
```

### 1-2) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

### 1-3) Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```

### 1-1') Namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```

### 1-2') Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-2
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/init
    ports:
    - containerPort: 8080
```
```
pod ip : curl 10.16.36.115:8080/hostname   
service ip : curl 10.96.92.97:9000/hostname
```

## Namespace Exception

### e-1) Service
노드포트는 Namespace로 나눌 수 없다.
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
```

### e-2) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
 name: pod-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```
```
echo "hello" >> hello.txt
```

## 2. ResourceQuota

### 2-1) Namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-3
```

### 2-2) ResourceQuota
```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```
```
kubectl describe resourcequotas --namespace=nm-3
```

### 2-3) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  containers:
  - name: container
    image: kubetm/app
```

### 2-4) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.5Gi
      limits:
        memory: 0.5Gi
```

### 2-5) Pod
파드 개수를 제한할 수 있다.
```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-2
spec:
  hard:
    pods: 2
```

## 3. LimitRange
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/a9ccf900-ea6a-462f-aeab-f0add0459516)

### 3-1) Namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-5
```

### 3-2) LimitRange
```yml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```
```
kubectl describe limitranges --namespace=nm-5
```

### 3-3) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.5Gi
```

## LimitRange Exception
하나의 Namespace에 여러 개의 LimitRange를 지정하게 되면 예상치 못한 동작에 의해 Pod가 생성이 안될 수도 있다.

### e-1) Namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-6
```

### e-2) LimitRange
```yml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-5
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.5Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.5Gi
    default:
      memory: 0.5Gi
```

### e-3) LimitRange
```yml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-3
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.3Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.3Gi
    default:
      memory: 0.3Gi
```

### e-4) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
```

## Tips
- Namespace에 ResourceQuota를 달면 Pod에 자신이 사용할 자원을 명시해야 한다.
- ResourceQuota는 Namespace 뿐만 아니라 Cluster 전체에 부여할 수 있는 권한이지만, LimitRange의 경우 Namespace내에서만 사용 가능하다.
