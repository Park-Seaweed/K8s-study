# Kubernetes HPA, In-place Resize of Pods, VPA 사용 방법 및 의의

Kubernetes에서는 워크로드의 자원을 동적으로 관리하기 위해 여러 가지 **오토스케일링 및 리소스 조정 기능**을 제공합니다. 이 문서에서는 **HPA(Horizontal Pod Autoscaler), In-place Resize of Pods, VPA(Vertical Pod Autoscaler)** 의 개념과 사용 방법을 설명합니다.

---

## 1. HPA (Horizontal Pod Autoscaler)

### **개념 및 의의**
- HPA는 **Pod의 개수를 자동으로 조정**하는 기능을 제공합니다.
- CPU 사용량, 메모리 사용량, 사용자 정의 메트릭 등을 기반으로 스케일링 가능
- 수평 확장을 통해 **애플리케이션의 가용성을 높이고 부하를 분산**
- `Deployment`, `StatefulSet`, `ReplicaSet` 등에 적용 가능

### **HPA 활성화 및 사용 방법**
#### **HPA를 적용할 Deployment 예제**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```

#### **HPA 생성 및 적용 (CLI 방식)**
```sh
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```
**설명**:
- CPU 사용량이 **50% 이상**이 되면 Pod 수를 증가
- 최소 1개, 최대 10개의 Pod을 유지

#### **HPA YAML 설정 (정적인 방식)**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
**설명**:
- `scaleTargetRef`: `my-app` Deployment에 적용
- `minReplicas`: 최소 1개의 Pod 유지
- `maxReplicas`: 최대 10개의 Pod까지 확장 가능
- CPU 사용률이 50% 이상이면 Pod 수 증가

#### **HPA 상태 확인**
```sh
kubectl get hpa
```

---

## 2. In-place Resize of Pods (Pod 리소스 변경)

### **개념 및 의의**
- 기존 Pod을 삭제하지 않고 **리소스를 즉시 조정**할 수 있는 기능
- CPU 및 메모리 변경 시 **Pod 재시작 없이 동적으로 반영**됨 (단, `resizePolicy` 필요)
- Kubernetes 1.27 이후부터 지원됨

### **In-place Resize 사용 방법**
#### **Pod 리소스 변경 (YAML 예제)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-5
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr-5
    image: nginx
    resizePolicy:
    - resourceName: cpu
      restartPolicy: NotRequired
    - resourceName: memory
      restartPolicy: RestartContainer
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```
**설명**:
- `resizePolicy`를 명시하여 CPU와 메모리 조정 방식을 정의
  - `cpu` → `restartPolicy: NotRequired` → **Pod 재시작 없이 조정 가능**
  - `memory` → `restartPolicy: RestartContainer` → **메모리 변경 시 컨테이너 재시작 필요**

#### **리소스 변경 적용 (CLI)**
```sh
kubectl patch pod qos-demo-5 -n qos-example -p '{"spec":{"containers":[{"name":"qos-demo-ctr-5","resources":{"requests":{"cpu":"1"},"limits":{"cpu":"1"}}}]}}'
```
**설명**:
- CPU를 `700m` → `1`로 변경
- `restartPolicy: NotRequired`이므로 **Pod 재시작 없이 즉시 반영**

```sh
kubectl patch pod qos-demo-5 -n qos-example -p '{"spec":{"containers":[{"name":"qos-demo-ctr-5","resources":{"requests":{"memory":"300Mi"},"limits":{"memory":"300Mi"}}}]}}'
```
📌 **설명**:
- 메모리를 `200Mi` → `300Mi`로 변경
- `restartPolicy: RestartContainer`이므로 **컨테이너가 재시작됨**

---

## 3. VPA (Vertical Pod Autoscaler)

### **개념 및 의의**
- VPA는 **Pod의 CPU 및 메모리 리소스를 동적으로 조정**하는 기능 제공
- HPA가 **Pod 수를 조정**하는 것과 달리, VPA는 **Pod의 리소스 할당량을 변경**
- 클러스터 자원을 효율적으로 사용하고, 과도한 리소스 요청을 방지

### **VPA 활성화 및 사용 방법**
#### **VPA 컨트롤러 설치**
```sh
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
```
**설명**:
- VPA 컨트롤러를 설치하여 사용 가능하도록 설정

#### **VPA 적용 (리소스 자동 조정)**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
```
**설명**:
- `targetRef`: `my-app` Deployment에 VPA 적용
- `updateMode: Auto`: VPA가 **자동으로 리소스를 조정**

#### **VPA 상태 확인**
```sh
kubectl get vpa
```

---

## 4. 참고 자료
- [Kubernetes 공식 문서 - HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Kubernetes 공식 문서 - In-place Resize](https://kubernetes.io/docs/tasks/configure-pod-container/resize-container-resources/)
- [Kubernetes 공식 문서 - VPA](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)