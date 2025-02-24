# Kubernetes ReplicaSet 생성 방법

Kubernetes에서 **ReplicaSet**은 특정 수의 동일한 Pod가 항상 실행되도록 보장하는 역할을 합니다. ReplicaSet을 생성하는 방법에는 **CLI(Command Line Interface) 사용**과 **YAML 파일 정의 후 적용** 두 가지 방식이 있습니다.

---

## 1. CLI를 사용하여 ReplicaSet 생성

```bash
kubectl create rs my-replicaset --image=nginx --replicas=3
```

### 설명
- `kubectl create rs` : ReplicaSet 생성 명령어
- `my-replicaset` : ReplicaSet의 이름 지정
- `--image=nginx` : 사용할 컨테이너 이미지
- `--replicas=3` : Pod 개수 지정 (기본값: 1)

ReplicaSet이 생성되었는지 확인하려면:
```bash
kubectl get rs
```

Pod 목록 확인:
```bash
kubectl get pods
```

삭제하려면:
```bash
kubectl delete rs my-replicaset
```

---

## 2. YAML 파일을 사용하여 ReplicaSet 생성

### replicaset.yaml 파일 작성
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
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

### YAML을 사용한 ReplicaSet 생성
```bash
kubectl apply -f replicaset.yaml
```

### 생성된 ReplicaSet 확인
```bash
kubectl get rs
```

### Pod 목록 확인
```bash
kubectl get pods
```

### ReplicaSet 삭제
```bash
kubectl delete -f replicaset.yaml
```

---

## CLI vs YAML 방식 비교

| 방법 | 특징 |
|------|------|
| **CLI** | 빠르게 ReplicaSet 생성 가능, 간단한 테스트에 유용 |
| **YAML** | 선언적 방식, 버전 관리 가능, 재사용 가능 |

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

---

ReplicaSet은 Kubernetes에서 특정 수의 Pod가 항상 실행되도록 유지하는 데 중요한 역할을 하며, 일반적으로 Deployment에서 내부적으로 사용됩니다.

