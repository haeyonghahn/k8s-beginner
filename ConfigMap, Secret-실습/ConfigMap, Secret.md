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

## Tip
- Boolean 값을 넣을 때 따옴표('')를 넣어주어야 에러가 발생하지 않는다.
