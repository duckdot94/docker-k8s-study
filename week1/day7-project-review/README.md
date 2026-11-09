# Week 1 - Day 7: 미니 프로젝트 1 (2/2) - 마무리와 복습

## 목표
- 프로젝트를 정리하고 1주차 내용을 복습한다

## 실습 1. 마무리
- [ ] healthcheck 추가
- [ ] `.env` 분리 및 `.env.example` 작성
- [ ] 프로젝트 `README.md` 작성 (실행 방법, 구성도, 포트)
- [ ] 이미지를 Docker Hub에 push

## 실습 2. 디버깅 연습
```bash
docker compose logs -f
docker compose exec <서비스> sh
docker stats
docker inspect <컨테이너>
docker system df
```
의도적으로 오류를 만들어 원인을 찾는다.
- 잘못된 환경변수 → 로그로 원인 확인
- 잘못된 포트 매핑 → `docker ps`로 확인
- DB 미기동 상태에서 앱 기동 → 재시도/healthcheck 필요성 확인

## 1주차 복습 퀴즈
- [ ] 이미지와 컨테이너의 차이를 설명하라
- [ ] Dockerfile 캐시를 잘 쓰려면 명령 순서를 어떻게 정해야 하나?
- [ ] 볼륨, 바인드 마운트, tmpfs를 비교하라
- [ ] bridge 네트워크에서 컨테이너끼리 통신하는 방법은?
- [ ] 멀티스테이지 빌드의 장점은?
- [ ] Compose가 해결해 주는 문제는 무엇인가?

## 정리
```bash
docker system prune -a --volumes   # 주의: 사용하지 않는 모든 리소스 삭제
```

## 한 주 회고
- 잘 이해한 것:
- 헷갈리는 것:
- 2주차 전에 보완할 것:
