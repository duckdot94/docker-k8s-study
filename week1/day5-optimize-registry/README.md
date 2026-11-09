# Week 1 - Day 5: 최적화와 레지스트리

## 목표
- 멀티스테이지 빌드로 이미지 크기를 줄인다
- Docker Hub에 이미지를 push/pull 한다
- Apple Silicon 아키텍처 이슈를 이해한다

## 실습 1. Go 앱 준비
`main.go`
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello from Go")
	})
	http.ListenAndServe(":8080", nil)
}
```

`go.mod`
```
module hello

go 1.22
```

## 실습 2. 단일 스테이지 (비교용)
`Dockerfile.single`
```dockerfile
FROM golang:1.22
WORKDIR /src
COPY . .
RUN go build -o /app main.go
CMD ["/app"]
```

## 실습 3. 멀티스테이지
`Dockerfile`
```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app main.go

FROM gcr.io/distroless/static
COPY --from=build /app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```

```bash
docker build -f Dockerfile.single -t go-single .
docker build -t go-multi .
docker images | grep go-
docker run -d --name go -p 8080:8080 go-multi && curl localhost:8080
```
크기 차이를 아래에 기록한다.
- go-single: ___ MB
- go-multi: ___ MB

## 실습 4. 레지스트리
```bash
docker login
docker tag go-multi <사용자명>/go-hello:1.0
docker push <사용자명>/go-hello:1.0
docker rmi <사용자명>/go-hello:1.0
docker pull <사용자명>/go-hello:1.0
```

## 실습 5. 아키텍처 (Apple Silicon)
```bash
docker image inspect go-multi --format '{{.Architecture}}'    # arm64
docker run --rm --platform linux/amd64 alpine uname -m         # x86_64 (에뮬레이션)
docker buildx build --platform linux/amd64,linux/arm64 -t <사용자명>/go-hello:multi --push .
```

## 확인 질문
- [ ] 멀티스테이지가 이미지를 작게 만드는 원리는?
- [ ] 태그를 `latest`로만 운영하면 어떤 문제가 있는가?
- [ ] `exec format error`가 발생하는 원인은?

## 정리
```bash
docker rm -f go
```

## 메모
