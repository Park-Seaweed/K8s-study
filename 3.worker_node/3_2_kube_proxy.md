# kube-proxy 개요

kube-proxy는 Kubernetes의 네트워크 컴포넌트로, 클러스터 내부에서 서비스 간 통신을 가능하게 하는 역할을 합니다. 각 노드에서 실행되며, 네트워크 규칙을 설정하여 Pod 간의 통신을 관리합니다.

---

## 주요 역할

- **서비스 라우팅**: Kubernetes 서비스의 클러스터 IP를 통해 트래픽을 적절한 Pod로 전달
- **iptables/ipvs 기반 트래픽 관리**: 네트워크 규칙을 적용하여 서비스 트래픽을 효율적으로 처리
- **로드 밸런싱**: 서비스 앞단에서 여러 Pod로 트래픽을 분산하여 부하를 조절
- **외부 트래픽 처리**: NodePort, LoadBalancer 서비스 유형을 사용하여 외부 트래픽을 적절히 전달

---

## 동작 방식

kube-proxy는 다양한 방식으로 네트워크 트래픽을 관리합니다.

1. **iptables 모드**
   - iptables를 사용하여 트래픽을 서비스의 엔드포인트(Pod)로 전달
   - 작은 규모의 클러스터에서 적절한 성능을 발휘

2. **IPVS 모드**
   - Linux 커널의 IPVS를 활용하여 고성능 로드 밸런싱 수행
   - 대규모 클러스터에서 성능 최적화 가능

3. **userspace 모드** (Deprecated)
   - 사용자 공간에서 패킷을 처리하는 방식으로, 현재는 거의 사용되지 않음

---

## kube-proxy vs 기타 네트워크 컴포넌트

| 컴포넌트 | 역할 |
|----------|------|
| **kube-proxy** | 클러스터 네트워크를 설정하고, 서비스 트래픽을 적절한 Pod로 전달 |
| **kubelet** | 노드에서 실행되는 Pod를 관리하고 상태를 유지 |
| **cni 플러그인** | Pod 간 네트워크 연결을 제공 (Calico, Flannel, Cilium 등) |

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/services-networking/service/)

---

kube-proxy는 Kubernetes 서비스 네트워크의 핵심 요소로, 안정적인 트래픽 라우팅을 제공하여 클러스터의 원활한 통신을 보장합니다.

