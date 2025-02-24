# Kubernetes Static Pod

Kubernetes에서 **Static Pod**는 **kube-apiserver와 상관없이 특정 노드에서 직접 실행되는 Pod**입니다. 이는 주로 **kubelet에 의해 직접 관리**되며, 클러스터의 일부로 인식되지 않고, `kubectl`을 사용하여 관리할 수 없습니다.

---

## 1. Static Pod의 주요 특징

- **kube-apiserver와 독립적으로 동작**
- **kubelet이 직접 관리**하며, kube-apiserver에 의해 제어되지 않음
- **각 노드의 특정 디렉토리에 있는 YAML 파일을 기반으로 실행됨**
- **Pod가 삭제되면 자동으로 재시작됨**
- **노드 장애 시 복구되지 않음** (클러스터 스케줄러가 관리하지 않음)

---

## 2. Static Pod 사용 사례

| 사용 사례 | 설명 |
|-----------|------|
| **Kubernetes 컨트롤 플레인 구성** | `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` 같은 핵심 컴포넌트 실행 |
| **노드별 필수 애플리케이션 실행** | `Log Collector`, `Monitoring Agent` 등 노드마다 필수적으로 배포해야 하는 서비스 |
| **Bootstraping 및 초기화 작업** | 클러스터 설정 및 부팅 과정에서 기본적인 Pod 배포 |

---

## 3. Static Pod 생성 방법

Static Pod는 kubelet이 특정 디렉토리에 위치한 YAML 파일을 감시하여 실행됩니다. 기본적으로 아래 경로를 사용합니다.

- `/etc/kubernetes/manifests/` (기본값)
- `/etc/kubelet.d/` (사용자 지정 가능)

### **3.1 Static Pod YAML 예제**

아래는 `nginx`를 Static Pod로 배포하는 예제입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
      hostPort: 8080
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: nginx-data
  volumes:
  - name: nginx-data
    hostPath:
      path: /var/nginx
```

### **3.2 Static Pod 배포**
Static Pod의 YAML 파일을 지정된 디렉토리에 저장하면 kubelet이 자동으로 Pod를 실행합니다.

```bash
sudo mv static-nginx.yaml /etc/kubernetes/manifests/
```

- `kubelet`이 해당 디렉토리를 감시하고 자동으로 Pod를 실행함
- 변경 사항이 있을 경우 kubelet이 자동으로 반영

---

## 4. Static Pod 확인 및 관리

### **4.1 실행 중인 Static Pod 확인**
```bash
kubectl get pods -A | grep static-nginx
```

### **4.2 노드에서 직접 확인**
Static Pod는 클러스터의 kube-apiserver가 아니라 개별 노드에서 실행됩니다. 따라서 노드에서 직접 확인해야 합니다.
```bash
crictl ps | grep static-nginx
```

### **4.3 Static Pod 로그 확인**
```bash
kubectl logs -f static-nginx -n kube-system
```

또는 노드에서 직접 실행:
```bash
journalctl -u kubelet | grep static-nginx
```

### **4.4 Static Pod 삭제**
Static Pod를 삭제하려면 해당 YAML 파일을 제거하면 됩니다.
```bash
sudo rm /etc/kubernetes/manifests/static-nginx.yaml
```

kubelet이 자동으로 Pod를 종료합니다.

---

## 5. Static Pod의 제한 사항

- **kube-apiserver에 의해 관리되지 않음** → `kubectl delete pod`로 삭제 불가
- **자동 복구되지 않음** → 노드가 장애 발생하면 클러스터 내 다른 노드에서 다시 실행되지 않음
- **로그 및 상태 관리 어려움** → `kubectl describe`로 모든 정보를 확인할 수 없음

---

## 6. Static Pod vs 일반 Pod 비교

| 항목 | Static Pod | 일반 Pod |
|------|-----------|----------|
| 관리 주체 | kubelet | kube-apiserver (스케줄러) |
| 배포 방식 | 특정 노드의 디렉토리 감시 | `kubectl apply -f` 명령어 사용 |
| 스케일링 | 불가능 (각 노드별 1개) | 가능 (ReplicaSet, Deployment) |
| 클러스터 내 이동 | 불가능 | 가능 (스케줄링됨) |

---

## 7. Static Pod 실전 활용

### **7.1 Kubernetes Control Plane 실행**
Kubernetes의 주요 컨트롤 플레인 컴포넌트들은 Static Pod로 실행됩니다. 예를 들어 `/etc/kubernetes/manifests/` 경로에 다음과 같은 파일이 위치함.
```bash
ls /etc/kubernetes/manifests/
kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml  etcd.yaml
```
이들은 kubelet에 의해 실행되며, 클러스터가 정상적으로 작동하는 데 필수적인 역할을 합니다.

### **7.2 마스터 노드의 kubelet 설정 확인**
kubelet이 Static Pod를 실행하는 디렉토리를 확인하려면 `/var/lib/kubelet/config.yaml`을 확인합니다.
```bash
grep staticPodPath /var/lib/kubelet/config.yaml
```
출력 예시:
```
staticPodPath: "/etc/kubernetes/manifests"
```

---

## 8. 참고 자료
- [Kubernetes 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

Static Pod는 노드 단위의 필수 애플리케이션 배포 및 Kubernetes 컨트롤 플레인 운영에 유용하게 활용됩니다.

