# Kubernetes NodePort 서비스

NodePort는 Kubernetes에서 제공하는 서비스 유형 중 하나로, 클러스터 외부에서 노드의 특정 포트를 통해 서비스에 접근할 수 있도록 합니다. 각 노드에서 지정된 포트를 열어 외부 트래픽을 해당 서비스로 전달합니다.

---

## 주요 특징

- **외부 접근 가능**: 클러스터 외부에서 Node의 IP와 포트를 통해 접근 가능
- **자동 포트 할당**: 기본적으로 30000~32767 범위의 포트가 할당됨
- **노드 간 로드 밸런싱**: 클러스터 내 모든 노드에서 동일한 포트를 통해 서비스에 접근 가능

---

## 1. CLI를 사용하여 NodePort 서비스 생성

```bash
kubectl expose deployment my-app --name=my-nodeport-service --port=80 --target-port=8080 --type=NodePort
```

### 생성된 서비스 확인
```bash
kubectl get services
```

### NodePort 확인
```bash
kubectl get svc my-nodeport-service -o jsonpath='{.spec.ports[0].nodePort}'
```

### 서비스 삭제
```bash
kubectl delete service my-nodeport-service
```

---

## 2. YAML을 사용하여 NodePort 서비스 생성

### nodeport-service.yaml 파일 작성
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 31000
  type: NodePort
```

### 생성 명령어
```bash
kubectl apply -f nodeport-service.yaml
```

### 서비스 확인
```bash
kubectl get services
```

---

## 3. NodePort 서비스에 접근하는 방법

1. **외부에서 접근**
   ```bash
   curl http://<NODE_IP>:<NODE_PORT>
   ```
2. **NodePort 직접 확인 후 접근**
   ```bash
   kubectl get svc my-nodeport-service -o jsonpath='{.spec.ports[0].nodePort}'
   curl http://<NODE_IP>:<NODE_PORT>
   ```

---

## 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/concepts/services-networking/service/)

---

NodePort는 클러스터 외부에서 직접 서비스에 접근할 수 있도록 하는 기본적인 방법 중 하나로, 테스트 및 내부 네트워크 서비스에 적합합니다.