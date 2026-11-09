# Week 2 - Day 2: kubectl과 YAML

## 목표
- kubectl 기본 명령과 매니페스트 구조를 익힌다

## YAML 기본 구조
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
  labels:
    app: example
spec:
  containers:
    - name: c
      image: nginx
```

## 실습 1. 명령형으로 시작
```bash
kubectl run nginx --image=nginx
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
kubectl logs nginx
kubectl exec -it nginx -- bash
kubectl delete pod nginx
```

## 실습 2. 선언형 (apply)
`pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
kubectl get pod nginx -o yaml
kubectl apply -f pod.yaml        # 변경 없음 확인
kubectl delete -f pod.yaml
```

## 실습 3. 유용한 기능
```bash
kubectl run test --image=nginx --dry-run=client -o yaml     # YAML 뼈대 생성
kubectl explain pod.spec.containers
kubectl api-resources
kubectl get pods --show-labels
kubectl get pods -l app=nginx
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 실습 4. 별칭 설정 (선택)
```bash
echo 'alias k=kubectl' >> ~/.zshrc
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
```

## 확인 질문
- [ ] 명령형과 선언형의 차이는?
- [ ] `kubectl apply`를 두 번 실행하면 어떻게 되는가?
- [ ] `kubectl explain`은 언제 유용한가?

## 메모
