# Kubernetes 주요 명령어 모음

쿠버네티스(Kubernetes)에서 자주 사용하는 명령어들을 정리하였습니다. 기본적인 클러스터 관리부터 리소스 조회, 배포, 디버깅까지 포함됩니다.

---

## 1. 클러스터 및 네임스페이스 관리

### 클러스터 상태 확인
```bash
kubectl cluster-info
kubectl get nodes
```

### 네임스페이스 조회 및 생성
```bash
kubectl get namespaces
kubectl create namespace my-namespace
kubectl delete namespace my-namespace
```

### 현재 네임스페이스 설정
```bash
kubectl config set-context --current --namespace=my-namespace
```

---

## 2. 리소스 조회

### 모든 리소스 조회
```bash
kubectl get all
```

### 특정 리소스 조회
```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get secrets
```

### 특정 리소스 상세 조회
```bash
kubectl describe pod my-pod
kubectl describe deployment my-deployment
kubectl describe service my-service
```

---

## 3. 배포 및 관리

### Pod 생성 (단일 컨테이너)
```bash
kubectl run my-pod --image=nginx --restart=Never
```

### Deployment 생성
```bash
kubectl create deployment my-app --image=nginx --replicas=3
```

### Service 생성 (ClusterIP, NodePort, LoadBalancer)
```bash
kubectl expose deployment my-app --port=80 --target-port=8080 --type=ClusterIP
kubectl expose deployment my-app --port=80 --target-port=8080 --type=NodePort
kubectl expose deployment my-app --port=80 --target-port=8080 --type=LoadBalancer
```

### YAML을 통한 배포
```bash
kubectl apply -f my-resource.yaml
```

### 리소스 삭제
```bash
kubectl delete pod my-pod
kubectl delete deployment my-app
kubectl delete service my-service
kubectl delete -f my-resource.yaml
```

---

## 4. 레이블 및 셀렉터 사용

### 레이블 추가
```bash
kubectl label pod my-pod app=my-app
```

### 레이블 제거
```bash
kubectl label pod my-pod app-
```

### 특정 레이블을 가진 리소스 조회
```bash
kubectl get pods -l app=my-app
```

### 레이블 변경 확인
```bash
kubectl get pods --show-labels
```

---

## 5. --dry-run 옵션 사용

### 명령어 실행 전 미리보기
```bash
kubectl create deployment my-app --image=nginx --dry-run=client
```

### YAML 형식으로 출력
```bash
kubectl create deployment my-app --image=nginx --dry-run=client -o yaml
```

### 적용 전 변경 사항 확인
```bash
kubectl apply -f my-resource.yaml --dry-run=server
```

---

## 6. 로그 및 디버깅

### Pod 로그 확인
```bash
kubectl logs my-pod
kubectl logs my-pod -f  # 실시간 로그 스트리밍
```

### 특정 컨테이너의 로그 확인 (멀티 컨테이너 Pod)
```bash
kubectl logs my-pod -c my-container
```

### Pod 내부 접근 (exec)
```bash
kubectl exec -it my-pod -- /bin/sh  # 또는 /bin/bash
```

### 이벤트 및 문제 확인
```bash
kubectl get events
kubectl describe pod my-pod
```

---

## 7. 노드 및 리소스 관리

### 노드 상태 확인
```bash
kubectl get nodes -o wide
```

### 특정 노드에서 실행 중인 Pod 조회
```bash
kubectl get pods --all-namespaces --field-selector spec.nodeName=my-node
```

### 노드에 Taint 추가 (Pod 제한)
```bash
kubectl taint nodes my-node key=value:NoSchedule
```

### 노드에서 Taint 제거
```bash
kubectl taint nodes my-node key=value-
```

---

## 8. 스케줄링 및 네트워크

### 특정 노드에 배포 (NodeSelector 사용)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: my-container
    image: nginx
```

### 특정 Pod 네트워크 정보 확인
```bash
kubectl get pod my-pod -o wide
```

### Service의 ClusterIP 또는 NodePort 확인
```bash
kubectl get svc my-service -o jsonpath='{.spec.clusterIP}'
kubectl get svc my-service -o jsonpath='{.spec.ports[0].nodePort}'
```

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/reference/kubectl/)

---

이 문서는 Kubernetes에서 자주 사용하는 주요 명령어들을 정리한 것으로, 운영 및 관리 시 참고할 수 있습니다.

