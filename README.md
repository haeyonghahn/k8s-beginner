# k8s-beginner
## Object - Pod
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/2eaa11d4-5627-4fa7-ad1c-cdf4a2042b97)

### Container
첫 번째 Pod의 특징을 보면 Pod 안에는 하나의 독립적인 서비스를 구동할 수 있는 컨테이너들이 있다. 컨테이너들은 서비스가 연결될 수 있도록 포트를 가지고 있는데, 한 컨테이너가 포트를 하나 이상 가질 수는 있지만 한 Pod 내에서 컨테이너들끼리 포트가 중복될 순 없다. 두 컨테이너는 한 호스트에 묶여 있다고 보면 되는데, 왼쪽 컨테이너에서 오른쪽 컨테이너로 접근을 할 때 localhost:8080 으로 접근할 수 있다. 그리고 Pod가 생성될 때 고유의 IP 주소가 할당이 되는데 Kubernetes 클러스터 내에서만 해당 IP를 통해서 Pod에 접근할 수가 있다. `외부에서는 해당 IP로 접근을 할 수가 없다.` 그리고 만약 피드에 문제가 생기면 시스템이 이걸 감지해서 Pod를 삭제시키고 다시 재생성을 하기 되는데 이 때 IP 주소는 변경이 된다. 

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container1
    image: tmkube/p8000
    ports:
    - containerPort: 8000
  - name: container2
    image: tmkube/p8080
    ports:
    - containerPort: 8080
```

### Label
Label은 Pod 뿐만 아니라 모든 오브젝트에 달 수 있는데 Pod에서 가장 많이 사용이 된다. Label을 사용하는 이유는 목적에 따라 오브젝트들을 분류하고 그 분류된 오브젝트들만 따로 골라서 연결한다. 라벨의 구성은 `Key`와 `Value`가 한 쌍이다. 한 Pod에는 여러 개의 Label을 달 수가 있다. 6개의 Pod가 있는데 Label을 단 것을 보면 Key가 `type`이고 Value가 `web`인 Pod가 두 개 있다. `type`이 `db`인 것과 `type`이 `dev` 환경과 `production` 환경으로 나눠서 구성이 되어 있다. 다시 정리를 하면 web과 db인 것이 있는데 해당 한 쌍으로 `dev` 서버에 하나가 있는 것이다. 그리고 다른 한 쌍은 `production`에서 돌아가는 Pod이다. 이 상황에서 웹 개발자가 웹 화면만 보고 싶다라고 한다면 type이 `web`인 Label이 달린 Pod들을 서비스에 연결을 해서 서비스의 정보를 웹 개발자에게 알려주면 된다. `이렇게 사용 목적에 따라 Label을 잘 달아 놓으면 우리가 해시태그를 붙여서 검색 용도로 사용하듯이 원하는 Pod를 선택해서 사용할 수 있다.`

아래 yml 파일 내용을 보면 Pod를 만들 때 labels에 key와 value 형식으로 내용을 넣을 수가 있다. 추후 서비스를 만들 때 selector에 key와 value를 넣으면 해당 내용과 매칭되는 label이 붙어 있는 Pod와 연결이 된다.
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
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    type: web
    lo: dev
spec:
  containers:
  - name: container
    image: tmkube/init
```

### Node Schedule
`Pod는 결국 여러 Node들 중에 한 Node에 올라가줘야 된다.` 그 방법에 대해서 직접 Node를 선택하는 방법과 Kubernetes가 자동으로 지정해주는 방법이 있다. 먼저 `직접 선택하는 방법`은 Pod에 Label을 단 것처럼 이번엔 Node에 Label을 달고 Pod를 만들 때 Node를 지정을 할 수 있다. 아래의 yml 파일을 보면 nodeSelector라는 항목에 Node의 Label과 매칭되는 key와 value를 넣으면 된다. 그리고 두번째로 `Kubernetes의 스케줄러가 판단을 해서 지정을 해주는 경우`이다. Node에는 전체 사용 가능한 자원량이 있다. 메모리와 CPU가 대표적인데 일단 메모리를 예를 들면 현재 이 Node에는 몇몇 Pod들이 들어가 있어서 남은 메모리가 1Gi이다. 다른 Node에는 3.7Gi의 메모리가 남은 상황이다. 그리고 Pod를 생성할 때 이 Pod에서 요구될 리소스의 사용량을 명시할 수 있는데 Pod에서 2Gi의 메모리를 요구한다고 했을 때 Kubernetes가 알아서 Pod를 Schedule해준다. 

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    hostname: node1
  containers:
  - name: container
    image: tmkube/init
