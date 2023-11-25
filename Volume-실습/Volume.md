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
