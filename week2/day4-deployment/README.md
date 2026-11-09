# Week 2 - Day 4: Deployment

## 목표
- 스케일링, 롤링 업데이트, 롤백을 직접 수행한다

## 실습 1. Deployment 생성
`deploy.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f deploy.yaml
kubectl get deploy,rs,pods
```

## 실습 2. 스케일링
```bash
kubectl scale deploy web --replicas=5
kubectl get pods
```

## 실습 3. 롤링 업데이트
```bash
kubectl set image deploy/web nginx=nginx:1.27
kubectl rollout status deploy/web
kubectl get rs                      # 새 RS와 이전 RS 확인
kubectl rollout history deploy/web
```

## 실습 4. 롤백
```bash
kubectl set image deploy/web nginx=nginx:does-not-exist    # 잘못된 이미지
kubectl rollout status deploy/web                          # 진행이 멈춤 (Ctrl+C)
kubectl get pods                                           # ImagePullBackOff 확인
kubectl rollout undo deploy/web
kubectl rollout status deploy/web
```

## 확인 질문
- [ ] Deployment, ReplicaSet, Pod의 관계는?
- [ ] `maxSurge`와 `maxUnavailable`의 의미는?
- [ ] 잘못된 이미지로 배포해도 서비스가 유지된 이유는?

## 정리
```bash
kubectl delete -f deploy.yaml
```

## 메모
