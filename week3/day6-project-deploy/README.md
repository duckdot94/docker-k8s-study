# Week 3 - Day 6: 미니 프로젝트 2 (1/2) - K8s 배포

## 목표
- 1주차 프로젝트를 Kubernetes에 배포한다

## 요구사항
- [ ] 앱 이미지를 로컬 클러스터에 로드: `kind load docker-image <이미지> --name study`
  (또는 Docker Hub의 이미지를 사용)
- [ ] 별도 Namespace 사용 (예: `myapp`)
- [ ] Deployment (replicas 2 이상, RollingUpdate)
- [ ] Service + Ingress로 외부 접근
- [ ] ConfigMap / Secret으로 설정과 비밀번호 분리
- [ ] DB는 StatefulSet + PVC
- [ ] 리소스 requests/limits, Liveness/Readiness Probe 설정

## 디렉터리 예시
```
manifests/
├── 00-namespace.yaml
├── 10-config.yaml
├── 20-db-statefulset.yaml
├── 30-app-deployment.yaml
├── 40-service.yaml
└── 50-ingress.yaml
```

## 진행 순서
1. Namespace → ConfigMap/Secret → DB → App → Service → Ingress 순으로 적용
2. `kubectl get all -n myapp`으로 상태 확인
3. 브라우저/curl로 접속 확인
4. 한 번에 `kubectl apply -f manifests/`로 올라가는지 확인

## 체크리스트
- [ ] Pod를 삭제해도 서비스가 유지된다
- [ ] DB Pod를 재생성해도 데이터가 남는다
- [ ] 새 이미지 태그로 롤링 업데이트가 된다
- [ ] 문제가 생기면 롤백할 수 있다

## 진행 기록
- 막혔던 점:
- 해결 방법:
