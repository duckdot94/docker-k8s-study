# Week 1 - Day 2: 이미지와 Dockerfile

> 상태: 완료 (2026-11-10) / `ENTRYPOINT` 직접 바꿔 보기와 `docker image inspect`는 하지 않음

## 목표
- 이미지 레이어와 빌드 캐시 개념을 이해한다
- Dockerfile을 작성해 이미지를 빌드한다

## 개념 정리
- 이미지는 읽기 전용 레이어의 쌓임. 파일을 바꾸는 명령(`FROM`, `COPY`, `RUN`)이 레이어를 만들고, `WORKDIR`/`EXPOSE`/`CMD`/`ENV`는 크기 0B의 설정 정보만 기록한다
- 빌드 캐시: 각 단계의 **입력**(앞 단계 결과 + 명령 + 복사할 파일 내용)이 지난번과 같으면 재사용(`CACHED`), 달라지면 그 단계부터 아래는 모두 다시 실행
- 변경이 적은 명령을 위에, 자주 바뀌는 명령(소스 복사)을 아래에 두면 캐시 효율이 좋아진다
- `FROM python:3.12-slim`은 그 이미지가 가진 모든 레이어를 통째로 가져온다 (우리 `COPY`는 4.1kB, 베이스가 대부분)
- `RUN`은 **빌드할 때**, `CMD`/`ENTRYPOINT`는 **컨테이너를 시작할 때** 실행
- `COPY`는 복사만, `ADD`는 압축 해제와 URL 다운로드까지 해서 예상과 다르게 동작할 수 있으므로 `COPY` 권장

## 실습 1. 앱 준비 [완료]
`app/app.py`
```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import socket

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(f"Hello from {socket.gethostname()}\n".encode())

HTTPServer(("0.0.0.0", 8000), H).serve_forever()
```
- 실습 4에서 문구를 `Hi from`으로 바꿨기 때문에 현재 `app/app.py`에는 `Hi from`이 들어 있다

## 실습 2. Dockerfile [완료]
`app/Dockerfile`
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python", "app.py"]
```
- 작성법 설명은 Node 예시와 비교하며 받았고, 파일은 Claude가 대신 만들었다. 다음에 Dockerfile을 쓸 때는 직접 쓰는 것이 목표

`app/.dockerignore`
```
__pycache__
*.pyc
.git
```

## 실습 3. 빌드와 실행 [완료]
```bash
cd app
docker build -t hello-py:1.0 .
docker run -d --name hello -p 8000:8000 hello-py:1.0
curl localhost:8000              # Hello from <컨테이너 ID 앞 12자>
docker history hello-py:1.0       # 레이어 확인
```
- 빌드 출력은 `[1/3] FROM`, `[2/3] WORKDIR`, `[3/3] COPY` 3단계 (`EXPOSE`, `CMD`는 단계로 안 나옴)
- 응답의 호스트명이 `docker run`이 출력한 컨테이너 ID 앞 12자와 같았다
- `docker history`: 우리가 만든 층은 `COPY` 4.1kB뿐, 베이스의 Debian 109MB와 Python 설치 43.7MB가 대부분

## 실습 4. 캐시 확인 [완료]
| 빌드 | app.py | 결과 |
|---|---|---|
| `1.1` | 변경 없음 | 모든 단계 `CACHED` (설정 해시도 `1.0`과 동일) |
| `1.2` | `Hello` → `Hi`로 수정 | `WORKDIR`는 `CACHED`, `COPY`부터 다시 실행 |

- 파일 변경에는 `sed -i '' 's/Hello from/Hi from/' app.py` 사용
- 이미지를 새로 빌드해도 **이미 실행 중인 컨테이너는 바뀌지 않는다**. `hello`(1.0)는 `Hello from`, `hello2`(1.2)는 `Hi from` 응답. 반영하려면 컨테이너를 지우고 새 이미지로 다시 만들어야 함

## 실습 5. CMD vs ENTRYPOINT [일부 완료]
```bash
docker run --rm hello-py:1.2 python -c "print('CMD를 덮어썼다')"   # CMD 덮어쓰기 확인
```
- `CMD`는 `docker run` 뒤에 명령을 쓰면 대체됨 (기본값), `ENTRYPOINT`는 뒤에 쓴 것이 인자로 붙음
- [ ] Dockerfile을 `ENTRYPOINT ["python"]` + `CMD ["app.py"]`로 바꿔 동작 차이를 직접 확인하기 (하지 않음)

## 확인 질문
- [x] `COPY`와 `ADD`의 차이는?
  - 처음에는 "모르겠다"고 답함 → 설명 후 이해. `COPY`는 복사만, `ADD`는 압축 해제와 URL 다운로드 포함
- [x] `RUN`, `CMD`, `ENTRYPOINT`의 실행 시점은?
  - 선택형 문제로 확인: `RUN`은 빌드할 때, `CMD`/`ENTRYPOINT`는 컨테이너 시작할 때. `docker run 이미지 echo hi`는 `CMD`를 덮어써서 `echo hi`가 실행됨
- [x] 소스를 수정했는데 이전 레이어가 재사용되는 이유는?
  - 입력이 바뀐 단계부터 다시 실행하고, 입력이 같은 앞 단계는 재사용. `app.py`를 바꿔도 `FROM`은 `app.py`와 무관해서 재사용된다

## 정리
```bash
docker rm -f hello hello2
```
- 컨테이너는 모두 삭제, 이미지 `hello-py:1.0/1.1/1.2`는 남겨 둠

## 메모

### 새로 알게 된 것
- `docker images`의 IMAGE ID는 설정 해시가 아니라 매니페스트 목록 해시라, 내용이 같은 빌드(`1.0`과 `1.1`)도 ID가 다르다 (빌드 증명 정보 때문). 빌드 출력의 `exporting config` 해시가 같으면 이미지 내용은 같다
- 같은 이미지의 레이어는 공유되므로 `DISK USAGE`가 각각 201MB여도 실제 디스크를 3배 쓰지는 않는다
- 버전 태그(`1.0`, `1.2`)가 있어야 어떤 컨테이너가 어느 버전인지 구분할 수 있다
- `DISK USAGE`는 풀어놓은 크기, `CONTENT SIZE`는 압축된 크기

### 겪은 문제
- 입력이 중간에 끊겨 전송되는 일이 몇 번 있었다 (`cd week1/d`, `docker im`). 길면 복사해서 붙여넣기
- 명령 앞에 `!`를 빼먹으면 실행되지 않는다
- 대소문자 구분: `Docker`는 오류 (Day 1)

### 복습 거리
- [ ] Dockerfile을 보지 않고 직접 처음부터 써 보기
- [ ] `ENTRYPOINT` + `CMD` 조합 실험
- [ ] 자주 바뀌는 `COPY`를 위로 올렸을 때 캐시가 어떻게 달라지는지 비교 (Day 4에서 `pip install`과 함께)
