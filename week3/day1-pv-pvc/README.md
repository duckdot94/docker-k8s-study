# Week 3 - Day 1: PersistentVolume과 PVC

## 목표
- Pod가 재생성되어도 데이터를 유지한다

## 개념 정리
- PV: 클러스터의 스토리지 자원
- PVC: 사용자의 스토리지 요청
- StorageClass: 동적 프로비저닝 정책 (kind는 `standard` 기본 제공)

## 실습 1. StorageClass 확인
```bash
kubectl get storageclass
```

## 실습 2. PVC 생성
`pvc.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pvc,pv
```

## 실습 3. Pod에 연결
`pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: writer
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh", "-c", "date >> /data/log.txt; sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: data-pvc
```

```bash
kubectl apply -f pod.yaml
kubectl exec writer -- cat /data/log.txt
kubectl delete pod writer
kubectl apply -f pod.yaml
kubectl exec writer -- cat /data/log.txt     # 이전 기록이 남아 있는지 확인
```

## 실습 4. 삭제 정책 확인
```bash
kubectl delete pod writer
kubectl delete pvc data-pvc
kubectl get pv            # reclaimPolicy(Delete)에 따라 PV 삭제 여부 확인
```

## 확인 질문
- [ ] PV, PVC, StorageClass의 역할은?
- [ ] `ReadWriteOnce`와 `ReadWriteMany`의 차이는?
- [ ] reclaimPolicy `Retain`과 `Delete`는 각각 언제 쓰는가?

## 메모
