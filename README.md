# k8s-beginner
## Pod
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

## Service
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
NodePort 타입으로 만들어도 서비스에는 기본적으로 ClusterIP가 할당이 돼서 ClusterIP 타입과 같은 기능이 포함되어 있다. 노드 타입만의 큰 특징은 Kubernetes 클러스터에 연결되어 있는 노드한테 똑같은 포트가 할당이 돼서 외부로부터 어느 노드건간에 그 IP의 포트로 접속을 하면 서비스에 연결이 된다. 그럼 또 서비스는 기본 역할인 자신한테 연결되어 있는 하드의 트래픽을 전달해준다. 주의할 점은 Pod가 있는 Node에만 포트가 할당되는 것이 아니라 모든 Node에 Port가 만들어진다는 게 특징이다.

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
