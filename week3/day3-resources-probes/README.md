# Week 3 - Day 3: 리소스 제한과 Probe

## 목표
- requests/limits를 설정하고 헬스체크로 자가 복구를 구현한다

## 실습 1. 리소스 설정
`res.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: res-demo
spec:
  containers:
    - name: c
      image: nginx
      resources:
        requests:
          cpu: 100m
          memory: 64Mi
        limits:
          cpu: 200m
          memory: 128Mi
```

```bash
kubectl apply -f res.yaml
kubectl describe pod res-demo | grep -A6 -i "limits\|requests\|QoS"
```

## 실습 2. OOMKilled 재현
`oom.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-demo
spec:
  containers:
    - name: c
      image: polinux/stress
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
      resources:
        limits:
          memory: 100Mi
```

```bash
kubectl apply -f oom.yaml
kubectl get pod oom-demo -w
kubectl describe pod oom-demo | grep -i -A3 "last state"
```

## 실습 3. Liveness / Readiness Probe
`probe.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh", "-c", "touch /tmp/healthy; sleep 30; rm /tmp/healthy; sleep 600"]
      livenessProbe:
        exec:
          command: ["cat", "/tmp/healthy"]
        initialDelaySeconds: 5
        periodSeconds: 5
      readinessProbe:
        exec:
          command: ["cat", "/tmp/healthy"]
        periodSeconds: 5
```

```bash
kubectl apply -f probe.yaml
kubectl get pod probe-demo -w          # RESTARTS 증가 확인
kubectl describe pod probe-demo | tail -20
```

## 실습 4. HTTP Probe (nginx)
`/` 경로에 httpGet probe를 걸고, 존재하지 않는 경로(`/nope`)로 바꿨을 때 어떻게 되는지 확인한다.

## 확인 질문
- [ ] requests와 limits의 차이는?
- [ ] CPU 초과와 메모리 초과 시 동작은 어떻게 다른가?
- [ ] Liveness 실패와 Readiness 실패의 결과는 각각 무엇인가?

## 정리
```bash
kubectl delete pod res-demo oom-demo probe-demo
```

## 메모
