# Kubernetes 스케줄링 개요

Kubernetes 스케줄링은 클러스터 내에서 적절한 노드를 선택하여 Pod를 배치하는 과정입니다. 스케줄러는 리소스 사용률, 제약 조건, 정책 등을 고려하여 최적의 노드를 결정합니다.

---

## 주요 개념

- **스케줄러**: kube-scheduler가 Pod를 적절한 노드에 배치하는 역할 수행
- **스케줄링 정책**: 다양한 조건을 고려하여 Pod를 배치하는 기준
- **노드 셀렉터(Node Selector)**: 특정 레이블을 기반으로 특정 노드에 Pod 배포
- **어피니티(Affinity) 및 안티어피니티(Anti-Affinity)**: 특정 노드 또는 Pod와의 배치를 조정하는 규칙
- **Taints & Tolerations**: 특정 노드에 Pod가 배치되지 않도록 하거나 예외적으로 배치할 수 있도록 설정

---

## 1. 기본 스케줄링 방식

### 자동 스케줄링
기본적으로 kube-scheduler가 자동으로 적절한 노드를 선택하여 Pod를 배포합니다.
```bash
kubectl get pods -o wide
```
Pod가 할당된 노드를 확인할 수 있습니다.

### 특정 노드에 배포 (Node Selector)
Pod을 특정 노드에 배포하려면 `nodeSelector`를 사용합니다.

#### pod-node-selector.yaml 파일 작성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: my-container
    image: nginx
```

배포 실행:
```bash
kubectl apply -f pod-node-selector.yaml
```

---

## 2. 어피니티 및 안티어피니티

보다 정교한 제어가 필요할 경우 `affinity` 및 `antiAffinity` 설정을 활용할 수 있습니다.

### Node Affinity 적용
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: my-container
    image: nginx
```

배포 실행:
```bash
kubectl apply -f affinity-pod.yaml
```

---

## 3. Taints & Tolerations

특정 노드에 Pod가 기본적으로 배치되지 않도록 하거나, 특정 Pod만 배치할 수 있도록 설정할 수 있습니다.

### 노드에 Taint 적용
```bash
kubectl taint nodes node1 key=value:NoSchedule
```

### Pod에서 Toleration 설정
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: my-container
    image: nginx
```

배포 실행:
```bash
kubectl apply -f toleration-pod.yaml
```

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/scheduling-eviction/)

---

Kubernetes 스케줄링은 리소스 효율성을 극대화하고, 특정 요구 사항에 맞게 Pod를 배치할 수 있도록 다양한 기능을 제공합니다.

