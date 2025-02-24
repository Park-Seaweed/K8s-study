# Kubernetes Pod 생성 방법

Kubernetes에서 Pod는 컨테이너를 실행하는 가장 작은 배포 단위입니다. Pod를 생성하는 방법에는 **CLI(Command Line Interface) 사용**과 **YAML 파일 정의 후 적용** 두 가지 방식이 있습니다.

---

## 1. CLI를 사용하여 Pod 생성

```bash
kubectl run my-pod --image=nginx
```

### 설명
- `kubectl run` : 새로운 Pod를 생성하는 명령어
- `my-pod` : Pod의 이름 지정
- `--image=nginx` : 사용할 컨테이너 이미지

Pod가 생성되었는지 확인하려면:
```bash
kubectl get pods
```

삭제하려면:
```bash
kubectl delete pod my-pod
```

---

## 2. YAML 파일을 사용하여 Pod 생성

### pod.yaml 파일 작성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80
```

### YAML을 사용한 Pod 생성
```bash
kubectl apply -f pod.yaml
```

### 생성된 Pod 확인
```bash
kubectl get pods
```

### Pod 삭제
```bash
kubectl delete -f pod.yaml
```

---

## CLI vs YAML 방식 비교

| 방법 | 특징 |
|------|------|
| **CLI** | 빠르게 Pod 생성 가능, 간단한 테스트에 유용 |
| **YAML** | 선언적 방식, 버전 관리 가능, 재사용 가능 |

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/workloads/pods/)

---

Pod는 Kubernetes에서 가장 기본적인 실행 단위이며, CLI 또는 YAML을 활용해 쉽게 배포 및 관리할 수 있습니다.

