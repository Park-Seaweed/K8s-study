# ETCD 개요

etcd는 분산 키-값 저장소로, 주로 쿠버네티스(Kubernetes)에서 클러스터 상태를 저장하는 데 사용됩니다. 고가용성(HA) 및 데이터 일관성을 보장하며, Raft 합의 알고리즘을 기반으로 동작합니다.

---

## 주요 특징

- **일관성 보장**: Raft 합의 알고리즘 사용
- **고가용성**: 여러 노드로 구성 가능
- **빠른 성능**: 낮은 지연시간으로 데이터 접근 가능
- **gRPC API 지원**: 다양한 언어에서 클라이언트 구현 가능

---

## 기본 명령어

### etcdctl 환경 설정
```bash
export ETCDCTL_API=3  # API v3 사용 설정
```

### 클러스터 상태 확인
```bash
etcdctl --endpoints=<ETCD_ENDPOINT> endpoint status --write-out=table
```

### 키-값 저장 및 조회
```bash
etcdctl put mykey "Hello ETCD"
etcdctl get mykey
```

### 키 삭제
```bash
etcdctl del mykey
```

### 모든 키 나열
```bash
etcdctl get "" --prefix
```

### 리더 노드 확인
```bash
etcdctl endpoint leader
```

---

## 백업 및 복원

### etcd 데이터 백업
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
```

### 백업 확인
```bash
etcdctl snapshot status backup.db
```

### 백업 복원
```bash
etcdutl --data-dir <data-dir-location> snapshot restore snapshot.db
```

---

## 참고 자료

- [ETCD 공식 문서](https://etcd.io/docs/)
- [Kubernetes ETCD 관련 문서](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

etcd는 Kubernetes의 핵심 구성 요소 중 하나로, 안정적인 클러스터 운영을 위해 올바른 관리 및 백업 전략이 필요합니다.

