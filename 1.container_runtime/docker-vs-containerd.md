# Docker vs Containerd

쿠버네티스(Kubernetes)에서 컨테이너를 실행하려면 **컨테이너 런타임**이 필요합니다. 대표적인 컨테이너 런타임으로는 **Docker**와 **Containerd**가 있으며, 최근 쿠버네티스에서는 **Containerd 사용이 증가**하는 추세입니다. 이 문서에서는 두 런타임의 차이점을 간단히 정리합니다.

---

## 차이점 요약

| 비교 항목       | Docker | Containerd |
|---------------|-------------|----------------|
| **역할**      | 컨테이너 빌드, 실행, 배포 포함 | 컨테이너 실행과 관리 전담 |
| **구성 요소**  | Docker CLI + Docker Daemon + Containerd + runc | Containerd + runc |
| **빌드 지원**  | `docker build` 명령어 제공 | 별도 도구 필요 (BuildKit, Kaniko 등) |
| **K8s 연동**  | Dockershim(1.24 이후 제거) 필요 | CRI 지원, 네이티브 호환 |
| **운영 방식**  | 추가 Daemon 실행으로 오버헤드 존재 | 경량 구조, 대규모 운영 적합 |

---

## 쿠버네티스에서의 선택

Docker  
- 개발 환경에서 친숙한 CLI 제공  
- 로컬 테스트 및 CI/CD 파이프라인에 적합  

Containerd  
- 쿠버네티스에서 CRI 호환 가능  
- 추가 Daemon 없이 경량 운영 가능  
- 대규모 프로덕션 환경 적합  

주의  
쿠버네티스 1.24부터 Dockershim 지원 종료  
Containerd 또는 CRI-O 권장  

---

## 결론

- 개발 및 로컬 환경: Docker  
- 쿠버네티스 운영 환경: Containerd  
- CI/CD 빌드 환경: Docker, Kaniko, BuildKit 등 고려  

기존에 Docker 사용 중이라면 Containerd 전환 고려  

---

## 참고 자료

- [Kubernetes 공식 문서 - Container Runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
- [Containerd 공식 GitHub](https://github.com/containerd/containerd)
- [Docker 공식 문서](https://docs.docker.com/)

---

Docker와 Containerd의 차이를 설명하고 쿠버네티스에서 어떤 선택을 해야 하는지 안내하는 문서

