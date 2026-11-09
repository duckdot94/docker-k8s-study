# Week 1 - Day 4: Docker Compose

## 목표
- 여러 컨테이너를 하나의 YAML로 정의하고 실행한다

## 실습 1. 프로젝트 구성
```
day4-compose/
├── docker-compose.yml
└── app/
    ├── app.py
    ├── requirements.txt
    └── Dockerfile
```

`app/requirements.txt`
```
flask==3.0.3
redis==5.0.8
```

`app/app.py`
```python
import os
import redis
from flask import Flask

app = Flask(__name__)
r = redis.Redis(host=os.getenv("REDIS_HOST", "redis"), port=6379)

@app.route("/")
def index():
    count = r.incr("hits")
    return f"방문 횟수: {count}\n"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`app/Dockerfile`
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

`docker-compose.yml`
```yaml
services:
  web:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      REDIS_HOST: redis
    depends_on:
      - redis
  redis:
    image: redis:7
    volumes:
      - redisdata:/data

volumes:
  redisdata:
```

## 실습 2. 실행
```bash
docker compose up -d --build
docker compose ps
curl localhost:5000        # 여러 번 호출
docker compose logs -f web
```

## 실습 3. 영속성 확인
```bash
docker compose down        # 볼륨 유지
docker compose up -d
curl localhost:5000        # 카운트 이어짐
docker compose down -v     # 볼륨까지 삭제
```

## 실습 4. 추가 과제
- [ ] `redis`에 `healthcheck`를 추가하고 `depends_on.condition: service_healthy` 적용
- [ ] `.env` 파일로 환경변수 분리
- [ ] `docker compose up -d --scale web=2` 시도 후 포트 충돌 원인 확인

## 확인 질문
- [ ] Compose에서 서비스 이름으로 통신되는 이유는?
- [ ] `down`과 `down -v`의 차이는?
- [ ] `depends_on`은 준비 완료까지 보장하는가?

## 메모
