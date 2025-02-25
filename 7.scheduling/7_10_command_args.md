# Kubernetes Command 및 Args 설정

Kubernetes에서 Pod의 컨테이너는 `command`(ENTRYPOINT)와 `args`(CMD)를 통해 실행됩니다. 이 문서에서는 `command`와 `args`의 개념 및 사용 방법을 설명합니다.

---

## 1. Command 및 Args 개념

- **`command`**: 컨테이너가 시작될 때 실행할 기본 명령어 (ENTRYPOINT 역할)
- **`args`**: `command`에서 실행될 추가 인자 (CMD 역할)
- `command`가 설정되지 않은 경우, 기본적으로 컨테이너의 `ENTRYPOINT` 사용
- `args`가 설정되지 않은 경우, 컨테이너의 `CMD`를 사용

`command`와 `args`를 함께 사용하면 컨테이너의 실행 방식을 세밀하게 제어할 수 있음

---

## 2. 기본 사용 방법

### **기본 `command` 및 `args` 설정 예제**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-example
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["echo"]
    args: ["Hello, Kubernetes!"]
```
**설명**:
- `command: ["echo"]` → 컨테이너 실행 시 `echo` 명령어 실행
- `args: ["Hello, Kubernetes!"]` → `echo` 명령어에 인자로 "Hello, Kubernetes!" 전달
- 결과적으로 **컨테이너 실행 시 `echo Hello, Kubernetes!` 실행됨**

---

## 3. `command`와 `args` 조합 예제

### **`command`만 설정한 경우**
```yaml
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sleep", "3600"]
```
**설명**:
- `command`만 지정된 경우, `sleep 3600` 실행 (CMD가 무시됨)

### **`args`만 설정한 경우**
```yaml
spec:
  containers:
  - name: my-container
    image: busybox
    args: ["3600"]
```
**설명**:
- `command`가 없으므로 기본 `ENTRYPOINT`(busybox의 `sleep`) 사용
- `args: ["3600"]`이므로 최종적으로 `sleep 3600` 실행됨

### **`command`와 `args`를 함께 사용하는 경우**
```yaml
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sleep"]
    args: ["600"]
```
**설명**:
- `command: ["sleep"]`이므로 `sleep` 실행
- `args: ["600"]`이므로 최종적으로 `sleep 600` 실행됨

---

## 4. 기본 엔트리포인트(ENTRYPOINT) 덮어쓰기

컨테이너 이미지의 `ENTRYPOINT`를 무시하고 새로운 명령을 실행하려면 `command`를 사용하면 됩니다.

### **ENTRYPOINT 덮어쓰기 예제**
```yaml
spec:
  containers:
  - name: my-container
    image: nginx
    command: ["nginx", "-g", "daemon off;"]
```
**설명**:
- 기본적으로 Nginx 컨테이너는 `nginx -g daemon off;`로 실행됨
- `command`를 지정하면 컨테이너의 기본 ENTRYPOINT를 무시하고 설정된 명령을 실행함

---

## 5. 다중 명령 실행 (`sh`, `bash` 활용)

하나의 컨테이너에서 여러 개의 명령을 실행하려면 `sh` 또는 `bash`를 사용해야 합니다.

### **다중 명령 실행 예제**
```yaml
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c"]
    args: ["echo Hello; sleep 30; echo Done"]
```
**설명**:
- `sh -c`를 사용하면 여러 개의 명령을 실행할 수 있음
- `"echo Hello; sleep 30; echo Done"` 실행됨

---

## 6. `command` 및 `args` 관련 주요 명령어

### **실행 중인 Pod의 `command` 및 `args` 확인**
```sh
kubectl get pod <pod-name> -o=jsonpath='{.spec.containers[*].command}'
kubectl get pod <pod-name> -o=jsonpath='{.spec.containers[*].args}'
```
**설명**:
- 실행 중인 Pod의 `command`와 `args`를 JSON Path로 출력

---

## 7. 참고 자료
- [Kubernetes 공식 문서 - Command & Args](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
- [Kubernetes 공식 문서 - Pod](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Kubernetes 공식 문서 - Container Lifecycle](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)

