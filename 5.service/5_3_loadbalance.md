# Kubernetes LoadBalancer 서비스

LoadBalancer는 Kubernetes에서 제공하는 서비스 유형 중 하나로, 클러스터 외부에서 서비스에 접근할 수 있도록 클라우드 제공자의 로드 밸런서를 사용하여 트래픽을 분산합니다.

---

## 주요 특징

- **외부 접근 가능**: 클라우드 환경에서 퍼블릭 IP를 통해 서비스에 접근 가능
- **자동 로드 밸런싱**: 클러스터 외부에서 들어오는 트래픽을 여러 Pod로 분산
- **클라우드 의존성**: AWS, GCP, Azure 등 클라우드 환경에서 지원됨

---

## 1. CLI를 사용하여 LoadBalancer 서비스 생성

```bash
kubectl expose deployment my-app --name=my-loadbalancer-service --port=80 --target-port=8080 --type=LoadBalancer
```

### 생성된 서비스 확인
```bash
kubectl get services
```

### 할당된 EXTERNAL-IP 확인
```bash
kubectl get svc my-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### 서비스 삭제
```bash
kubectl delete service my-loadbalancer-service
```

---

## 2. YAML을 사용하여 LoadBalancer 서비스 생성

### loadbalancer-service.yaml 파일 작성
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### 생성 명령어
```bash
kubectl apply -f loadbalancer-service.yaml
```

### 서비스 확인
```bash
kubectl get services
```

---

## 3. LoadBalancer 서비스에 접근하는 방법

1. **할당된 EXTERNAL-IP 확인 후 접근**
   ```bash
   kubectl get svc my-loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
   curl http://<EXTERNAL_IP>
   ```

2. **클라우드 환경에서 로드 밸런서 관리**
   - AWS: Elastic Load Balancer (ELB) 자동 생성
   - GCP: Cloud Load Balancer 자동 생성
   - Azure: Azure Load Balancer 자동 생성

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/services-networking/service/)

---

LoadBalancer 서비스는 클라우드 환경에서 외부 트래픽을 받아 여러 Pod로 분산하는 역할을 하며, 퍼블릭 IP를 제공하여 외부에서 쉽게 접근할 수 있도록 합니다.

