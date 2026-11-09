# Week 2 - Day 7: ConfigMap과 Secret

## 목표
- 설정과 민감정보를 이미지에서 분리해 주입한다

## 실습 1. ConfigMap 생성
```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=dev \
  --from-literal=LOG_LEVEL=debug
kubectl get cm app-config -o yaml
```

## 실습 2. 환경변수로 주입
`pod-env.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh", "-c", "env | grep -E 'APP_|LOG_|DB_'; sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

## 실습 3. Secret 생성
```bash
kubectl create secret generic db-secret --from-literal=password=s3cr3t
kubectl get secret db-secret -o yaml
echo "<base64 값>" | base64 -d          # base64는 암호화가 아니다
kubectl apply -f pod-env.yaml
kubectl logs env-demo
```

## 실습 4. 볼륨으로 마운트
`pod-vol.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-demo
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: cfg
          mountPath: /etc/config
  volumes:
    - name: cfg
      configMap:
        name: app-config
```

```bash
kubectl apply -f pod-vol.yaml
kubectl exec vol-demo -- ls /etc/config
kubectl exec vol-demo -- cat /etc/config/APP_ENV
```

## 실습 5. 변경 반영 확인
```bash
kubectl edit cm app-config               # LOG_LEVEL 변경
kubectl exec vol-demo -- cat /etc/config/LOG_LEVEL   # 잠시 후 반영
kubectl exec env-demo -- env | grep LOG  # 환경변수는 재시작 전까지 그대로
```

## 확인 질문
- [ ] 환경변수 주입과 볼륨 마운트 방식의 갱신 차이는?
- [ ] Secret이 안전하지 않은 이유와 보완책은?
- [ ] 설정을 이미지에 넣지 않는 이유는?

## 정리
```bash
kubectl delete pod env-demo vol-demo
kubectl delete cm app-config
kubectl delete secret db-secret
```

## 2주차 회고
- 잘 이해한 것:
- 헷갈리는 것:
