# 도커 컴포즈

## 도커 컴포즈 설치

- 리눅스는 별도 설치 필요
- Windows, MacOS 는 도커 데스크탑에 포함

## 도커 컴포즈 기본

### docker-compose.yml 작성

- 실행 명령어를 docker-compose.yml 파일로 변환

```shell
docker run -d --name mysql \
alicek106/composetest:mysql \
mysqld

docker run -d -p 80:80 \
--link mysql:db --name web \
alicek106/composetest:web \
apachetctl -DFOREGR0UND
```
- docker-compose.yml
  - 들여쓰기할 시, Tab 은 인식하지 못하므로 2개의 스페이스를 사용한다.
```yaml
version: '3.0'
services:
  web:
    image: alicek106/composetest:web
    ports:
      - "80:80"
    links:
      - mysql:db
    command: apachectl -DFOREGROUND
  mysql:
    image: alicek106/composetest:mysql
    command: mysqld
```

