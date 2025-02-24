# Configuring Kubernetes Scheduler Profiles

Kubernetes에서 **Scheduler Profiles**을 구성하면 특정 스케줄링 정책을 정의하고, 다양한 워크로드에 대해 서로 다른 스케줄링 전략을 사용할 수 있습니다. 이는 클러스터 내에서 다양한 애플리케이션 유형을 보다 세밀하게 조정하는 데 유용합니다.

---

## 1. Scheduler Profiles 개념

- **다양한 스케줄링 전략 지원**: 각 프로파일마다 다른 스케줄링 정책을 적용 가능
- **여러 개의 스케줄러 프로파일 설정 가능**: 특정 Pod이 어떤 프로파일을 사용할지 선택 가능
- **스케줄링 로직을 사용자 맞춤형으로 조정 가능**
- **Preemption, Score, Filter, Bind 등의 다양한 플러그인 조합 가능**

---

## 2. Scheduler Profiles 기본 설정

Kubernetes에서 여러 개의 스케줄링 프로파일을 설정하려면 `KubeSchedulerConfiguration`을 사용합니다.

### **예제: 다중 스케줄러 프로파일 설정**

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
  pluginConfig:
  - name: DefaultPreemption
    args:
      minCandidateNodesPercentage: 10
  - name: NodeResourcesFit
    args:
      ignoredResources:
      - "nvidia.com/gpu"

- schedulerName: high-priority-scheduler
  pluginConfig:
  - name: DefaultPreemption
    args:
      minCandidateNodesPercentage: 50
  - name: NodeResourcesFit
    args:
      ignoredResources:
      - "memory"
```

### **설명**
- **기본 스케줄러 (`default-scheduler`)**: GPU 리소스를 무시하도록 설정
- **우선순위가 높은 스케줄러 (`high-priority-scheduler`)**: 메모리 리소스를 무시하고, 사전 점유(preemption) 설정을 더 높게 조정

이러한 설정을 적용하면 Pod이 특정 스케줄러 프로파일을 선택하여 맞춤형 스케줄링이 가능해집니다.

---

## 3. Pod에 특정 Scheduler Profile 적용하기

Pod의 `schedulerName` 필드를 사용하여 특정 스케줄링 프로파일을 지정할 수 있습니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority-pod
spec:
  schedulerName: high-priority-scheduler  # 지정된 프로파일 사용
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```

이렇게 하면 `high-priority-scheduler` 프로파일이 적용된 Pod만 해당 정책을 사용하게 됩니다.

---

## 4. Scheduler Plugins 설정

Scheduler Profiles에서는 `plugins` 필드를 사용하여 특정 플러그인을 활성화하거나 비활성화할 수 있습니다.

### **예제: 특정 플러그인 활성화/비활성화**
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: custom-scheduler
  plugins:
    score:
      disabled:
        - name: NodeResourcesLeastAllocated  # 특정 플러그인 비활성화
      enabled:
        - name: NodeResourcesMostAllocated  # 특정 플러그인 활성화
```

### **설명**
- `score.disabled`: `NodeResourcesLeastAllocated` 플러그인을 비활성화
- `score.enabled`: `NodeResourcesMostAllocated` 플러그인을 활성화하여 더 많은 리소스를 사용하도록 스케줄링

이렇게 하면 특정 워크로드에 맞는 스코어링 전략을 적용할 수 있습니다.

---

## 5. 주요 플러그인 개요

스케줄러 프로파일에서는 여러 개의 플러그인을 활용하여 Pod 스케줄링을 조정할 수 있습니다.

| 플러그인 이름 | 기능 |
|--------------|------|
| **QueueSort** | Pod의 스케줄링 순서를 결정 |
| **PreFilter** | Pod이 특정 노드에 적합한지 미리 확인 |
| **Filter** | 노드에서 실행 가능한 Pod인지 확인 |
| **Score** | 노드를 점수화하여 최적의 배치 결정 |
| **Reserve** | 스케줄링된 Pod을 노드에 임시 예약 |
| **Permit** | 특정 조건을 만족하는 경우에만 실행 허용 |
| **Bind** | 최종적으로 Pod을 선택된 노드에 할당 |

---

## 6. Scheduler Profiles 적용 및 관리

### **6.1 현재 스케줄러 설정 확인**
```sh
kubectl get pods -o=jsonpath='{.spec.schedulerName}'
```

### **6.2 스케줄러 로그 확인**
```sh
kubectl logs -n kube-system kube-scheduler
```

### **6.3 특정 프로파일 사용 Pod 확인**
```sh
kubectl get pods --field-selector spec.schedulerName=high-priority-scheduler
```

### **6.4 새로운 Scheduler Profiles 적용**
변경 사항을 적용하려면 스케줄러를 다시 시작해야 합니다.
```sh
kubectl delete pod -n kube-system kube-scheduler
```

---

## 7. 참고 자료
- [Kubernetes 공식 문서 - Scheduler Configuration](https://kubernetes.io/docs/reference/scheduling/config/)

Scheduler Profiles을 활용하면 특정 워크로드별로 최적화된 스케줄링 정책을 구성할 수 있습니다.

