# Week 2 - Day 3: Pod와 ReplicaSet

## 목표
- Pod의 특성을 이해하고 ReplicaSet의 자가 복구를 확인한다

## 실습 1. 멀티 컨테이너 Pod
`multi-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi
spec:
  containers:
    - name: web
      image: nginx
    - name: sidecar
      image: busybox
      command: ["sh", "-c", "while true; do echo sidecar alive; sleep 5; done"]
```

```bash
kubectl apply -f multi-pod.yaml
kubectl logs multi -c sidecar
kubectl exec -it multi -c web -- bash
kubectl delete -f multi-pod.yaml
```

## 실습 2. 단독 Pod는 복구되지 않는다
```bash
kubectl run solo --image=nginx
kubectl delete pod solo
kubectl get pods                  # 사라짐
```

## 실습 3. ReplicaSet
`rs.yaml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```

```bash
kubectl apply -f rs.yaml
kubectl get rs,pods -o wide
kubectl delete pod <파드 하나>      # 자동으로 다시 생성되는지 확인
kubectl scale rs web-rs --replicas=5
kubectl get pods -w
```

## 실습 4. 레이블 셀렉터 실험
```bash
kubectl label pod <파드명> app=other --overwrite
kubectl get pods --show-labels     # 새 Pod가 추가 생성되는 이유 확인
```

## 확인 질문
- [ ] Pod가 죽으면 누가 다시 만들어 주는가?
- [ ] ReplicaSet은 Pod를 어떻게 식별하는가?
- [ ] 사이드카 패턴은 어떤 경우에 쓰는가?

## 정리
```bash
kubectl delete -f rs.yaml
```

## 메모
