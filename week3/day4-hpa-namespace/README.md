# Week 3 - Day 4: HPA와 Namespace

## 목표
- 부하에 따른 자동 확장을 확인하고 Namespace로 환경을 분리한다

## 실습 1. metrics-server 설치 (kind용 패치)
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch -n kube-system deployment metrics-server --type=json \
  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
kubectl top nodes          # 값이 나올 때까지 1~2분 대기
```

## 실습 2. 대상 앱 배포
`php-apache.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
        - name: php-apache
          image: registry.k8s.io/hpa-example
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  selector:
    app: php-apache
  ports:
    - port: 80
```

## 실습 3. HPA 생성 및 부하
```bash
kubectl apply -f php-apache.yaml
kubectl autoscale deploy php-apache --cpu-percent=50 --min=1 --max=5
kubectl get hpa -w
```
다른 터미널에서 부하를 준다.
```bash
kubectl run load --rm -it --image=busybox -- sh -c "while true; do wget -q -O- http://php-apache; done"
```
`REPLICAS`가 늘어나는지 확인하고, 부하를 끊은 뒤 줄어드는 시간을 관찰한다.

## 실습 4. Namespace
```bash
kubectl create ns dev
kubectl create ns prod
kubectl create deploy web --image=nginx -n dev
kubectl create deploy web --image=nginx -n prod
kubectl get deploy -A
kubectl config set-context --current --namespace=dev
kubectl get pods
kubectl config set-context --current --namespace=default
```

## 실습 5. ResourceQuota (선택)
```bash
kubectl create quota dev-quota --hard=pods=3,requests.cpu=1 -n dev
kubectl scale deploy web --replicas=5 -n dev
kubectl get pods -n dev; kubectl describe quota -n dev
```

## 확인 질문
- [ ] HPA는 무엇을 기준으로 replicas를 계산하는가?
- [ ] resources.requests가 없으면 CPU 기반 HPA가 동작하지 않는 이유는?
- [ ] Namespace는 어떤 것을 격리하고 어떤 것은 격리하지 않는가?

## 정리
```bash
kubectl delete hpa php-apache
kubectl delete -f php-apache.yaml
kubectl delete ns dev prod
```

## 메모
