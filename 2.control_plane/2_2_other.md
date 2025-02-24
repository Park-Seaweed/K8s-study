# Kubernetes Control Plane Components

Kubernetes의 컨트롤 플레인은 클러스터의 상태를 관리하고, 워커 노드에서 애플리케이션이 정상적으로 실행되도록 조정하는 역할을 합니다. 주요 컴포넌트로는 **kube-apiserver**, **kube-controller-manager**, **kube-scheduler**가 있습니다.

---

## kube-apiserver

kube-apiserver는 Kubernetes 클러스터의 중심 역할을 하는 API 서버로, 모든 클러스터 요청을 처리하는 게이트웨이입니다. 클러스터 내부 및 외부의 모든 요청을 받아들이고, 인증 및 인가를 수행하며, etcd와 연동하여 클러스터 상태를 저장하고 관리합니다.

### 주요 역할
- **API 요청 처리**: kubectl, 컨트롤러, 스케줄러 등에서 오는 요청을 처리
- **인증 및 인가**: 클라이언트의 요청을 검증하고 RBAC(Role-Based Access Control) 정책을 적용
- **etcd와 연동**: 클러스터 상태 정보를 저장하고 조회
- **컨트롤러 및 노드와 통신**: kube-controller-manager, kube-scheduler 및 노드와 상태 정보를 교환

---

## kube-controller-manager

kube-controller-manager는 Kubernetes의 다양한 컨트롤러들을 관리하는 역할을 합니다. 컨트롤러는 클러스터의 상태를 지속적으로 모니터링하고, 원하는 상태와 실제 상태가 일치하도록 자동으로 조정하는 역할을 합니다.

### 주요 역할
- **Node Controller**: 노드의 상태를 감시하고, 장애가 발생한 노드를 관리
- **Replication Controller**: Pod 개수를 원하는 개수로 유지
- **Service Account & Token Controller**: 서비스 계정과 인증 토큰을 생성 및 관리
- **Endpoint Controller**: 서비스와 관련된 엔드포인트 정보를 관리

---

## kube-scheduler

kube-scheduler는 클러스터 내에서 실행될 Pod의 적절한 배치(스케줄링)를 담당하는 컴포넌트입니다. 노드의 상태와 리소스 가용성을 고려하여, 최적의 노드에 새로운 Pod를 배정합니다.

### 주요 역할
- **스케줄링 결정**: 새로운 Pod가 실행될 적절한 노드를 선택
- **리소스 고려**: CPU, 메모리, 네트워크 등의 리소스를 고려하여 배치
- **정책 기반 스케줄링**: affinity, taint & toleration 등의 규칙을 적용하여 스케줄링 최적화

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/overview/components/)

---

Kubernetes 컨트롤 플레인의 핵심 컴포넌트인 kube-apiserver, kube-controller-manager, kube-scheduler는 클러스터의 안정성과 자동화를 유지하는 중요한 역할을 수행합니다.
