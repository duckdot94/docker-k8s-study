# Week 1 - Day 3: 볼륨과 네트워크

## 목표
- 컨테이너 데이터를 영속화한다
- 컨테이너 간 통신 방식을 이해한다

## 실습 1. 데이터 유실 확인
```bash
docker run -d --name pg1 -e POSTGRES_PASSWORD=pw postgres:16
docker exec -it pg1 psql -U postgres -c "CREATE TABLE t(id int);"
docker rm -f pg1
docker run -d --name pg1 -e POSTGRES_PASSWORD=pw postgres:16
docker exec -it pg1 psql -U postgres -c "\dt"     # 테이블 없음
docker rm -f pg1
```

## 실습 2. 네임드 볼륨
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

## 실습 3. 바인드 마운트
```bash
mkdir -p html && echo "<h1>bind mount</h1>" > html/index.html
docker run -d --name web -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html:ro nginx
curl localhost:8080
echo "<h1>changed</h1>" > html/index.html
curl localhost:8080                                 # 즉시 반영
docker rm -f web
```

## 실습 4. 사용자 정의 네트워크
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

## 확인 질문
- [ ] 볼륨과 바인드 마운트는 언제 각각 쓰는가?
- [ ] `-p 8080:80`에서 앞과 뒤 숫자의 의미는?
- [ ] 사용자 정의 네트워크에서 이름으로 통신되는 이유는?

## 정리
```bash
docker rm -f server server2
docker network rm mynet
docker volume rm pgdata
```

## 메모
