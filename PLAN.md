# Docker & Kubernetes 학습 계획

- 기간: 3주 (약 15~20일), 하루 1~2시간
- 환경: macOS (Apple Silicon, 24GB)
- 목표: 자격증 없이 실무 활용 수준의 이해와 실습
- 도구 설치는 학습 과정(1주차 Day 1)에서 진행

## 진행 현황 체크

- [ ] 1주차: Docker (진행 중, Day 1 완료 / 다음: Day 2 이미지와 Dockerfile)
- [ ] 2주차: Kubernetes 기초
- [ ] 3주차: Kubernetes 심화 + 프로젝트

---

## 1주차: Docker (5~7일)

### Day 1: 컨테이너 기본 + 환경 설치 (완료: 2026-11-09)
- [x] 컨테이너 vs VM 개념 → [container-vs-vm.md](week1/day1-container-basics/container-vs-vm.md)
- [x] OrbStack 설치 (`brew install --cask orbstack`) - OrbStack 2.2.5 / Docker 29.8.2
- [x] `docker run`, `ps`, `exec`, `logs`, `stop`, `start`, `rm`
- [x] nginx 컨테이너 띄워 `curl`로 확인 (호스트 8080 포트 충돌로 `-p 8081:80` 사용)
- [x] 대화형 컨테이너 `docker run -it --rm ubuntu bash` (별도 터미널에서 실습)

**배운 것**
- `run`은 새 컨테이너 생성, `start`는 기존 컨테이너 재시작. `stop`해도 컨테이너는 `Exited`로 남고 `rm`해야 삭제됨
- `--rm`: 종료 시 컨테이너 자동 삭제 / `-d`: 백그라운드 / `-p 호스트:컨테이너`: 포트 매핑 / `--name`: 이름 지정 (생략 시 무작위 이름)
- 컨테이너를 지워도 이미지는 남음 (이미지와 컨테이너는 별개)
- `stop`→`start`는 컨테이너 안 파일이 유지되지만, `rm`→`run`은 새 컨테이너라 파일이 사라짐 → Day 3 볼륨의 필요성
- 응답이 왔다고 컨테이너가 응답한 것은 아님. 포트 충돌 시 `docker logs`와 `lsof -nP -iTCP:<포트> -sTCP:LISTEN`으로 확인

### Day 2: 이미지와 Dockerfile
- [ ] 이미지, 레이어, 캐시 구조
- [ ] Dockerfile 작성 (`FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`)
- [ ] `docker build`, `.dockerignore`

### Day 3: 볼륨과 네트워크
- [ ] 볼륨 vs 바인드 마운트
- [ ] bridge 네트워크, 컨테이너 간 통신
- [ ] 포트 매핑

### Day 4: Docker Compose
- [ ] `docker-compose.yml` 작성
- [ ] 앱 + DB 다중 컨테이너 구성
- [ ] 환경변수, `depends_on`, 볼륨 연결

### Day 5: 최적화와 레지스트리
- [ ] 멀티스테이지 빌드
- [ ] 이미지 크기 줄이기
- [ ] Docker Hub push/pull
- [ ] Apple Silicon 주의점 (arm64 / `--platform linux/amd64`)

### Day 6~7: 미니 프로젝트 1
- [ ] 본인의 간단한 앱을 Dockerize
- [ ] Compose로 앱 + DB 구성
- [ ] README 작성

---

## 2주차: Kubernetes 기초 (7일)

### Day 1~2: 아키텍처와 환경
- [ ] Control Plane / Node 구성요소 (API Server, etcd, kubelet 등)
- [ ] kind 또는 minikube 설치 (`brew install kubectl kind helm`)
- [ ] `kubectl` 기본 명령어 (`get`, `describe`, `apply`, `logs`, `exec`)
- [ ] YAML 매니페스트 구조

### Day 3~4: 워크로드
- [ ] Pod
- [ ] ReplicaSet
- [ ] Deployment (스케일링, 롤링 업데이트, 롤백)

### Day 5~6: 네트워킹
- [ ] Service (ClusterIP / NodePort / LoadBalancer)
- [ ] Ingress (nginx ingress controller)
- [ ] `kubectl port-forward`

### Day 7: 설정 관리
- [ ] ConfigMap
- [ ] Secret
- [ ] 환경변수 / 볼륨으로 주입

---

## 3주차: Kubernetes 심화 + 프로젝트 (7일)

### Day 1~2: 스토리지
- [ ] PersistentVolume / PersistentVolumeClaim
- [ ] StorageClass
- [ ] StatefulSet (DB 배포)

### Day 3~4: 운영 기능
- [ ] 리소스 requests / limits
- [ ] Liveness / Readiness Probe
- [ ] HPA (오토스케일링)
- [ ] Namespace

### Day 5: Helm
- [ ] Helm 개념, chart 설치/삭제
- [ ] 간단한 chart 작성

### Day 6~7: 미니 프로젝트 2
- [ ] 1주차 앱을 K8s에 배포 (Deployment + Service + Ingress + ConfigMap/Secret + PVC)
- [ ] 장애 상황 실습 (Pod 삭제, 잘못된 이미지, 롤백)
- [ ] README 정리

---

## 학습 원칙

1. 읽기보다 직접 실행하기. 매일 실습 위주로 진행
2. 에러가 나면 `describe`, `logs`로 원인을 먼저 추적
3. 학습한 YAML과 명령어는 폴더에 저장 (예: `week1/`, `week2/`, `week3/`)
4. 막힌 내용은 `NOTES.md`에 기록

## 참고 자료

- Docker 공식 문서: https://docs.docker.com
- Kubernetes 공식 문서: https://kubernetes.io/docs
- Kubernetes 튜토리얼: https://kubernetes.io/docs/tutorials/
- kind: https://kind.sigs.k8s.io
- Helm: https://helm.sh/docs

## 주의사항 (Mac)

- Apple Silicon에서 일부 이미지는 amd64 전용 → `--platform linux/amd64` 사용
- 로컬에서는 LoadBalancer 외부 IP가 안 잡힘 → `port-forward` / NodePort 사용
