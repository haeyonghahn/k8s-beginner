# k8s-beginner
해당 문서 출처는 [대세는 쿠버네티스 [초급~중급]](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88) 기반으로 작성되었습니다. 

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

## Controller
- Auto Healing : Node 위에 있는 Pod가 있을 때 해당 Pod가 갑자기 다운된다거나 Pod가 스케줄링되어 있는 Node가 갑자기 다운되면 해당 Pod에서 돌아가고 있는 서비스에 장애가 발생한다. 이것을 Controller가 즉각적으로 인지하고 Pod를 다른 Node에 새로 만들어준다. 이것을 `Auto Healing` 기능이라고 한다.
- Auto Scaling : Pod의 리소스가 limit 상태가 되었을 때 Controller는 해당 상태를 파악하고 Pod를 하나 더 만들어 줌으로써 부하를 분산시키고 Pod가 죽지 않도록 해준다. 이것을 `Auto Scaling` 기능이라고 한다.
- Software Update : 여러 Pod에 대한 버전을 업그레이드 해야 될 경우 Controller를 통해서 한번에 쉽게 할 수 있고 업그레이드 도중에 문제가 생기면 롤백을 할 수 있는 기능을 제공한다.
- Job : 일시적은 작업을 해야 될 경우 Controller가 필요한 순간에 Pod를 만들어서 해당 작업을 이행하고 삭제시킬 수 있다. 이렇게 되면 그 순간에만 자원이 사용되고 작업 후에 다시 반환되기 때문에 효율적인 자원 활용이 가능해진다.

## Controller - Replication Controller, ReplicaSet
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/ee0bc109-f333-4f05-b03f-38b8c83f707d)

Replication Controller는 Deprecated된 오브젝트이다. 이걸 대체로 ReplicaSet이 있다. 아래의 `Template`과 `Replicas`는 두 오브젝트의 공통된 기능이고 `Selector`는 ReplicaSet에만 좀 더 확장된 기능을 가지고 있다. 

### Template
Controller와 Pod는 Service와 Pod처럼 selector로 연결이 된다. 그래서 Label이 붙어 있는 Pod가 있고 selector와 매핑되는 controller를 만들면 연결이 된다. 그리고 controller를 만들 때 template으로 pod의 내용을 넣게 되는데 Controller는 Pod가 죽으면 재생성시키기 때문에 Pod가 다운되면 template으로 Pod를 새로 만들어주게 된다. 이러한 특성을 사용해서 앱에 대한 업그레이드를 할 수 있는데 template v2에 대한 Pod를 업데이트하고 기존에 연결되어 있는 Pod를 다운시키면 Controller는 template을 가지고 Pod를 재생성하면서 버전 업그레이드를 할 수 있다. 

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    type: web
spec:
  containers:
  - name: container
    image:
      tmkube/app:v1
```
```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: replication-1
spec:
  replicas: 1
  selector:
    type: web
  template:
    metadata:
      name: pod-1
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: tmkube/app:v2
```

### Replicas
그림에 나와있는 replicas 숫자만큼 Pod의 개수가 관리가 된다. Pod가 삭제되면 하나의 Pod만 재생성해준다. 그런데 만약에 해당 수치를 3으로 늘리게 되면 그 수만큼 Pod가 늘어나면서 Scale-out이 되는 것이다. 마찬가지로 Pod들을 모두 삭제하면 Controller는 replicas에 대한 개수만큼 3개의 Pod를 다시 만들어준다. 위의 yml 파일의 내용을 보면 replicas 개수를 늘려주게 되면 scale-out이 되는 것이고 반대로 수치를 내리면 scale-in이 되는 것이다. 그리고 template 기능과 replicas 기능을 통해서 Pod와 Controller를 따로 만들지 않고 한 번에 만들 수 있는데 위의 yml 파일처럼 내용을 담아서 Pod없이 Controller를 만들면 Controller는 replicas가 2로 되어 있는 부분은 현재 Pod가 없기 때문에 template에 있는 Pod의 내용을 가지고 2개의 Pod를 만든다. 실제로 Controller를 사용할 때 이렇게 Controller에 대한 내용만 만들어서 사용한다.

### Selector
Replication Controller의 selector에는 key와 value가 같은 label의 Pod들과 연결을 해준다. 반면 ReplicaSet은 selector에 두 가지 추가적인 속성이 있는데 하나는 matchLabels라고 해서 Replication Controller와 동일하게 key와 value의 label이 같은 Pod들만 연결을 해준다. key와 value 중에 하나라도 다르면 연결을 하지 않는다. 그리고 다른 하나는 `matchExpressions`라는 속성이 있는데 value를 좀 더 디테일하게 컨트롤할 수 있다.

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-1
spec:
  replicas: 3
  selector:
    matchLabels:
      type: web
    matchExpressions:
    - {key: ver, operator: Exists}
template:
  metadata:
    name: pod
...
```

