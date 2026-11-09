# Week 1 - Day 1: 컨테이너 기본 + 환경 설치

> 상태: 완료 (2026-11-09)
> 개념 정리: [container-vs-vm.md](container-vs-vm.md)

## 목표
- 컨테이너와 VM의 차이를 설명할 수 있다
- Docker를 설치하고 기본 명령어로 컨테이너를 다룬다

## 개념 정리
자세한 내용은 [container-vs-vm.md](container-vs-vm.md) 참고

- VM: 하이퍼바이저 위에 게스트 OS 전체를 올림 (무겁다)
- 컨테이너: 호스트 커널을 공유하고 프로세스를 격리 (가볍고 빠르다)
- 이미지(설계도) → 컨테이너(실행 인스턴스)
- 컨테이너는 안의 프로세스가 끝나면 함께 종료된다 (`hello-world`는 출력 후 바로 종료, nginx는 계속 실행)

## 실습 1. 설치 [완료]
```bash
brew install --cask orbstack   # 또는 brew install --cask docker
# 앱을 한 번 실행한 뒤
docker --version
docker info
```
- 설치한 환경: OrbStack 2.2.5 / Docker 29.8.2 (Apple Silicon)

## 실습 2. 첫 컨테이너 [완료]
```bash
docker run hello-world
docker images
docker ps -a
docker rm <이름>                  # 종료된 컨테이너 삭제
docker run --rm hello-world       # 종료 시 자동 삭제
```

## 실습 3. nginx 실행 [완료]
```bash
docker run -d --name web -p 8081:80 nginx   # 8080은 호스트에서 이미 사용 중이라 8081 사용
curl http://localhost:8081        # 또는 브라우저로 접속
docker ps
docker logs web
docker logs -f web                # Ctrl+C로 종료
docker exec web ls /usr/share/nginx/html   # 대화형(-it)이 필요 없는 형태
docker stop web
docker ps -a
docker start web
docker rm -f web
```

## 실습 4. 대화형 컨테이너 [완료]
별도 터미널(Terminal.app 등)에서 실행해야 한다. (`-it`는 입력을 주고받아야 해서 Claude Code의 `!`로는 불가)
```bash
docker run -it --rm ubuntu bash
  cat /etc/os-release
  hostname
  exit
```

## 실습 5. 데이터 유실 실험 [완료]
```bash
docker exec web sh -c 'echo "test data" > /tmp/note.txt'
docker stop web && docker start web
docker exec web cat /tmp/note.txt    # test data (남아 있음)
docker rm -f web
docker run -d --name web -p 8081:80 nginx
docker exec web cat /tmp/note.txt    # No such file or directory (사라짐)
```

## 확인 질문
- [x] `-d`, `-p`, `--name`, `--rm`, `-it` 옵션은 각각 무엇인가?
  - `-d`: 백그라운드 실행 / `--name`: 컨테이너 이름 지정 / `-p 호스트:컨테이너`: 포트 매핑
  - `--rm`: 종료되면 컨테이너 자동 삭제 (대시 두 개. 하나면 `-r -m`으로 해석돼 오류)
  - `-it`: `-i`(입력 전달) + `-t`(터미널). 컨테이너 안에서 `bash`를 직접 조작할 때 필요 (실습 4에서 사용)
- [x] `docker stop`과 `docker rm`의 차이는?
  - `stop`: 컨테이너를 멈춤 (삭제되지 않고 `Exited`로 남아 `start`로 재시작 가능)
  - `rm`: 멈춘 컨테이너를 삭제. 실행 중이면 `-f` 필요
- [x] 컨테이너를 삭제하면 안의 데이터는 어떻게 되는가?
  - 함께 삭제됨. 같은 이름, 포트, 이미지로 다시 만들어도 새 컨테이너라 파일이 없음 (`No such file or directory`)

## 정리
```bash
docker rm -f $(docker ps -aq)
```
- 이미지(`hello-world`, `nginx`, `ubuntu`)는 다음 실습을 위해 남겨 둠

## 메모

### 진행 기록 (2026-11-09)
- 환경: OrbStack 2.2.5 / Docker 29.8.2 (Apple Silicon, arm64 이미지 자동 선택)
- Claude가 한 번 대신 실행해 본 뒤 체감이 없어서, 직접 `! 명령어`로 입력하는 방식으로 다시 진행
- 실습 1~5 완료 (실습 4는 대화 중에는 건너뛰었다가 별도 터미널에서 직접 실습)

### 새로 알게 된 것
- `docker run`: 이미지로 새 컨테이너를 만들고 시작 / `docker start`: 이미 있는 컨테이너를 다시 시작 (CONTAINER ID 그대로)
- `docker ps`는 실행 중만, `docker ps -a`는 종료된 것까지 보여 줌. STATUS의 `Exited (0)`은 정상 종료
- 이름을 안 주면 `busy_germain` 같은 무작위 이름이 붙음
- `docker run -d`는 64자 컨테이너 ID를 출력, `ps`에는 앞 12자만 표시
- 컨테이너를 지워도 이미지는 남음 (이미지와 컨테이너는 별개)
- `stop`→`start`는 컨테이너가 같아 파일이 유지, `rm`→`run`은 새 컨테이너라 파일이 사라짐 → Day 3 볼륨이 필요한 이유
- 터미널 명령은 대소문자를 구분함 (`Docker`는 `unsupported argv0 "Docker"` 오류)

### 겪은 문제: 포트 충돌
- `-p 8080:80`으로 띄운 뒤 `curl localhost:8080`을 했더니 nginx 기본 페이지가 아닌 한글 페이지가 응답됨
- 원인: 호스트의 Java 프로세스가 이미 8080 포트를 사용 중이었고, 컨테이너 로그(`docker logs web`)에는 요청 기록이 없었음
- 확인: `lsof -nP -iTCP:8080 -sTCP:LISTEN`
- 해결: 호스트 포트를 바꿔 `-p 8081:80`으로 실행 (`-p 호스트포트:컨테이너포트`이므로 앞 숫자만 바꾸면 됨)
- 교훈: 응답이 왔다고 컨테이너가 응답한 것은 아니다. `docker logs`의 `GET /` 기록과 `lsof`로 확인하자

### 복습 거리
- [x] 컨테이너 vs VM 개념 정리: [container-vs-vm.md](container-vs-vm.md)
- [x] 실습 4: 별도 터미널에서 `docker run -it --rm ubuntu bash` 직접 실행
- [ ] [container-vs-vm.md](container-vs-vm.md)의 "직접 확인해 볼 것"과 "스스로 설명해 보기" 체크리스트 확인
