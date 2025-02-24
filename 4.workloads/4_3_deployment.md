# Kubernetes Deployment 생성 방법

Kubernetes에서 **Deployment**는 애플리케이션을 배포하고 관리하는 주요 리소스입니다. Pod를 관리하고 업데이트를 쉽게 수행할 수 있도록 해줍니다. Deployment를 생성하는 방법에는 **CLI(Command Line Interface) 사용**과 **YAML 파일 정의 후 적용** 두 가지 방식이 있습니다.

---

## 1. CLI를 사용하여 Deployment 생성

```bash
kubectl create deployment my-deployment --image=nginx --replicas=3
```

### 설명
- `kubectl create deployment` : Deployment 생성 명령어
- `my-deployment` : Deployment의 이름 지정
- `--image=nginx` : 사용할 컨테이너 이미지
- `--replicas=3` : Pod 개수 지정 (기본값: 1)

Deployment가 생성되었는지 확인하려면:
```bash
kubectl get deployments
```

Pod 목록 확인:
```bash
kubectl get pods
```

삭제하려면:
```bash
kubectl delete deployment my-deployment
```

---

## 2. YAML 파일을 사용하여 Deployment 생성

### deployment.yaml 파일 작성
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
        ports:
        - containerPort: 80
```

### YAML을 사용한 Deployment 생성
```bash
kubectl apply -f deployment.yaml
```

### 생성된 Deployment 확인
```bash
kubectl get deployments
```

### Pod 목록 확인
```bash
kubectl get pods
```

### Deployment 삭제
```bash
kubectl delete -f deployment.yaml
```

---

## CLI vs YAML 방식 비교

| 방법 | 특징 |
|------|------|
| **CLI** | 빠르게 Deployment 생성 가능, 간단한 테스트에 유용 |
| **YAML** | 선언적 방식, 버전 관리 가능, 재사용 가능 |

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

Deployment는 애플리케이션을 지속적으로 실행하고, 업데이트 및 확장을 관리하는 데 유용한 Kubernetes 리소스입니다.

