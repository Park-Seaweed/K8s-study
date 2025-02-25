# Kubernetes 멀티 컨테이너 및 Init 컨테이너 사용 방법

Kubernetes에서 하나의 Pod 내에서 여러 개의 컨테이너를 실행하거나, 특정 컨테이너를 먼저 실행하도록 설정하는 방법을 설명합니다.

---

## 1. 멀티 컨테이너 (Multi-container) 개념

Kubernetes에서 **Pod은 하나 이상의 컨테이너**를 포함할 수 있습니다. 다중 컨테이너를 사용하면 **컨테이너 간 협업**을 통해 보다 강력한 애플리케이션 구성이 가능합니다.

### **멀티 컨테이너 사용 목적**
- **사이드카 패턴**: 주요 컨테이너를 보조하는 컨테이너 추가 (예: 로그 수집, 프록시, 데이터 변환)
- **로깅 및 모니터링**: 애플리케이션 로그를 수집하는 별도의 컨테이너 실행
- **프록시 서비스**: 트래픽을 중간에서 조정하는 컨테이너 실행

### **멀티 컨테이너 예제**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: main-app
    image: nginx
    ports:
      - containerPort: 80
  - name: log-collector
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
```
**설명**:
- `main-app`: Nginx 웹 서버 실행
- `log-collector`: Nginx의 로그 파일을 지속적으로 읽는 컨테이너
- 같은 네트워크 공간을 공유하며, `/var/log/nginx/access.log`를 통해 로그를 추적 가능

---

## 2. Init 컨테이너 (Init Containers)

Init 컨테이너는 **애플리케이션 컨테이너가 실행되기 전에 반드시 실행되어야 하는 컨테이너**입니다. 일반적으로 애플리케이션 컨테이너가 실행되기 전에 필요한 **설정, 초기화, 데이터 준비**를 수행하는 데 사용됩니다.

### **Init 컨테이너 사용 목적**
- **초기 데이터 로딩**: 메인 컨테이너 실행 전에 필요한 데이터를 다운로드 및 설정
- **환경 준비**: 애플리케이션이 정상적으로 동작할 수 있도록 종속성을 확인
- **보안 설정**: 애플리케이션이 실행되기 전에 필수 보안 검사를 수행

### **Init 컨테이너 예제**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ["sh", "-c", "echo 'Initializing database...'; sleep 5"]
  containers:
  - name: main-app
    image: nginx
    ports:
      - containerPort: 80
```
**설명**:
- `init-db`: **메인 컨테이너 실행 전에 실행되는 Init 컨테이너**
- `command`: 데이터베이스 초기화 시뮬레이션 (`sleep 5` 이후 종료)
- `main-app`: Init 컨테이너가 완료된 후 실행되는 Nginx 애플리케이션

---

## 3. 멀티 컨테이너 vs Init 컨테이너 비교

| 항목 | 멀티 컨테이너 | Init 컨테이너 |
|------|-------------|---------------|
| 실행 방식 | 동시에 실행됨 | 순차적으로 실행됨 |
| 목적 | 메인 컨테이너를 보조 | 메인 컨테이너 실행 전 초기화 수행 |
| 예제 | 로그 수집, 프록시 | 환경 변수 설정, 데이터 준비 |
| 컨테이너 상태 | 지속적으로 실행됨 | 실행 후 종료됨 |

---

## 4. Init 컨테이너의 동작 방식

### **여러 개의 Init 컨테이너 사용 예제**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-init-container-pod
spec:
  initContainers:
  - name: init-first
    image: busybox
    command: ["sh", "-c", "echo 'First Init Container'; sleep 5"]
  - name: init-second
    image: busybox
    command: ["sh", "-c", "echo 'Second Init Container'; sleep 5"]
  containers:
  - name: main-app
    image: nginx
    ports:
      - containerPort: 80
```
**설명**:
- `init-first` → `init-second` → `main-app` 순서로 실행됨
- 모든 Init 컨테이너가 성공적으로 완료되어야 `main-app`이 실행됨

---

## 5. 멀티 컨테이너 및 Init 컨테이너 상태 확인 명령어

### **Pod 상태 확인**
```sh
kubectl get pod <pod-name>
```

### **Init 컨테이너 상태 확인**
```sh
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses}'
```

### **모든 컨테이너 로그 확인**
```sh
kubectl logs <pod-name> --all-containers
```

### **특정 컨테이너 로그 확인**
```sh
kubectl logs <pod-name> -c <container-name>
```

---

## 6. 참고 자료
- [Kubernetes 공식 문서 - Multi-container Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)
- [Kubernetes 공식 문서 - Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

