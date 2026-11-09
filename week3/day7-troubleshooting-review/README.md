# Week 3 - Day 7: 장애 실습과 전체 복습

## 목표
- 흔한 장애를 재현하고 원인을 추적하는 방법을 익힌다

## 디버깅 기본 순서
```bash
kubectl get pods -n <ns>
kubectl describe pod <pod> -n <ns>      # Events 확인
kubectl logs <pod> [-c <container>] [--previous]
kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp
kubectl exec -it <pod> -- sh
```

## 장애 실습
각 항목을 일부러 만들고 증상과 원인, 해결을 아래 표에 기록한다.

| # | 상황 | 재현 방법 | 증상 | 원인 | 해결 |
|---|---|---|---|---|---|
| 1 | 잘못된 이미지 | 존재하지 않는 태그 지정 | | | |
| 2 | CrashLoopBackOff | 컨테이너 command를 `exit 1`로 | | | |
| 3 | Pending | requests.cpu를 매우 크게 | | | |
| 4 | Service 연결 안 됨 | selector 레이블 오타 | | | |
| 5 | Ingress 404 | 서비스 이름/포트 오타 | | | |
| 6 | OOMKilled | 메모리 limit 과소 설정 | | | |
| 7 | Probe 실패 | readiness 경로 오타 | | | |
| 8 | ConfigMap 누락 | 없는 ConfigMap 참조 | | | |

힌트: `kubectl get endpoints <svc>`가 비어 있으면 selector를 의심한다.

## 전체 복습 퀴즈
- [ ] Pod, ReplicaSet, Deployment의 관계
- [ ] Service 3가지 타입과 용도
- [ ] Ingress가 필요한 이유
- [ ] ConfigMap과 Secret의 차이와 한계
- [ ] PV/PVC/StorageClass 흐름
- [ ] Deployment와 StatefulSet 선택 기준
- [ ] Probe 두 종류의 역할
- [ ] HPA 동작 원리
- [ ] Helm이 해결하는 문제

## 다음 단계 후보
- Kustomize, ArgoCD 등 GitOps
- Prometheus + Grafana 모니터링
- RBAC, NetworkPolicy 등 보안
- 클라우드 관리형 K8s (EKS / GKE / AKS)
- 필요 시 CKA / CKAD 도전

## 정리
```bash
kind delete cluster --name study
docker system prune -a --volumes
```

## 3주 회고
- 가장 도움이 된 실습:
- 더 공부할 것:
- 실무에 적용할 아이디어:
