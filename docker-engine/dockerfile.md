# Dockerfile

- 빌드 명령어를 제공.
- 컨테이터에 설치해야할 패키지, 추가할 소스코드, 실행할 명령어와 셸 스크립등 등을 하나의 파일에 기록.
- Git 같은 개발 도구를 통해 어플리케이션 빌드 및 배포를 자동화할 수 있음.

## Dockerfile 작성

```shell
mkdir dockerfile && cd dockerfile
echo test >> test.html
```
- FROM : 생성할 이미지의 베이스가 될 이미지
- MAINTAINER : 개발자 정보 (더이상 사용 안함. LABEL로 대체)
- LABEL : 이미지의 메타데이터
- RUN : 컨테이너 내부에서 실행할 명령어 (Y/N 을 물어보는 경우 -y 추가)
  - RUN ["executableFile", "명령줄 인자1", "명령줄 인자2", ...]
- ADD : 파일을 이미지에 추가
- WORKDIR : 명령어를 실행할 디렉터리 (cd /xxx/xxx)
- EXPOSE : 노출할 포트
- CMD : 컨테이너가 시작될 때마다 실행할 커맨드.
```shell
vi Dockerfile
```
```dockerfile
FROM ubuntu:14.04.1
MAINTAINER sbd530
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND # detached 모드로 생성
```

## Dockerfile 빌드

### 이미지 생성

```shell
# -t, --tag : 이미지 이름
# 맨끝(./) : Dockerfile 경로 (URL 도 가능)
docker build -t mybuild:0.0 ./

 => [internal] load build definition from Dockerfile                                                                                                                                                                               0.0s
 => => transferring dockerfile: 301B                                                                                                                                                                                               0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:14.04.1                                                                                                                                                                  2.5s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                                                                                                      0.0s
 => [1/6] FROM docker.io/library/ubuntu:14.04.1@sha256:2eb231f768446001a7cf9024ec9724cec23f6768e9ccb4e61499718d34621fbe                                                                                                           11.7s
 => => resolve docker.io/library/ubuntu:14.04.1@sha256:2eb231f768446001a7cf9024ec9724cec23f6768e9ccb4e61499718d34621fbe                                                                                                            0.0s
 => => extracting sha256:76a4cab4eb20ece7d3f09197088c9865343bcfa5b551221e67ed3e9b39800c6b                                                                                                                                          3.6s
 => => sha256:76a4cab4eb20ece7d3f09197088c9865343bcfa5b551221e67ed3e9b39800c6b 65.75MB / 65.75MB                                                                                                                                   0.0s
 => => sha256:d2ff49536f4de0a4ed21970313da88f4c0a1fe889513bdf08332bed821cc5f67 71.50kB / 71.50kB                                                                                                                                   0.0s
 => => sha256:f94adccdbb9cec09b3e64ec31aab2b07cbc020fe8f9151418ca6b7ede22273c0 682B / 682B                                                                                                                                         0.0s
 => => extracting sha256:d2ff49536f4de0a4ed21970313da88f4c0a1fe889513bdf08332bed821cc5f67                                                                                                                                          0.0s
 => => extracting sha256:f94adccdbb9cec09b3e64ec31aab2b07cbc020fe8f9151418ca6b7ede22273c0                                                                                                                                          0.0s
 => [internal] load build context                                                                                                                                                                                                  0.1s
 => => transferring context: 40B                                                                                                                                                                                                   0.0s
 => [2/6] RUN apt-get update                                                                                                                                                                                                     116.1s
 => [3/6] RUN apt-get install apache2 -y                                                                                                                                                                                          39.4s
 => [4/6] ADD test.html /var/www/html                                                                                                                                                                                              0.0s
 => [5/6] WORKDIR /var/www/html                                                                                                                                                                                                    0.0s
 => [6/6] RUN ["/bin/bash", "-c", "echo hello >> test2.html"]                                                                                                                                                                      0.4s
 => exporting to image                                                                                                                                                                                                             0.2s
 => => exporting layers                                                                                                                                                                                                            0.2s
 => => writing image sha256:9bdfc94dd1f30fbcd11d84e60174cb547aa9b3434f250dcfcf775082b4ce78c7                                                                                                                                       0.0s
 => => naming to docker.io/library/mybuild:0.0    
```

```shell
# -P : EXPOSE 의 모든 포트를 호스트에 연결
docker run -d -P --name myserver mybuild:0.0
558f3fb2c95a3a981ea2bf60d0418368926708f29653907b40068020d5c0ea4d

docker port myserver
80/tcp -> 0.0.0.0:49153
80/tcp -> :::49153

# http://localhost:49153/test.html
# http://localhost:49153/test2.html
```

```shell
docker images --filter "label=purpose=practice"
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mybuild      0.0       9bdfc94dd1f3   13 minutes ago   226MB
```

- 하위 디렉터리를 포함하여 전부 빌드하기 떄문에 .gitignore 처럼 .dockerignore 를 Dockerfile 이 있는 디렉터리에 위치시켜 배제할 수 있다.

```gitignore
# .dockerignore 예시
test2.html
*.html
*/*.html
test.htm?
```

- `ADD`, `RUN` 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성되고, 이를 이미지로 커밋한다. (이미지 레이어가  저장) 

### 멀티 스테이지를 이용한 Dockerfile 빌드하기

- 어플리케이션 빌드시 많은 의존성과 라이브러리가 필요하다.
- main.go 파일과 Dockerfile
```shell
package main
import "fmt"
func main() {
    fmt.Println("hello world");
}
```
```dockerfile
FROM    golang
ADD     main.go /root
WORKDIR /root
RUN     go build -o /root/mainApp /root/main.go
CMD     ["./mainApp"]
```
```shell
docker build . -t go_helloworld
```
- 각종 패키지, 라이브러리로 인해 단순한 프로그램을 실행하는 이미지이지만 이미지 크기는 매우 크다.
```shell
docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED          SIZE
go_helloworld                      latest                  3a102f0d5aa0   10 seconds ago   943MB
```
- 멀티 스테이지 빌드는 하나의 Dockerfile 에 여러개의 FROM 이미지를 정의하여 이미지 크기를 줄인다.
  - 첫번째 FROM에 사용된 이미지의 최종 상태에 존재하는 /root/mainApp 파일을 두번째 이미지인 alipine:latest에 복사
  - `--from=0` : 첫번째에서 빌드된 이미지의 최종 상태를 의미
  - alpine, busybox : ubuntu 같은 OS 이미지에 비해 크기가 매우 작고, 프로그램 실행에 필요한 필수적 런타임 요소가 포함된 리눅스 배포판 이미지
```dockerfile
FROM    golang
ADD     main.go /root
WORKDIR /root
RUN     go build -o /root/mainApp /root/main.go

FROM    alpine:latest
WORKDIR /root
COPY    --from=0 /root/mainApp .
CMD     ["./mainApp"]
```
- 동일한 역할의 이미지이지만, 이미지의 최종 사이즈가 크게 감소한다.
```shell
docker build . -t go_helloworld:multi-stage

docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED          SIZE
go_helloworld                      multi-stage             4c7001353512   27 seconds ago   7.35MB
```
- FROM 에 명시된 순서대로 0,1,... 순으로 차례대로 구분되어 사용된다.
- `as builder` : 특정 단계의 이미지에 별도의 이름을 정의할 수 있다.
```dockerfile
FROM    golang as builder
ADD     main.go /root
WORKDIR /root
RUN     go build -o /root/mainApp /root/main.go

FROM    golang as builder2
ADD     main2.go /root
WORKDIR /root
RUN     go build -o /root/mainApp2 /root/main2.go

FROM    alpine:latest
WORKDIR /root
COPY    --from=0 /root/mainApp .
COPY    --from=1 /root/mainApp2 .
```

### 기타 Dockerfile 명령어

- `ENV` : Dockerfile 에서 사용될 환경변수 지정. 
  - Dockerfile 뿐만 아니라 이미지에도 저장되므로, 컨테이너 내부에서도 사용할 수 있다.
  - `${ENV_NAME}` 또는 `ENV_NAME` 으로 사용한다.
  - `${ENV_NAME:-value}` : ENV_NAME 이라는 환경변수가 없다면 value 라는 문자열을 사용한다.
  - `${ENV_NAME:+value}` : ENV_NAME 이라는 환경변수가 있다면 value 라는 문자열을 사용하고, 없다면 빈 문자열을 사용한다.
```dockerfile
FROM    ubuntu:14.04
ENV     HOME /home
ENV     env1 my_env1
WORKDIR $HOME
RUN     touch $HOME/mytouchfile
RUN     / echo ${env1:-val} / ${env1:+val} / ${env2:-val} / ${env2:+val} /
#       /           my_env1 /          val /          val /              /
```
- `VOLUME` : 빌드된 이미지로 컨테이너 생성시, 호스트와 공유할 컨테이너 내부 디렉터리 설정.
```dockerfile
FROM   ubuntu:14.04
RUN    mkdir /home/volume
RUN    echo test >> /home/volume/testfile
VOLUME /home/volume
```
```shell
docker build -t myvolume:0.0 .
docker run -i -t -d --name volume_test myvolume:0.0
docker volume ls
```
- `ARG` : `build` 명령시 추가로 인자를 입력 받아 Dockerfile 내에서 사용될 변수 값을 설정.
  - 빌드시에 `--build-arg` 옵션으로 인자를 받는다.
  - `ENV` 로 동일한 변수명을 선언하면 `ENV` 변수로 덮어써진다.
