## 1. DaemonSet
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/2742f2a0-5202-4ac5-89d1-ad3989d19714)

### 1-1) DaemonSet - HostPort
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-1
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
          hostPort: 18080
```
Command
```
curl 192.168.56.31:18080/hostname
```

### 1-2) DaemonSet - NodeSelector
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-2
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      nodeSelector:
        os: centos
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
```
Label Add
```
kubectl label nodes k8s-node1 os=centos
kubectl label nodes k8s-node2 os=ubuntu
```
Label Remove
```
kubectl label nodes k8s-node2 os-
```

## 2. Job
### 2-1) Job1
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
      terminationGracePeriodSeconds: 0
```
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/21c76bf1-c535-4f2d-909b-0a2bc1e9e530)

### 2-2) Job2
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-2
spec:
  completions: 6
  parallelism: 2
  activeDeadlineSeconds: 30
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
      terminationGracePeriodSeconds: 0
```

## 3. CronJob
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/83c06e59-37e5-477d-8f43-d7cd3c4ad95d)

### 3-1) CronJob
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```
Manual
```
kubectl create job --from=cronjob/cron-job cron-job-manual-001
```
Suspend
```
kubectl patch cronjobs cron-job -p '{"spec" : {"suspend" : false }}'
```

### 3-2) CronJob - ConcurrencyPolicy
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/5ddc2184-ce8b-4b8a-9615-a63cc4881bcd)

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job-2
spec:
  schedule: "20,21,22 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["sh", "-c", "echo 'job start';sleep 140; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```
