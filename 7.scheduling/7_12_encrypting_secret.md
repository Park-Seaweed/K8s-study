# Kubernetes Encrypting Secret Data at Rest

Kubernetes에서는 **Secret 데이터를 저장 시 암호화(Encryption at Rest)** 기능을 제공하여 보안성을 강화할 수 있습니다. 기본적으로 Secret은 etcd에 **평문(plaintext)** 으로 저장되기 때문에, 보안이 필요한 경우 **암호화 설정**이 필요합니다.

---

## 1. 개념

- **암호화 대상**: Secret, ConfigMap, Token, 그리고 기타 민감한 데이터
- **기본 저장 방식**: etcd에 평문으로 저장됨
- **암호화 활성화**: `EncryptionConfiguration` 설정을 통해 활성화 가능
- **사용 가능한 암호화 방식**:
  - `identity`: 평문 저장 (기본값)
  - `aescbc`: AES-CBC 방식 (권장)
  - `kms`: 외부 Key Management System(KMS) 사용
  - `secretbox`: NaCl/libsodium 기반 암호화

---

## 2. 암호화 활성화 (AES-CBC 사용)

### **암호화 설정 파일 생성**
Kubernetes API 서버가 사용할 **EncryptionConfiguration** 파일을 생성합니다.

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0LWtleS1leGFtcGxlMTIzNDU2Nzg5Cg==  # Base64 인코딩된 키
      - identity: {}
```
**설명**:
- `secrets` 리소스를 암호화 대상으로 지정
- `aescbc`를 사용하여 Secret을 암호화
- `secret` 필드는 **Base64 인코딩된 AES 키**
- `identity`는 **기본적으로 적용되는 평문 저장 방식** (백업용)

### **API 서버에 암호화 설정 적용 (파일 마운트 포함)**

**EncryptionConfiguration 파일을 `/etc/kubernetes/` 디렉터리에 저장**
```sh
sudo mkdir -p /etc/kubernetes/
sudo tee /etc/kubernetes/encryption-config.yaml <<EOF
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: $(echo -n 'your-256-bit-key' | base64)
      - identity: {}
EOF
```

**kube-apiserver 실행 플래그 및 마운트 설정 (`/etc/kubernetes/manifests/kube-apiserver.yaml`)**:
```yaml
spec:
  volumes:
    - name: encryption-config
      hostPath:
        path: /etc/kubernetes/encryption-config.yaml
        type: File
  containers:
  - name: kube-apiserver
    volumeMounts:
      - name: encryption-config
        mountPath: /etc/kubernetes/encryption-config.yaml
        subPath: encryption-config.yaml
    command:
      - "kube-apiserver"
      - "--encryption-provider-config=/etc/kubernetes/encryption-config.yaml"
```
**설명**:
- `/etc/kubernetes/encryption-config.yaml`을 **Volume으로 마운트**
- `--encryption-provider-config` 플래그를 사용해 **암호화 설정 파일을 적용**

### **Kubernetes API 서버 재시작**
```sh
kubectl delete pod -n kube-system -l component=kube-apiserver
```
**설명**:
- API 서버를 재시작하여 새로운 암호화 설정 적용

---

## 3. 기존 Secret 데이터 암호화 적용 (Re-encryption)

API 서버 설정을 변경한 후 기존 Secret 데이터를 암호화하려면 **Secret을 다시 저장**해야 합니다.

### **기존 Secret을 다시 저장하여 암호화 적용**
```sh
kubectl get secrets -o json | kubectl replace -f -
```
**설명**:
- 기존 Secret을 가져와서 다시 저장하면 새로운 암호화 설정이 적용됨

---

## 4. Secret 암호화 여부 확인

### **etcd에서 Secret 데이터 확인**
암호화 적용 여부를 확인하려면 **etcd에 저장된 데이터**를 직접 조회해야 합니다.

```sh
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/secret1 | hexdump -C
```
**설명**:
- 암호화 적용 전 → **평문**(Base64로 인코딩된 Secret 값이 직접 노출됨)
- 암호화 적용 후 → **암호화된 값이 저장됨** (해독 불가능한 형식)

---

## 5. Key Rotation (키 교체) 방법
보안 강화를 위해 **주기적으로 암호화 키를 교체**하는 것이 좋습니다.

### **새로운 암호화 키 추가**
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key2
              secret: Zm9vYmFyYmF6MTIzNDU2Cg==  # 새로운 Base64 인코딩된 키
            - name: key1
              secret: c2VjcmV0LWtleS1leGFtcGxlMTIzNDU2Nzg5Cg==  # 기존 키
      - identity: {}
```
**설명**:
- `key2`를 추가하고 `key1`을 유지 (새로운 키가 최우선 사용됨)

### **기존 Secret 재저장하여 새 키로 암호화 적용**
```sh
kubectl get secrets -o json | kubectl replace -f -
```
**설명**:
- Secret을 다시 저장하여 **새로운 키로 암호화**

### **이전 키 제거**
모든 Secret을 새 키로 암호화한 후 `key1`을 제거합니다.

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key2
              secret: Zm9vYmFyYmF6MTIzNDU2Cg==  # 새로운 키만 유지
      - identity: {}
```
**설명**:
- `key1`을 제거하여 **불필요한 이전 키 삭제**

---

## 6. 참고 자료
- [Kubernetes 공식 문서 - Encrypting Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
- [Kubernetes 공식 문서 - API 서버 보안](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

