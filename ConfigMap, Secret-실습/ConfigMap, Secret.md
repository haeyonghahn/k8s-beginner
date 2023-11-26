## 1. Env (Literal)
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/6ba87071-1324-49ab-9ad6-cba7f97140e4)

### 1-1) ConfigMap
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: 'false'
  User: dev
```

### 1-2) Secret
```yml
apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA==
```

### 1-3) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```

## 2. Env (File)
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/afdc16ab-286f-497f-928f-6742970e1ccd)

### 2-1) Configmap
```
echo "Content" >> file-c.txt
kubectl create configmap cm-file --from-file=./file-c.txt
```

### 2-2) Secret
```
echo "Content" >> file-s.txt
kubectl create secret generic sec-file --from-file=./file-s.txt
```

### 2-3) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-file
spec:
  containers:
  - name: container
    image: kubetm/init
    env:
    - name: file-c
      valueFrom:
        configMapKeyRef:
          name: cm-file
          key: file-c.txt
    - name: file-s
      valueFrom:
        secretKeyRef:
          name: sec-file
          key: file-s.txt
```

## 3. Volume Mount (File)
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/678b394d-387b-4440-8957-82ad4cf5a538)

### 3-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-mount
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount
  volumes:
  - name: file-volume
    configMap:
      name: cm-file
```

## kubectl
### ConfigMap
```
# file-c.txt 라는 파일로 cm-file라는 이름의 ConfigMap 생성
kubectl create configmap cm-file --from-file=./file-c.txt
# key1:value1 라는 상수로 cm-file라는 이름의 ConfigMap 생성
kubectl create configmap cm-file --from-literal=key1=value1
# 여러 key:value로 cm-file라는 이름의 ConfigMap 생성 
kubectl create configmap cm-file --from-literal=key1=value1 --from-literal=key2=value2
```
### Secret Generic
```
# file-s.txt 라는 파일로 sec-file라는 이름의 Secret 생성
kubectl create secret generic sec-file --from-file=./file-s.txt
# key1:value1 라는 상수로 sec-file라는 이름의 Secret 생성
kubectl create secret generic sec-file --from-literal=key1=value1
```

## Tip
- Boolean 값을 넣을 때 따옴표('')를 넣어주어야 에러가 발생하지 않는다.
### Secret
- 데이터가 메모리에 저장되기 때문에 보안에 유리
- 한 Secret당 최대 1M까지만 저장됨
