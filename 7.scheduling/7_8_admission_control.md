# Kubernetes Admission Control

Kubernetes **Admission Control**은 API 서버가 리소스를 생성, 수정 또는 삭제하는 요청을 처리하기 전에 수행하는 정책 실행 단계입니다. 이를 통해 보안, 검증, 리소스 제한 등을 적용할 수 있습니다.

---

## 1. Admission Controller 개념

- **Validating Admission Controllers**: API 요청을 검증하고 거부할 수 있음
- **Mutating Admission Controllers**: API 요청을 수정할 수 있음 (예: 기본 값 추가)
- API 서버가 리소스를 etcd에 저장하기 전에 실행됨
- 보안, 네트워크 정책, 리소스 할당 등의 기능을 적용 가능

---

## 2. Admission Controller 동작 방식

Admission Controller는 **두 단계로 동작**합니다.

1. **Mutating Admission Controllers** (변경 가능)
   - 요청을 수정하여 기본 값을 추가하거나 보강
   - 예: `MutatingAdmissionWebhook`, `PodPreset`

2. **Validating Admission Controllers** (검증 전용)
   - 요청이 정책을 준수하는지 확인하고 거부할 수 있음
   - 예: `ValidatingAdmissionWebhook`, `PodSecurity`

---

## 3. 활성화된 Admission Controllers 확인

Kubernetes API 서버에서 실행 중인 Admission Controller 목록을 확인하려면:

### **`kube-apiserver` 실행 플래그 확인**
```sh
ps aux | grep kube-apiserver | grep enable-admission-plugins
```
출력 예시:
```sh
--enable-admission-plugins=NamespaceLifecycle,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
```

### **설정 파일에서 확인** (`kube-apiserver.yaml`)
```sh
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep enable-admission-plugins
```

---

## 4. 주요 Admission Controllers 목록

| 이름 | 유형 | 설명 |
|------|------|------|
| `NamespaceLifecycle` | Validating | 삭제된 네임스페이스의 리소스 생성 차단 |
| `NodeRestriction` | Validating | 노드에 대한 특정 API 요청 제한 |
| `PodSecurity` | Validating | Pod 보안 정책 검증 (PSP 대체) |
| `LimitRanger` | Mutating | 요청된 리소스가 네임스페이스 제한을 초과하지 않도록 설정 |
| `ResourceQuota` | Validating | 네임스페이스 별 리소스 사용량 제한 |
| `MutatingAdmissionWebhook` | Mutating | 웹훅을 통해 요청 변환 가능 |
| `ValidatingAdmissionWebhook` | Validating | 웹훅을 통해 요청 검증 가능 |

---

## 5. Admission Webhook 설정 예제

Admission Webhook을 사용하면 외부 검증 로직을 Kubernetes와 연동할 수 있습니다.

### **Webhook을 사용하는 Validating Admission Controller 예제**
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: example-validating-webhook
webhooks:
  - name: validate.example.com
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE", "UPDATE"]
        resources: ["pods"]
    clientConfig:
      service:
        name: example-webhook
        namespace: default
        path: "/validate"
      caBundle: Cg==
```

이 예제에서는 `example-webhook` 서비스가 Pod 생성 요청을 검증합니다.

---

## 6. Admission Controller 활성화 및 비활성화

### **특정 Admission Controller 활성화**
API 서버 실행 옵션에 추가:
```sh
--enable-admission-plugins=NamespaceLifecycle,NodeRestriction,ResourceQuota
```

### **특정 Admission Controller 비활성화**
```sh
--disable-admission-plugins=PodSecurity,LimitRanger
```

### **현재 적용된 설정 확인**
```sh
kubectl api-versions
```

---

## 7. Admission Control을 활용한 보안 및 정책 적용

### **보안 강화**
- `PodSecurity`: Pod에 대한 보안 정책 적용
- `NodeRestriction`: 노드의 불필요한 API 요청 제한

### **리소스 관리**
- `LimitRanger`: 네임스페이스 별 리소스 제한 설정
- `ResourceQuota`: 전체 네임스페이스의 리소스 사용량 제한

### **맞춤형 검증 및 변환**
- `MutatingAdmissionWebhook`: Pod 생성 시 자동 레이블 추가
- `ValidatingAdmissionWebhook`: 특정 컨테이너 이미지를 제한하는 정책 적용

---

## 8. 참고 자료
- [Kubernetes 공식 문서 - Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

Admission Controllers를 적절히 활용하면 **보안, 리소스 관리 및 정책 적용을 자동화**할 수 있습니다.

