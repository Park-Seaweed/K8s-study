# Kubernetes ClusterIP 서비스

ClusterIP는 Kubernetes에서 기본적으로 제공되는 서비스 유형으로, 클러스터 내부에서만 접근 가능한 가상 IP를 제공합니다. 외부에서는 직접 접근할 수 없으며, 클러스터 내부에서 서비스 간 통신을 위해 사용됩니다.

---

## 주요 특징

- **클러스터 내부 전용**: 외부에서는 접근 불가
- **가상 IP 할당**: 특정 노드가 아닌 클러스터 내부에서 접근 가능
- **내부 DNS 지원**: `service-name.namespace.svc.cluster.local` 형태로 접근 가능

---

## 1. CLI를 사용하여 ClusterIP 서비스 생성

```bash
kubectl expose deployment my-app --name=my-clusterip-service --port=80 --target-port=8080 --type=ClusterIP
```

### 생성된 서비스 확인
```bash
kubectl get services
```

### ClusterIP 확인
```bash
kubectl get svc my-clusterip-service -o jsonpath='{.spec.clusterIP}'
```

### 서비스 삭제
```bash
kubectl delete service my-clusterip-service
```

---

## 2. YAML을 사용하여 ClusterIP 서비스 생성

### clusterip-service.yaml 파일 작성
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### 생성 명령어
```bash
kubectl apply -f clusterip-service.yaml
```

### 서비스 확인
```bash
kubectl get services
```

---

## 3. ClusterIP 서비스에 접근하는 방법

1. **Pod 내부에서 DNS를 사용한 접근**
   ```bash
   curl http://my-clusterip-service
   ```
2. **Pod 내부에서 직접 ClusterIP를 사용한 접근**
   ```bash
   kubectl get svc my-clusterip-service -o jsonpath='{.spec.clusterIP}'
   curl http://<CLUSTER_IP>
   ```

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/services-networking/service/)

---

ClusterIP는 Kubernetes에서 기본적으로 제공되는 서비스 유형으로, 내부 애플리케이션 간 통신을 원활하게 합니다.

