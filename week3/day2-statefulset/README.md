# Week 3 - Day 2: StatefulSet

## 목표
- 안정적인 이름과 전용 스토리지가 필요한 워크로드를 배포한다

## 실습 1. Headless Service + StatefulSet
`sts.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: pg
spec:
  clusterIP: None
  selector:
    app: pg
  ports:
    - port: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pg
spec:
  serviceName: pg
  replicas: 2
  selector:
    matchLabels:
      app: pg
  template:
    metadata:
      labels:
        app: pg
    spec:
      containers:
        - name: postgres
          image: postgres:16
          env:
            - name: POSTGRES_PASSWORD
              value: pw
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

```bash
kubectl apply -f sts.yaml
kubectl get sts,pods,pvc
kubectl get pods -w          # pg-0 → pg-1 순서로 생성
```

## 실습 2. 데이터 유지 확인
```bash
kubectl exec pg-0 -- psql -U postgres -c "CREATE TABLE t(id int); INSERT INTO t VALUES (1);"
kubectl delete pod pg-0
kubectl wait --for=condition=ready pod/pg-0 --timeout=90s
kubectl exec pg-0 -- psql -U postgres -c "SELECT * FROM t;"
```

## 실습 3. 고정 DNS
```bash
kubectl run tmp --rm -it --image=busybox -- nslookup pg-0.pg
```

## 실습 4. 스케일 다운 후 PVC 확인
```bash
kubectl scale sts pg --replicas=1
kubectl get pods,pvc         # pg-1의 PVC는 남아 있음
```

## 확인 질문
- [ ] Deployment와 StatefulSet의 차이는?
- [ ] Headless Service는 왜 필요한가?
- [ ] StatefulSet 삭제 시 PVC는 어떻게 되는가?

## 정리
```bash
kubectl delete -f sts.yaml
kubectl delete pvc -l app=pg
```

## 메모