```
밑에 Pod를 만들 때 yml 파일의 내용을 보면 Pod에서 사용될 리소스는 request가 메모리를 2기가를 요구한다는 뜻이고 limits는 최대 허용 메모리가 3기가라는 내용이다. limit는 메모리가 limit를 넘어 버리면 Pod를 종로시켜버린다. 반면 CPU의 경우 limit를 넘겨버리면 request 수치까지 낮추기는 하지만 직접 Pod를 종료시키지는 않는다.
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: tmkube/init
    resources:
      requests:
        memory: 2Gi
      limits:
        memory: 3Gi
```

## Object - Service
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/b83b194d-62e0-410f-b18b-f505371ddd65)

### ClusterIP
서비스는 기본적으로 자신의 ClusterIP를 가지고 있다. 그리고 이 서비스를 Pod에 연결을 시켜 놓으면 서비스의 IP를 통해서도 Pod에 접근을 할 수가 있게 된다. Pod에도 똑같이 클러스터 내에서 접근할 수 있는 IP가 있는데 굳이 서비스에 달아서 이걸 통해서 접근을 하지? 라고 생각할 수 있다. Pod라는 존재는 Kubernetes에서 시스템 장애건, 성능 장애건 언제든지 죽을 수가 있다. 그러면서 다시 재생성되도록 설계가 되어있는 오브젝트이다. Pod가 죽을 때 Pod의 IP는 재생성이 되면 변한다. 그렇게 때문에 파드의 IP는 신뢰성이 떨어진다. 그런데 서비스는 사용자가 직접 지우지 않는 한 삭제되거나 재생성되고 그러진 않는다. ClusterIP는 클러스터 내에서만 접근이 가능한 IP이다. Pod에 있는 IP와 특징이 똑같다. 그래서 ClusterIP도 외부에서는 접근할 수 없다. 그리고 Pod를 하나만 연결할 수 있는 건 아니고 여러 개의 Pod를 연결시킬 수가 있는데 여러 개의 Pod를 연결시켰을 때 서비스가 트래픽을 분산해서 Pod에 전달해준다.

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
  type: ClusterIP
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: tmkube/app
    ports:
    - containerPort: 8080
```

### NodePort
NodePort 타입으로 만들어도 서비스에는 기본적으로 ClusterIP가 할당이 돼서 ClusterIP 타입과 같은 기능이 포함되어 있다. 노드 타입만의 큰 특징은 Kubernetes 클러스터에 연결되어 있는 노드한테 똑같은 포트가 할당이 돼서 외부로부터 어느 노드건간에 그 IP의 포트로 접속을 하면 서비스에 연결이 된다. 그럼 또 서비스는 기본 역할인 자신한테 연결되어 있는 파드의 트래픽을 전달해준다. 주의할 점은 Pod가 있는 Node에만 포트가 할당되는 것이 아니라 모든 Node에 Port가 만들어진다는 게 특징이다.

30,000번 대에서 23,767번대 사이에서만 할당을 할 수 있다. 해당 값도 옵션이기 때문에 옵션을 적용하지 않으면 자동으로 이 범위 내에서 할당이 된다. 그리고 각 Node에 Pod가 하나씩 올라가 있다. 이 상태에서 1번 Node의 IP로 접근을 하더라도 Service는 2번 Node에 있는 Pod한테 트래픽을 전달할 수가 있다. Service 입장에서는 어떤 Node한테 온 트래픽인지 상관없이 그냥 자신한테 달려있는 Pod들한테 트래픽을 전달해주기 때문이다. 근데 만약 `externalTrafficPolicy` 옵션 값을 `Local`이라고 주면 특정 NodePort의 IP로 접근을 하는 트래픽은 Service가 해당 Node 위에 올려져 있는 Pod한테만 트래픽을 전달해준다.
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
```

### Load Balancer
Load Balancer 타입으로 서비스를 만들면 기본적으로 NodePort의 성격을 그대로 가지고 있다. 그리고 트래픽을 분산시켜주는 역할을 한다. 로그 밸런서에 접근을 하기 위한 외부 접속 IP 주소는 개별적으로 Kubernetes를 설치했을 때 기본적으로 생기지 않는다. 구글 클라우드 플랫폼이나 AWS에서 제공해주는 Kubernetes를 사용할 경우 자체적으로 플러그인이 설치되어 있어서 로그 밸런서 타입으로 서비스를 만들면 알아서 외부에서 접속할 수 있는 IP를 만들어준다.

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

## Object - Volume
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/05d0c455-5147-4d2e-ad2a-00e162ee79ed)

### emptyDir
컨테이너들끼리 데이터를 공유하기 위해서 사용하는 것이다. 볼륨이 생성될 때는 항상 볼륨 안에 내용이 비어 있기 때문에 `emptyDir`라고 명칭이 된 것이다. 만약 Container1이 웹 역할을 하는 서버이고 Container2이 백엔드를 처리해주는 서버라고 했을 때 웹 서버로 받은 어떤 특정 파일을 마운트가 된 볼륨에 저장을 해 놓고 백엔드에 있는 컨테이너 역시 볼륨을 마운트해 놓으면 두 서버가 볼륨을 자신의 로컬에 있는 파일처럼 사용을 하기 때문에 두 서버가 서로 파일을 주고 받을 필요 없이 편하게 사용할 수 있다. 그리고 볼륨은 그림에서처럼 Pod 안에 생성이 되기 때문에 Pod에 문제가 발생해서 재생성이 되면 데이터가 싹 없어진다. 그래서 emptyDir 볼륨은 `꼭 일시적인 허용 목적에 의한 데이터만 넣어주는 것이 좋다.`

yml 내용을 보면 Container1은 `mountPath`를 `/mount1`을 사용했고 Container2는 `mountPath`를 `/mount2`를 사용했다. 서로의 Path이름이 다르더라도 결국 각각의 Path가 지정되는 볼륨이 `emptyDir`로 똑같은 볼륨을 지정을 하고 있기 때문에 컨테이너마다 자신이 원하는 경로를 사용하고 있지만 결국 한 볼륨을 마운트하고 있는 것이다.
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: tmkube/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: tmkube/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```

### hostPath
이름 그대로 한 호스트. Pod들이 올라가 있는 노드의 Path를 볼륨으로 사용하는 것이다. emptyDir와 다른 점은 Path를 각각의 Pod들이 마운트해서 공유하기 때문에 Pod들이 죽어도 노드에 있는 데이터는 사라지지 않는다. 이러한 점에서 좋아 보일 수 있지만 Pod 입장에서는 문제가 있다. 만약 Pod2가 죽어서 재생성이 될 때 꼭 해당 노드에 재생성이되리라는 보장은 없다. 만약 재생성이되는 순간 스케줄러가 자원 상황을 보고 Node2에 Pod를 만들어 줄 수도 있다. 또는 Node1에 장애가 생겨서 다른 Node에 Pod가 옮겨질 수도 있다. 그래서 Pod가 다른 노드로 옮겨졌을 때 원래 있었던 Node에 있는 볼륨을 마운트할 수 없게 된다. 굳이 방법을 찾는다면 Node가 추가될 때마다 똑같은 이름의 경로를 만들어서 직접 노드에 있는 Path끼리 마운트를 시켜주면 되지만 이것은 Kubernetes가 해주는 역할은 아니고 운영자가 직접 노드에 추가될 때마다 리눅스 시스템 별도의 마운트 기술을 사용해서 연결해주어야 한다. 이러한 개입은 실수를 발생할 여지가 있기 때문에 추천하진 않는다. hostPath가 사용되는 용도는 기본적으로 각 노드마다 노드 자신을 위해서 사용되는 파일들이 있을 것이다. 시스템 파일이나 여러 설정 파일이 있는데 Pod 자신이 할당되어 있는 호스트의 데이터를 읽거나 써야할 때 사용하면 된다.

한 가지 주의할 점은 hostPath는 Pod가 만들어지기 전에 사전에 만들어져 있어야 에러가 발생하지 않는다. 다시 한 번 더 말하자면 hostPath는 Pod의 데이터를 저장하기 위한 용도가 아니라 Node에 있는 데이터를 Pod에 쓰기 위한 용도이다.
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-2
spec:
  containers:
  - name: container
    image: tmkube/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: Directory
```

### PVC / PV
Pod에 영속성 있는 볼륨을 제공하기 위한 개념이다. 로컬 볼륨이 있지만 외부의 원격으로 사용되는 형태의 볼륨도 있다. 그림처럼 아마존이나 Git에 연결할 수도 있고 NFS를 써서 다른 서버와 연결할 수도 있다. 그리고 스토리지 OS 같이 볼륨을 직접 만들고 관리할 수 있는 솔루션도 있다. 이러한 것들을 Persistent Volume을 정리하고 연결한다. 그런데 Pod는 PV(Persistent Volume)에 바로 연결하지 않고 Persistent Volume Claim을 통해서 PV와 연결이 된다. 바로 Pod에 PV로 연결하는 것이 더 깔끔해 보일 수가 있지만 중간에 PVC를 두는 것은 Kubernetes는 볼륨 사용에 있어 User 영역과 Admin 영역을 나눠서 Admin은 Kubernetes를 담당하는 운영자일 것이고 User는 Pod의 서비스를 만들고 배포를 관리하는 서비스를 만드는 담당자일 것이다. 그래서 전문적으로 관리하는 운영자가 PV를 만들어 놓으면 User가 PVC를 만들어서 사용하게끔 하는 것이다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  containers:
  - name: container
    image: tmkube/init
    volumeMounts:
    - name: pvc-pv
      mountPath: /volume
  volumes:
  - name : pvc-pv
    persistentVolumeClaim:
      claimName: pvc-01
```
Persistent Volume Claim을 만들기 위한 yml 파일을 보면 읽기 쓰기 모드가 되면서 용량이 1G인 볼륨을 할당해달라고 요청하는 것이다. `storageClassName`은 쌍따움표("")로 작성하면 현재 만들어져 있는 PVC들 중에서 선택이 된다. 쌍따움표("")를 안 넣고 생략을 하면 다른 동작으로 사용될 수 있기 때문에 storageClassName은 다음 과정에서 더 배워보자.  
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
Persistent Volume은 hostPath처럼 localPath를 사용하는 것이고 PV에 연결되는 Pod들은 Node1이라고 라벨링이 되어있는 Node 위에만 무조건 만들어진다는 뜻이다. `capacity`와 `accessModes`를 지정해두면 PVC에서 PV를 Kubernetes가 자동으로 연결해줄 때 어떠한 근거가 필요한데 `capacity`와 `accessMode`를 보고 Kubernetes가 연결해 준다.
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-01
spec:
  capacity:
    storage: 1G
  accessModes:
    - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: node, operator: In, values: [node1]}
```

## Object - ConfigMap, Secret
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/e422386d-16ee-4679-a15b-0fd784f2e490)

### Env (Literal)
 가장 기본적인 형태인 상수를 넣는 방법으로 ConfigMap은 Key와 Value로 고정이 되어 있다. 그래서 필요한 상수들을 정의해 놓으면 Pod를 생성할 때 ConfigMap을 가져와서 컨테이너 안에 환경 변수에 셋팅을 할 수가 있다. 그리고 Secret도 똑같은 역할을 하는데 이름에서 느껴지는 것처럼 보안적인 요소의 값들을 저장하는 용도로 사용된다. 주로 패스워드라든지 인증키를 Secret에 담는다. 그리고 `사용상 ConfigMap과 다른 점은 Value를 넣을 때 Base64 Encoding을 해서 만들어야 된다는 것이다. 이것은 단순히 Secret을 만들 때 규칙이고 Pod로 주입될 때에는 자동으로 decoding이 되서 환경 변수에서는 원래의 값이 보이게 된다.` 그리고 일반적인 오브젝트 값들은 Kubernetes의 DB에 저장이 되는데 Secret은 메모리에 저장이 된다. `ConfigMap의 경우 Key와 Value 목록을 무한히 넣을 수 있는데에 반해 Secret은 1MB까지만 넣을 수 있다.`

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: False
  User: dev
```
```yml
apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA==
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: tmkube/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```

### Env (File)
파일을 통으로 ConfigMap에 담을 수가 있는데 이럴 때 파일 이름이 Key가 되고 파일 안의 내용은 Value가 되서 ConfigMap이 만들어진다. 만드는 방법은 파일을 ConfigMap으로 만드는 것은 대시보드에서 지원해주지 않기 때문에 직접 마스터의 콘솔로 들어가서 `kubectl` 명령을 실행한다. 한가지 주의할 점은 Secret의 경우 텍스트 안에 내용이 Base64로 변경이 되기 때문에 만약에 파일 안에 내용이 이미 Base64였다면 두번 인코딩이 되는 셈이기 때문에 유의해야 한다.
```
kubectl create configmap cm-file --from-file=./file.txt
kubectl create secret generic sec-file --from-file=./file.txt
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: file
spec:
  containers:
  - name: container
    image: tmkube/init
    env:
- name: file
  valueFrom:
    configMapKeyRef:
      name: cm-file
      key: file.txt
```

### Volume Mount (File)
파일을 마운팅하는 방법이다. 파일을 ConfigMap에 담는 데까지는 동일하다. Pod를 만들 때 컨테이너 안에 mountPath를 지정하고 Path안에 파일을 마운트할 수 있다. Env (File)과 Volume Mount (File)의 큰 차이점은 만약 Pod를 생성한 다음에 각각의 ConfigMap에 내용을 변경하게 된다면 `환경변수 방식은 한번 주입을 하면 끝이기 때문에 ConfigMap의 데이터가 변경되어도 Pod의 환경변수 값에는 영향이 없다. 반면에 File Mount는 마운트라는 것이 원본과 연결시켜준다는 개념이기 때문에 내용이 변경되면 Pod에 마운팅된 내용도 변하게 된다.` 그렇게 때문에 이러한 특성을 잘 알고 필요한 상황에 따라 활용하면 된다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: mount
spec:
  containers:
  - name: container
    image: tmkube/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount
  volumes:
  - name: file-volume
    configMap:
      name: cm-file
```

## Object - Namespace, ResourceQuota, LimitRange
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/2fa88d56-4033-4d03-a960-faa072968902)

### Namespace
한 네임스페이스 안에는 같은 타입의 오브젝트들은 이름이 중복될 수 없다는 특징을 가지고 있다. 오브젝트들마다 별도의 UUID가 존재하긴 하지만 네임스페이스 안에서는 같은 종류의 오브젝트라면 이름 또한 UUID 같이 유일한 키 역할을 할 수가 있는 셈이다. 그리고 네임스페이스의 대표적인 특징이 타 네임스페이스에 있는 자원과 분리가 돼서 관리가 된다는 것이다. Namespace-2가 있고 그 안에는 서비스가 존재한다. 위에서 Pod와 Service 간의 연결을 Pod에는 Label을 달고 Service에는 Selector를 달아서 연결을 한다. `그런데 타 네임스페이스에 있는 Pod에는 연결이 되지 않는다.` 이뿐만 아니라 대부분의 자원들은 네임스페이스 안에서만 사용할 수가 있다. 물론 Node나 PV같이 네임스페이스에서 공용으로 사용되는 오브젝트가 존재하긴 한다. 그리고 네임스페이스를 지우게 되면 그 안에 있는 자원들도 모두 지워진다. 네임스페이스를 지울 때 유의해야 한다. 그런데 Pod마다 IP를 가지고 있는데 Namepace-2에 있는 Pod1에서 Namespace-1에 있는 Pod1에 접근을 하려면 나중에 학습할 Networking Pause라는 오브젝트를 이용해서 접근이 가능하다.

yml 파일을 보면 Namespace를 만들 때 이름 외에는 특별하게 설정되는 옵션이 없다. Pod나 Service를 만들 때 내가 속한 Namespace를 지정할 수 있다. 두 오브젝트는 Namespace가 서로 다르기 때문에 Selector의 값과 Label의 값이 일치하더라도 연결되지 않는다.
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    nm: pod1
spec:
  containers:
  ...
```
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-2
spec:
  selector:
    nm: pod1
  ports:
  ...
```

### ResourceQuota
ResourceQuota는 Namespace에 자원 한계를 설정하는 오브젝트이다. Namespace에 제한하고 싶은 자원을 명시해서 아래의 yml 파일을 보면 Namespace에 들어갈 Pod들의 전체 Request 자원들을 최대 3Gi로 설정하겠다는 의미이고 memory의 limit은 6Gi로 설정하겠다는 의미이다. 한가지 중요한 것은 ResourceQuota가 지정되어 있는 Namespace에 Pod를 만들 때 Pod는 무조건 spec을 명시해야 한다. spec자체가 없으면 Namespace에 만들어지지 않는다. 그리고 현재 총 3Gi의 Request 중에 2Gi를 사용하는 Pod가 있는데 2Gi를 더 사용하는 Pod가 들어온다면 Pod가 만들어지지 않는다. yml 파일을 보면 ResourceQuota를 만들 때 할당한 Namespace를 설정하고 hard라는 속성에 제한할 종류와 자원과 그 한계치가 들어가게 된다. 메모리 뿐만 아니라 제한할 수 있는 건 CPU와 스토리지가 있다.

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-1
spec:
  hard:
    requests.memory: 3Gi
    limits.memory: 6Gi
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  containers:
  - name: container
    image: 
      tmkube/app
    resources:
      requests:
        memory: 2Gi
      limits:
        memory: 2Gi
```

### LimitRange
LimitRange의 기능은 각각의 Pod마다 Namespace에 들어올 수 있는지 자원을 체크해준다. 체크되는 항목 중 min은 Pod에서 설정되는 메모리의 Limit 값이 1Gi가 넘어야 된다는 것이고 max는 4Gi를 초과할 수 없다는 의미이다. 그리고 maxLimitRequestRatio는 그림처럼 3을 설정하게 되면 Request 값과 limit의 비율이 최대 3배를 넘으면 안된다는 의미이다. 예시를 보면 Pad1의 경우 설정된 limit의 값이 memory가 5Gi이다. 하지만 LimitRange의 memory 값은 4Gi이기 때문에 Pod1은 들어올 수 없다. Pod2의 경우 Request와 limit의 값이 1과 4로 4배의 비율인데 3배까지만 허용된다고 되어 있기 때문에 Pod2 역시 들어올 수 없다. 추가적인 옵션으로 defaultRequest와 default라는 항목이 있는데 해당 설정을 지정해 놓으면 Pod에 자동으로 Request 값과 limit의 값이 명시된 값으로 할당이 된다.

```yml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
  namespace: nm-1
spec:
  limits:
  - type: Container
    min:
      memory: 1Gi
    max:
      memory: 4Gi
    defaultRequest:
      memory: 1Gi
    default:
      memroy: 2Gi
    maxLimitRequestRatio:
      memory: 3
```
