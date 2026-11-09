# Week 3 - Day 5: Helm

## 목표
- 차트를 설치/업그레이드/롤백하고, 간단한 차트를 직접 만든다

## 실습 1. 기존 차트 설치
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm install my-nginx bitnami/nginx
helm list
kubectl get all -l app.kubernetes.io/instance=my-nginx
```

## 실습 2. values 오버라이드
```bash
helm show values bitnami/nginx | head -60
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
helm get values my-nginx
helm history my-nginx
helm rollback my-nginx 1
```

## 실습 3. 직접 차트 만들기
```bash
helm create mychart
tree mychart
```
`mychart/values.yaml`에서 다음을 수정한다.
```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.27"
service:
  type: ClusterIP
  port: 80
```

```bash
helm template mychart ./mychart          # 렌더링 결과 확인
helm lint ./mychart
helm install demo ./mychart
helm upgrade demo ./mychart --set replicaCount=4
```

## 실습 4. 환경별 values
`values-dev.yaml`, `values-prod.yaml`을 만들고 아래처럼 배포한다.
```bash
helm install demo-dev ./mychart -f values-dev.yaml -n dev --create-namespace
```

## 확인 질문
- [ ] Chart, Release, Repository의 차이는?
- [ ] `helm template`은 언제 유용한가?
- [ ] `helm rollback`은 무엇을 되돌리는가?

## 정리
```bash
helm uninstall my-nginx demo
helm uninstall demo-dev -n dev
```

## 메모
