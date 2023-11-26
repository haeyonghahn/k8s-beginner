## 1. emptyDir
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/5c7e826c-3fbd-48c9-9ec5-7c49b25af7e7)

### 1-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```
```
mount | grep mount1
echo "file context" >> file.txt
```

## 2. hostPath
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/8327137a-62ce-4764-99e1-202188d1dbd2)

### 2-1) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
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

## 3. PVC / PV
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/4c2c2850-7b38-4d8a-a48e-34049c08bdaa)

### 3-1) PersistentVolume
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-03
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
```

### 3-2) PersistentVolumeClaim
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-01
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
```

### 3-3) Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: pvc-pv
      mountPath: /mount3
  volumes:
  - name : pvc-pv
    persistentVolumeClaim:
      claimName: pvc-01
```

## Tips
- PV에 PVC가 바인딩되면 다른 클레임에서 사용할 수 없다.
### hostPath Type
- DirectoryOrCreate : 실제 경로가 없다면 생성
- Directory : 실제 경로가 있어야됨
- FileOrCreate : 실제 경로에 파일이 없다면 생성
- File : 실제 파일이 었어야함
