# Kubernetes Deployment 업데이트 및 롤아웃 관리

Kubernetes **Deployment**는 애플리케이션 배포 및 관리를 쉽게 하기 위해 사용됩니다. 이 문서에서는 `kubectl set`을 이용한 이미지 업데이트 방법과 `kubectl rollout`을 활용한 배포 관리 방법을 설명합니다.

---

## 1. Deployment 개념

- **이미지 업데이트 (`kubectl set`)**: Deployment에서 실행 중인 컨테이너의 이미지를 변경할 수 있음
- **롤아웃 관리 (`kubectl rollout`)**: Deployment의 배포 상태를 확인하고, 이전 버전으로 롤백 가능
- **배포 전략**: RollingUpdate(기본값) 및 Recreate 방식을 지원

---

## 2. Deployment의 이미지 업데이트 (`kubectl set`)

`kubectl set image` 명령어를 사용하여 Deployment의 컨테이너 이미지를 업데이트할 수 있습니다.

### **✅ 기본 명령어**
```sh
kubectl set image deployment/<Deployment명> <컨테이너명>=<새로운 이미지>
```

### **✅ 예제: Deployment의 컨테이너 이미지 변경**
```sh
kubectl set image deployment/my-app my-container=my-app:v2
```
📌 **설명**:
- `my-app` Deployment 내 `my-container`의 이미지를 `my-app:v2`로 변경
- 자동으로 새로운 Pod이 생성되고 기존 Pod이 순차적으로 교체됨

---

## 3. Deployment 롤아웃 관리 (`kubectl rollout`)

`kubectl rollout`을 사용하면 Deployment의 배포 상태를 확인하고, 이전 버전으로 되돌릴 수 있습니다.

### **✅ 현재 롤아웃 상태 확인**
```sh
kubectl rollout status deployment/<Deployment명>
```

**예제:**
```sh
kubectl rollout status deployment/my-app
```
출력 예시:
```
deployment "my-app" successfully rolled out
```

### **✅ 이전 버전으로 롤백 (`rollback`)**
```sh
kubectl rollout undo deployment/<Deployment명>
```

**예제:**
```sh
kubectl rollout undo deployment/my-app
```
📌 **설명**: 이전에 실행되던 안정적인 버전으로 되돌립니다.

### **✅ 특정 버전으로 롤백**
1. **배포 히스토리 확인**
```sh
kubectl rollout history deployment/my-app
```
출력 예시:
```
deployments "my-app"
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Updated image to my-app:v2
```

2. **특정 버전으로 롤백**
```sh
kubectl rollout undo deployment/my-app --to-revision=1
```
📌 **설명**: `REVISION 1` 버전으로 되돌리기

---

## 4. 주요 배포 전략 (RollingUpdate vs Recreate)

### **✅ RollingUpdate (기본값)**
- 새 버전의 Pod을 **점진적으로 배포**하고, 기존 Pod을 하나씩 종료함
- 다운타임 없이 배포 가능

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```
📌 **설명**:
- `maxSurge: 25%` → 새 Pod을 최대 25% 추가 생성 가능
- `maxUnavailable: 25%` → 기존 Pod의 최대 25% 삭제 가능

### **✅ Recreate 전략**
- 기존 Pod을 **모두 종료한 후** 새로운 버전을 배포
- **다운타임이 발생**할 수 있음

```yaml
strategy:
  type: Recreate
```
📌 **사용 예시**: 데이터베이스와 같이 **동시에 여러 개의 인스턴스를 실행할 수 없는 경우** 사용

---

## 5. Deployment 모니터링 및 관리

### **✅ 변경 사항 확인 (`diff`)**
```sh
kubectl diff -f deployment.yaml
```
📌 **설명**: 현재 클러스터의 설정과 `deployment.yaml` 파일의 차이를 출력

### **✅ 실시간 배포 로그 확인**
```sh
kubectl logs -f deployment/my-app
```
📌 **설명**: 현재 실행 중인 Pod의 로그를 실시간 확인 가능

### **✅ 배포된 Pod 확인**
```sh
kubectl get pods -l app=my-app
```
📌 **설명**: `app=my-app` 레이블이 적용된 모든 Pod을 확인

---

## 6. Deployment 관련 주요 명령어 요약

| 명령어 | 설명 |
|--------|------|
| `kubectl set image deployment/<Deployment명> <컨테이너명>=<새 이미지>` | Deployment의 컨테이너 이미지 업데이트 |
| `kubectl rollout status deployment/<Deployment명>` | 현재 롤아웃 진행 상태 확인 |
| `kubectl rollout undo deployment/<Deployment명>` | 이전 버전으로 롤백 |
| `kubectl rollout history deployment/<Deployment명>` | 배포 히스토리 확인 |
| `kubectl rollout undo deployment/<Deployment명> --to-revision=<번호>` | 특정 버전으로 롤백 |

---

## 7. 참고 자료
- [Kubernetes 공식 문서 - Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes 공식 문서 - Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Kubernetes 공식 문서 - kubectl rollout](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout)

