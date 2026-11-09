# Week 2 - Day 6: Ingress

## 목표
- Ingress Controller를 설치하고 경로/호스트 기반 라우팅을 구성한다

## 실습 1. Ingress용 클러스터 재생성
기존 클러스터를 삭제하고 80/443 포트를 매핑해 다시 만든다.
`kind-ingress.yaml`
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
  - role: worker
```

```bash
kind delete cluster --name study
kind create cluster --name study --config kind-ingress.yaml
```

## 실습 2. Ingress Controller 설치
```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller --timeout=120s
```

## 실습 3. 두 개의 앱 배포
```bash
kubectl create deploy app1 --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
kubectl create deploy app2 --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
kubectl expose deploy app1 --port=80 --target-port=8080
kubectl expose deploy app2 --port=80 --target-port=8080
```

## 실습 4. Ingress 작성
`ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2
                port:
                  number: 80
```

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
curl localhost/app1/hostname
curl localhost/app2/hostname
```

## 실습 5. 호스트 기반 라우팅 (선택)
`rules[].host: app1.local` 형태로 바꾸고 아래처럼 호출한다.
```bash
curl -H "Host: app1.local" localhost/hostname
```

## 확인 질문
- [ ] Ingress와 Ingress Controller의 차이는?
- [ ] Service로 노출하는 방식과 비교한 장점은?
- [ ] `pathType`의 `Prefix`와 `Exact` 차이는?

## 정리
```bash
kubectl delete -f ingress.yaml
kubectl delete svc app1 app2
kubectl delete deploy app1 app2
```

## 메모
