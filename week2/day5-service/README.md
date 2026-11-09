# Week 2 - Day 5: Service

## 목표
- Service 타입별 차이를 이해하고 Pod에 안정적으로 접근한다

## 사전 준비
Day 4의 `deploy.yaml`을 복사해 배포한다.
```bash
cp ../day4-deployment/deploy.yaml .
kubectl apply -f deploy.yaml
```

## 실습 1. ClusterIP
`svc-clusterip.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f svc-clusterip.yaml
kubectl get svc,endpoints web-svc
kubectl run tmp --rm -it --image=curlimages/curl -- curl -s http://web-svc
```

## 실습 2. DNS 확인
```bash
kubectl run tmp --rm -it --image=busybox -- nslookup web-svc
# FQDN: web-svc.default.svc.cluster.local
```

## 실습 3. NodePort
`svc-nodeport.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-np
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

kind에서는 노드가 컨테이너라 localhost로 바로 접근되지 않을 수 있다. 아래 port-forward를 사용한다.

## 실습 4. port-forward
```bash
kubectl port-forward svc/web-svc 8080:80
curl localhost:8080
```

## 실습 5. 로드밸런싱 확인
각 Pod가 자신의 호스트명을 응답하도록 만들어 요청이 분산되는지 본다.
```bash
kubectl create deploy echo --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
kubectl expose deploy echo --port=80 --target-port=8080
kubectl scale deploy echo --replicas=3
kubectl run tmp --rm -it --image=curlimages/curl -- sh -c 'for i in 1 2 3 4 5 6; do curl -s http://echo/hostname; echo; done'
```

## 확인 질문
- [ ] ClusterIP, NodePort, LoadBalancer의 차이는?
- [ ] Service는 어떤 Pod로 트래픽을 보낼지 어떻게 아는가?
- [ ] 로컬에서 LoadBalancer의 EXTERNAL-IP가 `<pending>`인 이유는?

## 정리
```bash
kubectl delete svc web-svc web-np echo
kubectl delete deploy web echo
```

## 메모
