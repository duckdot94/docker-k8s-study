# Week 2 - Day 1: 아키텍처와 환경 구성

## 목표
- Kubernetes 구성요소를 이해한다
- 로컬 클러스터(kind)를 만들고 접속한다

## 개념 정리
- Control Plane: API Server, etcd, Scheduler, Controller Manager
- Node: kubelet, kube-proxy, 컨테이너 런타임
- 선언형 모델: 원하는 상태(spec)를 적으면 컨트롤러가 현재 상태를 맞춘다

## 실습 1. 도구 설치
```bash
brew install kubectl kind helm
kubectl version --client
kind --version
```

## 실습 2. 클러스터 생성
`kind-config.yaml` (노드 3개)
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

```bash
kind create cluster --name study --config kind-config.yaml
kubectl cluster-info --context kind-study
kubectl get nodes -o wide
```

## 실습 3. 구성요소 확인
```bash
kubectl get pods -n kube-system
kubectl describe node study-control-plane | head -50
docker ps          # 노드가 컨테이너로 떠 있는 것 확인
kubectl config get-contexts
```

## 확인 질문
- [ ] `kubectl`은 어떤 컴포넌트와 통신하는가?
- [ ] etcd에는 무엇이 저장되는가?
- [ ] kind에서 노드는 무엇으로 구현되는가?

## 메모