## Controller - Deployment
Deployment는 현재 서비스가 운영 중인데 서비스를 업데이트해야 돼서 재배포를 해야될 때 도움을 주는 Controller이다.

![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/dff8d152-5665-48e9-8f0b-04e6787c3a7f)

### ReCreate
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/ca61392e-f7f0-4a5a-a44f-936b6124aa3f)

Deployment를 만들 때 replica에서 넣었던 selector와 replicas 그리고 template을 똑같이 넣게 된다. 하지만 이 값들은 직접 Deployment가 Pod를 만들어서 관리를 하기 위한 것은 아니고 ReplicaSet을 만들고 여기에 값들을 지정하기 위한 용도로 사용된다. 그래서 만들어진 ReplicaSet은 Pod를 만들게 된다. 그리고 Service를 만들어서 Service에 붙어있는 Label에 연결하면 Service를 통해서 Pod에 접근할 수 있게 된다. 본격적으로 ReCreate 업그레이드를 하려면 template을 v2 버전으로 업데이트해주면 되는데 그러면 Deployment는 먼저 ReplicaSet에 replicas를 0으로 변경한다. 그럼 ReplicaSet은 Pod들을 제거하고 서비스도 연결 대상이 없어지기 때문에 다운타임이 발생한다. 그리고 새로운 ReplicaSet을 만드는데 template에는 변경된 v2의 Pod를 넣기 때문에 Pod들도 v2버전으로 생성이 된다. Service에는 Label이 있어서 자동적으로 Pod들에 연결이 된다. 그럼 ReCreate 배포는 끝나게 된다. 

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
yml 파일의 내용을 보면 배포 방식을 정할 수가 있는데 Strategy 타입에 ReCreate라고 지정한다. `revisionHistoryLimit`는 새로운 ReplicaSet을 만들었지만 기존에 있는 ReplicaSet은 지우지 않았다. 만약 이 상태에서 새로운 버전으로 업그레이드를 하게 되면 ReplicaSet도 지워지지 않고 남아있게 되고 새로운 ReplicaSet이 만들어지게 된다. 근데 1이라고 지정하면 이 상태에서 새로 업그리에드할 때 새로운 ReplicaSet이 생기면서 0이 되고 해당 ReplicaSet은 삭제된다. 그러니까 0인 ReplicaSet을 하나만 남기겠다는 의미이다. 
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
```

### Rolling Update (default)
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/63293439-5918-45e4-be7b-dcec5635c438)

새로운 버전으로 템플릿을 교체하게 되면서 Rolling Update가 시작되는데 먼저 ReplicaSet을 하나 만들고 해당 상태에서는 ReplicaSet이 하나이기 때문에 Pod가 하나 만들어지면서 v1 라벨과 똑같은 라벨(v2)이 만들어지니까 Service는 연결이 된다. 그래서 이제 Service에 연결을 하면 v1과 v2의 트래픽이 분산되서 보내지게 된다. 그런 다음에 기존에 있던 replicas를 1로 변경이 되면서 Pod가 하나 줄어들고 삭제가 완료되면 v2의 replicas를 2로 만들어서 하나를 더 늘리고 마지막으로 v1을 0으로 만들면서 남은 Pod들도 모두 없애준다. 이것도 마찬가지로 ReplicaSet을 지우지는 않고 배포를 종료하게 된다. 

```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    type: app
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
yml 파일의 내용을 보면 strategy type에 RollingUpdate라고 명시가 되어 있고 `minReadySeconds`라고 해서 10으로 해놓았는데 해당 값없이 업데이트를 하게 되면 v1과 v2의 Pod들이 추가되고 삭제되는 게 순식간에 진행이 된다. 근데 이렇게 값을 지정하게 되면 10초라는 텀을 갖고 진행을 하게 된다.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  selector:
    matchLabels:
      type: app
  replicas: 2
  strategy:
    type: RollingUpdate
  minReadySeconds: 10
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
```

## Controller - DeamonSet, Job, CronJob
![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/f878ad82-cf3e-42ff-ab49-0bee25ef0c69)

__DeamonSet__   
각각의 Node에 자원이 다르게 남아있는 상태에서 이전에 배운 ReplicaSet의 경우 Pod를 스케줄러에 의존해서 Node에 배치를 할 때 만약 Node1에 자원이 많이 남아있는 상태라면 Pod를 많이 배치를 할 것이다. 그리고 Node3처럼 자원이 별로 없다면 Pod를 배치안할 수도 있다. 반면 DaemonSet은 노드의 자원 상태랑 상관없이 모든 Node에 Pod가 하나씩 생긴다는 특징이 있다. 만약 Node가 10개면 각각의 Node에 하나씩 총 10개의 Pod가 생기는 것이다. 이렇게 각각의 Node마다 설치를 해서 사용해야 되는 서비스들이 있다. 대표적으로 첫째, 성능수집인데 각 Node들의 성능상태는 모두 감시 대상이다. 그래서 모니터링 화면이 있을 것이고 각각의 노드에 Prometheus 같은 성능 수집 에이전트가 깔려 있어야 모든 노드들의 정보들을 모니터링 시스템에 전달해 줄 수가 있다. 둘째, 로그 수집이다. 특정 노드에 장애가 발생했을 경우 문제를 파악하려면 로그를 봐야 한다. 이렇게 Fluentd와 같은 서비스는 각각의 노드에 설치돼서 로그 정보를 수집한다. 마지막으로 노드들을 스토리지에 활용할 수가 있는데 GlusterFS처럼 각각의 노드에 설치돼서 해당 자원을 가지고 네트워크 파일 시스템을 구축할 수가 있다. 그리고 Kubernetes 자체도 네트워킹 관리를 하기 위해서 각각의 노드에 DeamonSet으로 프록시 역할을 하는 Pod를 만든다. 

__Job, CronJob__   
ReplicaSet에 의해 만들어진 Pod가 있고 Job을 통해서 만들어진 Pod가 있다. 같은 Pod들이지만 누구에 의해서 만들어졌냐에 따라서 달라지는 부분들이 있다. Pod들이 Node1에서 돌아가고 있는 상태에서 해당 Node가 다운이 됐다. 그리고 직접 만들어진 Pod도 장애가 발생한 것이다. 그렇다면 해당 서비스는 더 이상 유지될 수가 없다. 근데 이렇게 Controller에 의해서 만들어진 Pod들은 Controller에 의해서 장애가 감지가 되면 다른 Node에 재생성되기 때문에 서비스는 계속 유지가 된다. 그리고 이렇게 ReplicaSet에서 만들어진 Pod는 일을 하지 않으면 다시 Pod를 restart시켜주기 때문에 Pod에 있는 서비스는 무슨 일이 있어도 서비스가 유지되어야 하는 목적으로 써야 한다. 잠시 여기서 `Recreate`와 `Restart`에 대한 차이는 Recreate는 Pod를 다시 만들어주기 때문에 Pod의 이름이나 IP들이 변경되고 Restart는 Pod는 그대로 있고 Pod 안에 있는 컨테이너만 재기동시켜 준다는 차이점이 있다. 반면 Job으로 만들어진 Pod는 프로세서가 일을 하지 않으면 Pod는 종료가 된다. 이때 종료의 의미는 Pod가 삭제되는 건 아니고 자원을 사용하지 않는 상태로 멈춰 있다는 것인데 우리가 작업을 걸어 놓고 끝난다고 해서 Pod가 지워지면 결과를 못 본다. 우리는 해당 Pod 안에 들어가서 로그를 확인할 수 있다. 그 이후에 필요가 없으면 직접 삭제를 하면 된다. 이렇게 Pod를 만드는 주체에 따라서 상황별로 Pod의 동작이 틀리기 때문에 잘 알고 사용해야 한다. 그리고 CronJob은 주기적인 시간에 따라서 생성을 하는 역할을 하는데 대체로 CronJob을 하나 단위로는 사용하지 않고 CronJob을 만들어서 특정 시간에 반복적으로 실행할 목적으로 사용이 된다. 

![image](https://github.com/haeyonghahn/k8s-beginner/assets/31242766/c378e85d-3bad-4cb6-92dd-d854c2fbfdee)

### DaemonSet
DaemonSet은 한 노드에 하나를 추가해서 Pod를 만들 수는 없지만 노드에 Pod를 안 만들 순 있다. 그리고 특정 노드로 접근을 했을 때 이 노드에 들어있는 Pod에 접근이 되도록 많이 사용을 하는데, 이전 학습에서 노드 타입의 서비스를 만들고 해당 옵션을 추가하면 특정 노드에 노드 포트로 접근을 하면 트랙픽은 서비스로 가고 서비스가 해당 노드에 Pod로 연결이 되도록 했다. 이렇게 hostPort라고 해서 해당 설정을 지정하면 직접 노드에 있는 포트가 Pod로 연결이 돼서 똑같은 결과를 얻을 수가 있다.

yml 파일을 보면 18080번 포트로 들어온 트래픽은 8080번 컨테이너 포트로 연결이 된다.
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
  nodeSelector:
    os: centos
  containers:
  - name: container
    image: tmkube/app
    ports:
    - containerPort: 8080
      hostPort: 18080
```

### Job
하나의 Pod를 생성하고 해당 파드가 일을 다하면 Job도 종료가 되지만 completions라고 해서 값을 6을 주면 6개의 Pod를 하나씩 순차적으로 실행시켜서 모두 작업이 끝나야 Job도 종료가 된다. 그리고 parallelism 옵션은 값을 2를 주면 2개씩 Pod가 생성되고 activeDeadlineSeconds의 값을 30을 주면 30초 후에 기능이 정지해버린다. 그리고 실행되고 있는 모든 Pod는 삭제가 된다. 아직 실행되지 못한 Pod들도 앞으로 실행이 안된다. 이걸 어느 용도로 사용하냐면 만약 10초가 걸릴 일에 Job을 만들었는데 30초가 되도록 작업이 끝나지 않으면 뭔가 문제가 걸렸을 확률이 큰 거고 그럴 경우 Pod들을 삭제해서 자원을 릴리즈하고 더이상 작업을 진행하지 않도록 설정을 할 때 사용을 한다. restartPolicy는 Never가 고정된 값이고 Never와 onfailer만 사용할 수 있다.

```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  completions: 6
  parallelism: 2
  activeDeadlineSeconds: 30
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: tmkube/init
```

### CronJob
CronJob은 jobTemplate이 있어서 해당 내용을 통해 Job을 만들어주고 스케줄이 있어서 해당 시간을 주기로 Job을 만든다. 그림처럼 1분으로 설정을 하면 1분 간격으로 Job이 생성되고 Job 또 자신의 역할인 Pod를 만들게 된다. 그리고 conCurrencyPolicy라는 기능이 있는데 `Allow`, `Forbid`, `Replace` 3가지 옵션이 있다. 일반적으로 policy를 설정하지 않으면 allow가 디폴트 값이다. allow는 1분 간격으로 스케줄한다고 설정을 했을 때 1분이 됐을 때 Job이 만들어지고 Pod가 생성된다. 그리고 2분이 됐을 때 사전에 만들어진 Pod가 Running 중이거나 아니면 종료가 됐던 간에 상관없이 자신의 스케줄 타임이 되면 또 새로운 Job을 만들고 Pod가 생긴다. 마찬가지로 3분이 됐을 때도 Job이 만들어진다. Forbid는 1분이 됐을 때 Job이 생성되지만 2분이 됐을 때까지 Pod가 종료되지 않고 실행이 되고 있으면 이때 2분째 생겨야 되는 Job은 스킵이 되고 Pod가 종료되는 즉시 다음 스케줄 타임에 있는 Job이 만들어진다. Replace는 1분의 Job이 만들어졌고 2분 스케줄이 됐는데 Pod가 계속 Running 중이라면 새로운 Pod를 만들어서 해당 Job 연결을 새로운 Pod로 교체시켜준다. 2분이 됐을 때 새로운 잡은 생기지 않지만 새로운 Pod가 생기면서 이전 스케줄 때 만들었던 Job이 연결이 되는 것이다. 이 Pod가 자신의 스케줄 타임에 종료가 됐다면 3분 스케줄이 됐을 때 새로운 Job이 만들어지고 Pod가 만들어지게 된다.

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "* /1 * * * *"
  concurrencyPolicy: Allow
  jobTemplate:
  template:
    spec:
      template:
        restartPolicy: Never
        containers:
        - name: container
          image: tmkube/app
```
