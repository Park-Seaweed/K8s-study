# Kubernetes DaemonSet

Kubernetes에서 **DaemonSet**은 클러스터 내 **각 노드에서 하나의 Pod를 실행하도록 보장하는** 리소스입니다. 주로 로그 수집기, 모니터링 에이전트, 네트워크 관리 도구와 같은 **노드 레벨의 서비스**를 배포하는 데 사용됩니다.

---

## 1. DaemonSet의 주요 특징

- **각 노드에서 하나의 Pod 실행**: 새로운 노드가 추가되면 자동으로 Pod가 배포됨
- **스케일링 불필요**: 워커 노드 개수에 따라 자동으로 관리됨
- **노드 셀렉터, 토폴로지 선택 가능**
- **노드에 특정한 애플리케이션 배포에 적합**

---

## 2. DaemonSet 사용 사례

| 사용 사례 | 설명 |
|-----------|------|
| **로그 수집** | `Fluentd`, `Filebeat`, `Logstash` 등을 실행하여 노드에서 로그를 수집 |
| **모니터링 에이전트** | `Prometheus Node Exporter`, `Datadog Agent` 등을 각 노드에서 실행 |
| **네트워크 관리** | `CNI 플러그인` (Calico, Flannel 등) 배포 |
| **보안 및 정책 적용** | `Falco`, `Sysdig` 등의 보안 모니터링 도구 실행 |

---

## 3. DaemonSet YAML 예제

아래 예제는 `Fluentd`를 DaemonSet으로 배포하는 방법을 보여줍니다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
  namespace: logging
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          requests:
            cpu: "100m"
            memory: "200Mi"
          limits:
            cpu: "200m"
            memory: "400Mi"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

### 설명
- **`selector.matchLabels`**: DaemonSet이 관리하는 Pod를 정의
- **`spec.template.spec.containers`**: 실행할 컨테이너 (`fluentd` 사용)
- **`resources`**: CPU 및 메모리 리소스 제한 설정
- **`volumeMounts` 및 `hostPath`**: 노드의 `/var/log` 디렉토리를 마운트하여 로그 수집 가능

---

## 4. DaemonSet 관리 명령어

### 4.1 DaemonSet 생성
```bash
kubectl apply -f fluentd-daemonset.yaml
```

### 4.2 실행 중인 DaemonSet 확인
```bash
kubectl get daemonset -n logging
```
출력 예시:
```
NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-daemonset   3         3         3       3            3           <none>          5m
```

### 4.3 특정 DaemonSet의 상세 정보 확인
```bash
kubectl describe daemonset fluentd-daemonset -n logging
```

### 4.4 DaemonSet 삭제
```bash
kubectl delete daemonset fluentd-daemonset -n logging
```

---

## 5. 특정 노드에만 배포하는 DaemonSet 설정

DaemonSet을 특정 노드에서만 실행하려면 `nodeSelector` 또는 `nodeAffinity`를 설정할 수 있습니다.

### **5.1 `nodeSelector` 사용 예제**
```yaml
spec:
  template:
    spec:
      nodeSelector:
        dedicated: logging
```
- 특정 라벨(`dedicated=logging`)이 있는 노드에서만 실행됨

### **5.2 `nodeAffinity` 사용 예제**
```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: role
                    operator: In
                    values:
                      - monitoring
```
- `role=monitoring` 라벨이 있는 노드에서만 실행됨

---

## 6. DaemonSet 업데이트 전략

DaemonSet을 업데이트하는 방법은 `RollingUpdate`와 `OnDelete` 두 가지가 있습니다.

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
```
- **RollingUpdate**: 노드를 하나씩 업데이트하며 새로운 버전의 Pod 배포 (권장)
- **OnDelete**: 기존 Pod가 삭제될 때만 새로운 Pod가 생성됨

---

## 7. 참고 자료
- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

DaemonSet을 올바르게 활용하면 노드별로 실행해야 하는 서비스들을 효율적으로 관리할 수 있습니다.

