# Kubernetes Namespace 개요

Namespace는 Kubernetes 클러스터 내에서 리소스를 논리적으로 분리하여 관리하는 기능을 제공합니다. 이를 통해 다수의 팀이 동일한 클러스터를 공유하면서도 격리된 환경을 사용할 수 있습니다.

---

## 주요 특징

- **리소스 격리**: 서로 다른 팀 또는 프로젝트 간 리소스를 구분 가능
- **이름 충돌 방지**: 같은 이름의 리소스를 다른 Namespace에서 독립적으로 생성 가능
- **RBAC 적용 가능**: 특정 Namespace에 대한 접근 권한을 세분화할 수 있음

---

## 1. CLI를 사용하여 Namespace 생성

```bash
kubectl create namespace my-namespace
```

### 생성된 Namespace 확인
```bash
kubectl get namespaces
```

### 특정 Namespace의 리소스 조회
```bash
kubectl get pods -n my-namespace
```

### Namespace 삭제
```bash
kubectl delete namespace my-namespace
```

---

## 2. YAML을 사용하여 Namespace 생성

### namespace.yaml 파일 작성
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

### 생성 명령어
```bash
kubectl apply -f namespace.yaml
```

### Namespace 확인
```bash
kubectl get namespaces
```

---

## 3. 특정 Namespace에서 리소스 실행

1. **Namespace 지정하여 Deployment 생성**
   ```bash
   kubectl create deployment my-app --image=nginx -n my-namespace
   ```

2. **kubectl 실행 시 기본 Namespace 변경**
   ```bash
   kubectl config set-context --current --namespace=my-namespace
   ```

3. **특정 네임스페이스에 Pod 배포 (YAML)**

   ### pod-in-namespace.yaml 파일 작성
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
     namespace: my-namespace
   spec:
     containers:
     - name: my-container
       image: nginx
   ```

   ### Pod 배포
   ```bash
   kubectl apply -f pod-in-namespace.yaml
   ```

   ### 네임스페이스 내 Pod 확인
   ```bash
   kubectl get pods -n my-namespace
   ```

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

---

Namespace는 Kubernetes에서 리소스를 그룹화하고 격리하는 중요한 기능을 제공하며, 대규모 클러스터 운영에서 필수적인 역할을 합니다.

