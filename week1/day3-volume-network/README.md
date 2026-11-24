# Week 1 - Day 3: 볼륨과 네트워크

> 상태: 완료 (2026-11-24) / 확인 질문 3개 모두 "모르겠다"에서 시작 → 설명 후 이해, 복습 때 다시 답해 보기

## 목표
- 컨테이너 데이터를 영속화한다
- 컨테이너 간 통신 방식을 이해한다

## 개념 정리
- 컨테이너의 쓰기 층은 컨테이너와 함께 사라진다. 남길 데이터는 컨테이너 밖(볼륨, 바인드 마운트)에 둔다
- **네임드 볼륨**: Docker가 관리하는 저장소. 컨테이너가 만들고 쓰는 데이터(DB 등)에 적합. Mac에서는 Docker VM 안(`/var/lib/docker/volumes/...`)이라 호스트에서 직접 못 연다
- **바인드 마운트**: 호스트의 내 경로를 그대로 연결. 내가 고치는 파일(소스, 설정, 정적 페이지)에 적합. `:ro`로 컨테이너의 쓰기를 막을 수 있다
- **`-p 호스트포트:컨테이너포트`**: 앞은 내 Mac에서 열리는 포트(다른 프로세스와 겹치면 안 됨), 뒤는 컨테이너 안 프로그램의 포트
- **사용자 정의 네트워크**는 내장 DNS가 있어 컨테이너 이름이 IP로 해석된다. 기본 `bridge`는 이름 해석이 없고 IP로만 접근 가능. 같은 네트워크 안 컨테이너끼리는 `-p` 없이 컨테이너 포트로 통신

## 실습 1. 데이터 유실 확인 [완료]
```bash
docker run -d --name pg1 -e POSTGRES_PASSWORD=pw postgres:16
docker exec -it pg1 psql -U postgres -c "CREATE TABLE t(id int);"
docker rm -f pg1
docker run -d --name pg1 -e POSTGRES_PASSWORD=pw postgres:16
docker exec -it pg1 psql -U postgres -c "\dt"     # 테이블 없음
docker rm -f pg1
```

## 실습 2. 네임드 볼륨 [완료]
```bash
docker volume create pgdata
docker run -d --name pg2 -e POSTGRES_PASSWORD=pw -v pgdata:/var/lib/postgresql/data postgres:16
docker exec -it pg2 psql -U postgres -c "CREATE TABLE t(id int);"
docker rm -f pg2
docker run -d --name pg2 -e POSTGRES_PASSWORD=pw -v pgdata:/var/lib/postgresql/data postgres:16
docker exec -it pg2 psql -U postgres -c "\dt"     # 테이블 유지
docker volume ls
docker volume inspect pgdata
docker rm -f pg2
```

## 실습 3. 바인드 마운트 [완료]
```bash
mkdir -p html && echo "<h1>bind mount</h1>" > html/index.html
docker run -d --name web -p 8081:80 -v $(pwd)/html:/usr/share/nginx/html:ro nginx
curl localhost:8081
echo "<h1>changed</h1>" > html/index.html
curl localhost:8081                                 # 즉시 반영
docker rm -f web
```
- 호스트 8080을 다른 Java 앱이 쓰고 있어서 `-p 8081:80`으로 바꿨다 (아래 메모 참고)
- `:ro`라서 컨테이너 안에서 `touch`하면 `Read-only file system`

## 실습 4. 사용자 정의 네트워크 [완료]
```bash
docker network create mynet
docker run -d --name server --network mynet nginx
docker run --rm --network mynet curlimages/curl curl -s http://server   # 이름으로 접근
docker network inspect mynet
```

기본 bridge 네트워크에서는 이름 해석이 안 되는지 확인해 본다.
```bash
docker run -d --name server2 nginx
docker run --rm curlimages/curl curl -s http://server2    # 실패 예상
```

| 네트워크 | 이름으로 접근 | IP로 접근 |
|---|---|---|
| 사용자 정의 `mynet` | 성공 | 성공 |
| 기본 `bridge` | 실패 (`exit=6`) | 성공 |

- IP 확인: `docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' server2`

## 확인 질문
- [x] 볼륨과 바인드 마운트는 언제 각각 쓰는가?
  - 모르겠다고 답함 → 설명 후 이해. 컨테이너가 만든 데이터 보존은 볼륨, 호스트 파일을 그대로 보여줄 때는 바인드 마운트
- [x] `-p 8080:80`에서 앞과 뒤 숫자의 의미는?
  - 모르겠다고 답함 → 설명 후 이해. 앞은 호스트(Mac) 포트, 뒤는 컨테이너 포트
- [x] 사용자 정의 네트워크에서 이름으로 통신되는 이유는?
  - 모르겠다고 답함 → 설명 후 이해. 내장 DNS가 컨테이너 이름을 IP로 해석

## 정리
```bash
docker rm -f server server2
docker network rm mynet
docker volume rm pgdata
```
- 실습 1의 `pg1`이 만든 익명 볼륨 2개도 `docker volume rm <해시>`로 삭제. `html/` 폴더도 삭제
- 컨테이너, 볼륨, 사용자 정의 네트워크는 모두 정리. 이미지(`postgres:16`, `nginx`, `curlimages/curl`)는 남겨 둠

## 메모

### 새로 알게 된 것
- `postgres` 이미지는 데이터 경로를 `VOLUME`으로 선언해서, `-v` 없이 실행해도 익명 볼륨이 자동 생성된다. `docker rm`으로 컨테이너를 지워도 볼륨은 남는다 (`docker volume ls`로 확인)
- 기본 bridge에서도 연결은 되고 이름 해석만 안 된다
- IP는 컨테이너를 다시 만들면 바뀔 수 있어서 이름 접근이 중요 (Day 4 Compose, Week 2 Service로 이어짐)

### 겪은 문제
- 호스트 8080을 다른 Java 앱(PID 32616)이 사용 중이라 `-p 8080:80`이 에러 없이 실행됐는데, `curl localhost:8080`은 그 앱의 페이지를 응답했다. `lsof -nP -iTCP:8080 -sTCP:LISTEN`으로 확인하고 `8081`로 변경 (Day 1과 같은 문제)
- `docker inspect -f '{{.NetworkSettings.IPAddress}}'`는 이 Docker 버전에서 오류. `.NetworkSettings.Networks.bridge.IPAddress` 사용

### 복습 거리
- [ ] 확인 질문 3개를 보지 않고 다시 답해 보기
