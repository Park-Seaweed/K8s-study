# Kubernetes 환경변수 사용 방법

Kubernetes에서 환경변수를 설정하는 방법은 여러 가지가 있으며, 대표적으로 **직접 설정 (env), ConfigMap, Secret**을 이용하는 방식이 있습니다. 이 문서에서는 환경변수 사용 방법을 정리합니다.

---

## 1. 환경변수 직접 설정 (`env`)

Pod의 컨테이너에서 환경변수를 직접 정의할 수 있습니다.

### **예제: `env`를 사용한 환경변수 설정**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-direct-pod
spec:
  containers:
  - name: my-container
    image: busybox
    env:
      - name: APP_ENV
        value: "production"
      - name: DB_HOST
        value: "db.example.com"
```
**설명**:
- `APP_ENV=production` 설정
- `DB_HOST=db.example.com` 설정

---

## 2. ConfigMap을 이용한 환경변수 설정

### **CLI로 ConfigMap 생성**
```sh
kubectl create configmap my-config --from-literal=APP_ENV=production --from-literal=DB_HOST=db.example.com
```
**설명**:
- `APP_ENV=production`, `DB_HOST=db.example.com`을 포함하는 ConfigMap 생성

### **ConfigMap YAML 생성**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default
data:
  APP_ENV: "production"
  DB_HOST: "db.example.com"
```
**설명**: `APP_ENV`와 `DB_HOST` 값을 저장하는 ConfigMap 생성

### **ConfigMap을 환경변수로 사용 (`envFrom`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap-pod
spec:
  containers:
  - name: my-container
    image: busybox
    envFrom:
      - configMapRef:
          name: my-config
```
**설명**:
- `my-config`의 모든 값을 환경변수로 적용

### **ConfigMap에서 특정 키만 환경변수로 사용 (`env`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap-key-pod
spec:
  containers:
  - name: my-container
    image: busybox
    env:
      - name: APP_ENV
        valueFrom:
          configMapKeyRef:
            name: my-config
            key: APP_ENV
```
**설명**:
- `APP_ENV`만 `my-config`에서 가져와 환경변수로 설정

---

## 3. Secret을 이용한 환경변수 설정 (평문 사용)

Secret은 **민감한 데이터 (예: 비밀번호, API 키)** 저장에 사용됩니다.

### **CLI로 Secret 생성**
```sh
kubectl create secret generic my-secret --from-literal=DB_PASSWORD=password
```
**설명**:
- `DB_PASSWORD=password`를 포함하는 Secret 생성

### **Secret YAML 생성**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  DB_PASSWORD: "cGFzc3dvcmQ="  # base64로 인코딩된 값 (password)
```
**설명**:
- `DB_PASSWORD=password`를 **base64**로 인코딩하여 저장

### **Secret을 환경변수로 사용 (`envFrom`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-secret-pod
spec:
  containers:
  - name: my-container
    image: busybox
    envFrom:
      - secretRef:
          name: my-secret
```
**설명**:
- `my-secret`의 모든 키를 환경변수로 사용

### **Secret에서 특정 키만 환경변수로 사용 (`env`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-secret-key-pod
spec:
  containers:
  - name: my-container
    image: busybox
    env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: DB_PASSWORD
```
**설명**:
- `DB_PASSWORD`만 `my-secret`에서 가져와 환경변수로 설정

### **Base64 인코딩 및 디코딩 방법**
Secret의 `data` 필드는 **base64 인코딩된 값**을 저장합니다.

#### **인코딩 (문자열 → base64)**
```sh
echo -n 'password' | base64
```
출력 예시:
```
cGFzc3dvcmQ=
```

#### **디코딩 (base64 → 원래 값)**
```sh
echo 'cGFzc3dvcmQ=' | base64 --decode
```
출력 예시:
```
password
```

---

## 4. 환경변수 설정 방법 비교

| 방법 | 사용 목적 | 예제 |
|------|----------|------|
| `env` | 직접 환경변수 설정 | `env: value: "production"` |
| `envFrom (ConfigMap)` | ConfigMap의 모든 값을 환경변수로 설정 | `envFrom: configMapRef:` |
| `env (ConfigMap)` | ConfigMap의 특정 값을 환경변수로 설정 | `valueFrom: configMapKeyRef:` |
| `envFrom (Secret)` | Secret의 모든 값을 환경변수로 설정 | `envFrom: secretRef:` |
| `env (Secret)` | Secret의 특정 값을 환경변수로 설정 | `valueFrom: secretKeyRef:` |

---

## 5. 참고 자료
- [Kubernetes 공식 문서 - 환경변수 설정](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
- [Kubernetes 공식 문서 - ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes 공식 문서 - Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

