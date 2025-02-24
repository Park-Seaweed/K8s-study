# Kubernetes 리소스 제한 (Resource Limits)

Kubernetes에서는 Pod 및 컨테이너의 리소스 사용량을 관리하기 위해 **리소스 요청(Requests)** 및 **리소스 제한(Limits)** 을 설정할 수 있습니다. 이를 통해 클러스터 내 리소스를 효율적으로 할당하고, 특정 컨테이너가 과도한 리소스를 소비하지 않도록 제어할 수 있습니다.

---

## 1. 리소스 요청(Requests)과 제한(Limits)의 차이

| 설정 | 설명 |
|------|------|
| **Requests** | 컨테이너가 실행될 때 최소한으로 보장받는 CPU 및 메모리 |
| **Limits** | 컨테이너가 사용할 수 있는 최대 CPU 및 메모리 |

- `Requests`는 **스케줄링 결정**에 영향을 줍니다. (즉, 해당 리소스를 사용할 수 있는 노드에 배치됨)
- `Limits`는 컨테이너가 설정된 값을 초과하여 사용할 수 없도록 **제한**합니다.

---

## 2. 리소스 제한 설정 (YAML 예제)

아래 예제는 특정 Pod의 컨테이너에 CPU 및 메모리 제한을 설정하는 방법을 보여줍니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limited-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        cpu: "250m"  # 0.25 CPU 코어
        memory: "64Mi"  # 64MB 메모리 요청
      limits:
        cpu: "500m"  # 최대 0.5 CPU 코어 사용 가능
        memory: "128Mi"  # 최대 128MB 메모리 사용 가능
```

### 설명
- `requests.cpu: "250m"` → 컨테이너는 최소 **0.25 CPU 코어**를 보장받음
- `requests.memory: "64Mi"` → 컨테이너는 최소 **64Mi 메모리**를 보장받음
- `limits.cpu: "500m"` → 컨테이너는 **최대 0.5 CPU 코어**까지만 사용 가능
- `limits.memory: "128Mi"` → 컨테이너는 **최대 128Mi 메모리**까지만 사용 가능

---

## 3. 리소스 제한 확인 명령어

### Pod의 리소스 설정 확인
```bash
kubectl get pod resource-limited-pod -o yaml | grep resources -A 6
```

### Node의 전체 리소스 사용량 확인
```bash
kubectl describe node <노드명>
```

### 특정 Namespace의 리소스 사용량 조회
```bash
kubectl top pod -n <namespace>
```

---

## 4. 리소스 제한이 적용되지 않을 경우

- **LimitRange 설정이 필요한 경우**: Namespace에서 기본적으로 적용할 `LimitRange`를 설정해야 할 수도 있음
- **노드 리소스 부족**: 설정한 `requests` 값보다 노드의 가용 리소스가 부족한 경우, Pod가 `Pending` 상태로 남을 수 있음
- **OOMKilled 발생**: 컨테이너가 `limits.memory`를 초과하면 OOMKilled(Out of Memory) 오류가 발생하며 자동 종료됨

---

## 5. LimitRange 설정 (Namespace 전체에 기본 제한 적용)

Namespace 내의 모든 Pod에 기본적으로 적용할 수 있도록 `LimitRange`를 설정할 수 있습니다.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: my-namespace
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "250m"
      memory: "128Mi"
    type: Container
```

### 적용 방법
```bash
kubectl apply -f limitrange.yaml
```

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

리소스 제한을 적절히 설정하면 클러스터 내에서 안정적인 애플리케이션 운영이 가능하며, 리소스 오버유스를 방지할 수 있습니다.

