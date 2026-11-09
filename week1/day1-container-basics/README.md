# Week 1 - Day 1: 컨테이너 기본 + 환경 설치

## 목표
- 컨테이너와 VM의 차이를 설명할 수 있다
- Docker를 설치하고 기본 명령어로 컨테이너를 다룬다

## 개념 정리
- VM: 하이퍼바이저 위에 게스트 OS 전체를 올림 (무겁다)
- 컨테이너: 호스트 커널을 공유하고 프로세스를 격리 (가볍고 빠르다)
- 이미지(설계도) → 컨테이너(실행 인스턴스)

## 실습 1. 설치
```bash
brew install --cask docker     # 또는 brew install --cask orbstack
# 앱을 한 번 실행한 뒤
docker --version
docker info
```

## 실습 2. 첫 컨테이너
```bash
docker run hello-world
docker images
docker ps -a
```

## 실습 3. nginx 실행
```bash
docker run -d --name web -p 8080:80 nginx
curl http://localhost:8080        # 또는 브라우저로 접속
docker ps
docker logs web
docker logs -f web                # Ctrl+C로 종료
docker exec -it web bash          # 컨테이너 내부 진입
  ls /usr/share/nginx/html
  exit
docker stop web
docker start web
docker rm -f web
```

## 실습 4. 대화형 컨테이너
```bash
docker run -it --rm ubuntu bash
  cat /etc/os-release
  exit
```

## 확인 질문
- [ ] `-d`, `-p`, `--name`, `--rm`, `-it` 옵션은 각각 무엇인가?
- [ ] `docker stop`과 `docker rm`의 차이는?
- [ ] 컨테이너를 삭제하면 안의 데이터는 어떻게 되는가?

## 정리
```bash
docker rm -f $(docker ps -aq)
```

## 메모

### 진행 기록 (2026-11-09)
- 환경: OrbStack 2.2.5 / Docker 29.8.2 (Apple Silicon, arm64 이미지 자동 선택)
- 실습 1~4 완료. `hello-world`, `nginx`, `ubuntu` 실행 및 정리까지 확인

### 겪은 문제: 포트 충돌
- `-p 8080:80`으로 띄운 뒤 `curl localhost:8080`을 했더니 nginx 기본 페이지가 아닌 한글 페이지가 응답됨
- 원인: 호스트의 Java 프로세스가 이미 8080 포트를 사용 중이었고, 컨테이너 로그(`docker logs web`)에는 요청 기록이 없었음
- 확인: `lsof -nP -iTCP:8080 -sTCP:LISTEN`
- 해결: 호스트 포트를 바꿔 `-p 8081:80`으로 실행 (`-p 호스트포트:컨테이너포트`이므로 앞 숫자만 바꾸면 됨)
- 교훈: 응답이 왔다고 컨테이너가 응답한 것은 아니다. 로그와 `lsof`로 확인하자

### 직접 확인한 것
- `docker stop` 후에도 컨테이너는 `Exited` 상태로 남고 `docker start`로 재시작 가능
- 컨테이너를 `rm` 후 같은 이름으로 다시 만들면 `/tmp`에 써둔 파일이 사라짐 (데이터는 컨테이너와 함께 삭제)
- `--rm`으로 실행한 ubuntu 컨테이너는 종료 즉시 자동 삭제되어 `docker ps -a`에 남지 않음
