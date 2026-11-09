# Week 1 - Day 2: 이미지와 Dockerfile

## 목표
- 이미지 레이어와 빌드 캐시 개념을 이해한다
- Dockerfile을 작성해 이미지를 빌드한다

## 개념 정리
- 이미지는 읽기 전용 레이어의 쌓임, Dockerfile 한 줄이 대개 한 레이어
- 변경이 적은 명령을 위에, 자주 바뀌는 명령(소스 복사)을 아래에 두면 캐시 효율이 좋아진다

## 실습 1. 앱 준비
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

## 실습 2. Dockerfile
`app/Dockerfile`
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python", "app.py"]
```

`app/.dockerignore`
```
__pycache__
*.pyc
.git
```

## 실습 3. 빌드와 실행
```bash
cd app
docker build -t hello-py:1.0 .
docker run -d --name hello -p 8000:8000 hello-py:1.0
curl localhost:8000
docker history hello-py:1.0       # 레이어 확인
docker image inspect hello-py:1.0 | head -40
```

## 실습 4. 캐시 확인
1. `app.py` 문구를 수정
2. `docker build -t hello-py:1.1 .` 재빌드
3. 어떤 단계가 `CACHED`로 나오는지 확인

## 실습 5. CMD vs ENTRYPOINT
```bash
docker run --rm hello-py:1.0 python -c "print('override CMD')"
```
Dockerfile의 `CMD`를 `ENTRYPOINT ["python"]` + `CMD ["app.py"]`로 바꿔 동작 차이를 확인한다.

## 확인 질문
- [ ] `COPY`와 `ADD`의 차이는?
- [ ] `RUN`, `CMD`, `ENTRYPOINT`의 실행 시점은?
- [ ] 소스를 수정했는데 이전 레이어가 재사용되는 이유는?

## 정리
```bash
docker rm -f hello
```

## 메모
