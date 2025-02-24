# Kubernetes Multiple Scheduler

Kubernetes에서는 기본적으로 **kube-scheduler**가 Pod을 스케줄링하지만, **사용자 정의 스케줄러(Custom Scheduler)** 를 추가하여 여러 개의 스케줄러를 사용할 수 있습니다. 이를 통해 다양한 배포 전략을 적용하거나 특정 워크로드에 맞는 맞춤형 스케줄링 정책을 활용할 수 있습니다.

---

## 1. Multiple Scheduler 개념

- **기본 스케줄러(kube-scheduler)**: Kubernetes에서 기본 제공하는 스케줄러
- **사용자 정의 스케줄러(Custom Scheduler)**: 특정 요구 사항에 맞게 설정한 별도의 스케줄러
- **스케줄러 지정 방법**: Pod의 `schedulerName` 필드를 이용하여 특정 스케줄러 사용 가능

---

## 2. Multiple Scheduler 사용 사례

| 사용 사례 | 설명 |
|-----------|------|
| **고성능 작업 분리** | 특정 노드에 CPU/GPU 작업을 할당하는 맞춤형 스케줄러 사용 |
| **워크로드 분산** | 특정 네임스페이스나 태그를 기반으로 Pod을 스케줄링 |
| **특정 노드에 강제 할당** | 특정 노드에만 Pod을 배치하는 커스텀 로직 적용 |
| **Node Affinity와 결합** | 사용자 정의 규칙과 Kubernetes의 기본 기능을 혼합 |

---

## 3. Custom Scheduler 배포

### **3.1 Custom Scheduler 실행**
기본 kube-scheduler와 충돌하지 않도록, 새로운 스케줄러를 배포합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-scheduler
  namespace: kube-system
spec:
  containers:
  - name: scheduler
    image: k8s.gcr.io/kube-scheduler:v1.27.0
    command:
    - "/usr/local/bin/kube-scheduler"
    - "--config=/etc/kubernetes/custom-scheduler-config.yaml"
    volumeMounts:
    - name: config-volume
      mountPath: /etc/kubernetes
  volumes:
  - name: config-volume
    configMap:
      name: custom-scheduler-config
```

### **3.2 Custom Scheduler 설정 파일 (ConfigMap)**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: custom-scheduler-config
  namespace: kube-system
data:
  custom-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    leaderElection:
      leaderElect: false  # 리더 선출 비활성화
    profiles:
    - schedulerName: custom-scheduler
```

---

## 4. Custom Scheduler를 사용하는 Pod 배포
Pod의 `schedulerName`을 `custom-scheduler`로 지정하여 기본 스케줄러가 아닌 사용자 정의 스케줄러를 사용하도록 설정합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-scheduled-pod
spec:
  schedulerName: custom-scheduler  # 사용자 정의 스케줄러 지정
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```

배포 후 확인:
```sh
kubectl get pods -o wide
```
출력 예시:
```
NAME                  READY   STATUS    NODE          SCHEDULER
custom-scheduled-pod   1/1     Running   node-1        custom-scheduler
```

---

## 5. Custom Scheduler 관리 명령어

### **5.1 실행 중인 스케줄러 확인**
```sh
kubectl get pods -n kube-system | grep scheduler
```

### **5.2 특정 Pod의 스케줄러 확인**
```sh
kubectl get pod custom-scheduled-pod -o=jsonpath='{.spec.schedulerName}'
```

### **5.3 Custom Scheduler 로그 확인**
```sh
kubectl logs custom-scheduler -n kube-system
```

---

## 6. 참고 자료
- [Kubernetes 공식 문서 - Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

Multiple Scheduler를 활용하면 특정 요구 사항에 맞춰 Pod을 배치할 수 있어 클러스터의 효율성을 극대화할 수 있습니다.