```dockerfile
FROM ubuntu:14.04
ARG  my_arg
# 인자의 기본값 설정 가능
ARG  my_arg_2=value2 
RUN  touch ${my_arg}/mytouch
```
```shell
docker build --build-arg my_arg=/home -t myarg:0.0 ./
```
- `USER` : 컨테이너 내에서 사용될 계정명이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다.
  - 컨테이너 내부에서는 기본적으로 root 사용자를 설정한다.
```dockerfile
# ...
RUN  groupadd -r author && useradd -r -g author sbd530
USER sbd530
# ... 
```
- `ONBUILD` : 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile 로 생성될 때 실행할 명령어를 추가.
  - 자식 이미지에 상속할 때 실행될 명령어를 추가하므로써, 새로운 Dockerfile 을 심플하게 관리할 수 있다.
```dockerfile
FROM    ubuntu:14.04
RUN     echo "this is onbuild test!"
ONBUILD RUN echo "onbuild!" >> /onbuild_file
```
```dockerfile
# 메이븐 이미지의 Dockerfile 예시
FROM    mavern:3-jdk-8-alpine
RUN     mkdir -p /usr/src/app
WORKDIR /usr/src/app
ONBUILD ADD . /usr/src/app
ONBUILD RUN mvn insatall
```
- `STOPSIGNAL` : 컨테이너 정지시 사용될 시스템 콜의 종류를 지정
  - 기본은 SIGTERM
```dockerfile
FROM       ubuntu:14.04
STOPSIGNAL SIGKILL
```
- `HEALTHCHECK` : 컨테이너에서 동작하는 어플리케이션의 상태를 체크하도록 설정.
  - `docker ps` 의 `STATUS`에 healthy 로 표시된다.
  - `docker inspect` 출력중 State - Health - Log 항목에서 확인 가능하다.
```dockerfile
FROM        nginx
RUN         apt-get update -y && apt-get install curl -y
HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
# 1분마다 curl 실행해 nginx 상태를 체크
# 3초 이상 소요되면 한 번의 실패로 간주
# 3번 이상 타임아웃이 나면 컨테이너는 unhealty 상태
```
- `SHELL` : 별도로 사용할 셸을 지정.
  - 리눅스는 /bin/sh -c , 윈도우는 cmd /S /C 가 기본적으로 설정된다.
  - node 등의 별도의 셸을 사용할 경우 설정한다.
```dockerfile
FROM  node
RUN   echo hello, node!
SHELL ["/usr/local/bin/node"]
RUN   -v
# -v 는 node.js의 버전을 확인하는 명령
```
```shell
docker build ./ -t nodetest
Step 4/4 : RUN -v
v7.4.0
```
- `ADD` : 디렉터리에서 읽어들인 컨텍스트로부터 이미지에 파일을 복사. 외부 URL 도 가능
- `COPY` : 디렉터리에서 읽어들인 컨텍스트로부터 이미지에 파일을 복사. 로컬만 가능
  - 빌드 당시 `ADD` 를 하면 어떤 파일이 추가될지 알 수 없으므로 로컬에서 `COPY`로 복사하는 것이 안전하다. 

- `CMD` : 컨테이너가 시작될 때 실행할 명령어 설정. (docker run 커맨드와 같은 역할)
- `ENTRYPOINT` : 컨테이너가 시작될 때 실행할 명령어 설정. (커맨드를 인자로 받아 사용하는 스크립트 역할)
```dockerfile
ENTRYPOINT ["echo"]
CMD        ["hello", "world"]
# echo hello world
```
- Dockerfile 예시
```dockerfile
FROM       ubuntu:14.04
RUN        apt-get update
RUN        apt-get install apache2 -y
ADD        entrypoint.sh /entrypoint.sh
RUN        chmod +x /entrypoint.sh
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
# /bin/bash entrypoint.sh echo test
```
- entrypoint.sh 예시
```shell
echo $1 $2
apachectl -DFOREGROUND
```

### Dockerfile 빌드 시 주의점

- 여러개의 `RUN`보다는 하나의 `RUN` 에서 여러개의 명령어를 실행하도록 한다.
  - 예상치 못한 레이어 미스컨트롤을 방지한다.
```dockerfile
FROM ubuntu:14.04
RUN mkdir /test && \
fallocate -l 100m /test/dummy && \
rm /test/dummy
```
