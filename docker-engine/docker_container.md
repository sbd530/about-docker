## 컨테이너 생성, 실행, 정지, 삭제

- 버전 확인

```shell
docker -v
```

- 컨테이너 생성 + 실행 + 내부 진입 (`-i` : 상호 입출력, `-t` : tty 활성화하여 bash 셸 사용)

```shell
docker run -i -t unbuntu:14.04
root@d3cb5a9a7087:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

- 컨테이너 정지 + 내부 탈출

```shell
root@d3cb5a9a7087:/# exit
exit
```

- 컨테이너 정지하지 않고 탈출 : `Ctrl` + `P` + `Q`

- 이미지 다운로드

```shell
docker pull centos:7
```

- 도커 엔진에 존재하는 이미지 목록 조회

```shell
docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED        SIZE
centos                             7                       eeb6ee3f44bd   3 months ago   204MB
ubuntu                             14.04                   13b66b487594   9 months ago   197MB
```

- 컨테이너 생성 (`--name` : 컨테이너명)
  - 16진수 컨테이너 고유 ID 반환. 일반적으로 앞 12자리 사용

```shell
docker create -i -t --name mycentos centos:7
194dce41c3780de9bea2e19efa5670e598d2158577ed71e033f4a340f501987d
```

- 컨테이너 실행

```shell
docker start mycentos
mycentos
```

```shell
docker start 194d
mycentos
```

- 컨테이너 내부 진입

```shell
docker attach mycentos
```

- 컨테이터 목록 조회

```shell
docker ps
CONTAINER ID   IMAGE      COMMAND       CREATED         STATUS         PORTS     NAMES
194dce41c378   centos:7   "/bin/bash"   5 minutes ago   Up 3 seconds             mycentos
```

- 컨테이터 목록 조회 (`-a` : 정지상태 포함한 모든 컨테이너 목록)
  - Up : 실행중. Exited : 정지상태

```shell
docker ps -a
CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS                      PORTS     NAMES
194dce41c378   centos:7                   "/bin/bash"              9 minutes ago    Up 4 minutes                          mycentos
8c43b752c376   ubuntu:14.04               "/bin/bash"              22 minutes ago   Exited (0) 18 minutes ago             pedantic_turing
```

- 컨테이너 이름 변경

```shell
docker rename pedantic_turing my_container
```

- 컨테이너 삭제 : 복구 불가, 실행중이면 삭제 불가

```shell
docker rm my_container
```

```shell
docker rm mycentos
Error response from daemon: You cannot remove a running container 194dce41c3780de9bea2e19efa5670e598d2158577ed71e033f4a340f501987d. Stop the container before attempting removal or force remove
```

- 컨테이너 정지 후 삭제

```shell
docker stop mycentos
docker rm mycentos
```

- 컨테이너 강제 삭제 : 실행중이어도 삭제

```shell
docker rm -f mycentos
```

- 모든 컨테이너 삭제

```shell
docker container prune
```

- 질의적으로 명령 (`-q` : 컨테이너 ID만 조회)

```shell
docker ps -a -q
d3cb5a9a7087
cd07c93cd846
84a9ee955a69
a7aae2745d34
3e3466b339cf

docker stop ${docker ps -a -q}
docker rm ${docker ps -a -q}
```

## 컨테이너 노출

- 컨테이너 가상 IP 주소 : 172.17.0.x의 IP를 순차적으로 할당
  - `172.17.0.2` : docker NAT IP, `127.0.0.1` : localhost
  - 설정이 따로 없으면, 도커가 설치된 호스트에서만 해당 컨테이너에 접근 가능.

```shell
docker run -i -t --name network_test ubuntu:14.04
root@a5ee4537f9e9:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:9 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:766 (766.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

- 웹서버 컨테이너 생성
  - `-p ([ip of host]:)[port of host]:[port of container]` : 컨테이너의 포트를 호스트의 포트와 바인딩
  - `-p [ph1]:[pc1] -p [ph2]:[pc2]` : 다수의 포트를 외부에 개방

```shell
docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
```

```shell
docker run -i -t -p 3306:3306 -p 192.168.0.100:7777:80 ubuntu:14.04
```

- 셸로 컨테이너에 아파치 서버 설치

```shell
root@b981264cad6e:/# apt-get update
root@b981264cad6e:/# apt-get install apache2 -y
root@b981264cad6e:/# service apache2 start
```

- 외부에서 컨테이너 접근
  - `[ip of host]:80/`
  - 호스트 80번포트로 접근 &rarr; 컨테이너의 80번포트로 포워딩 &rarr; 서버 접근

```shell
curl -X GET http://127.0.0.1:80/
```

## 컨테이너 어플리케이션 구축

- App 컨테이너와 DB 컨테이너를 생성하여 연동
  
- RDB 생성, 실행
  - Windows cmd 는 `^`로 개행 가능
```shell
docker run -d \
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mysql:5.7
```
- Wordpress 생성, 실행
  - `-i -t` : attach 가능한 상태로 설정
  - `-d` : Detached 모드로 컨테이너 실행. 백그라운드에서 동작하도록 설정. 입출력 X (bash 불가)
  - `-p 80` : 컨테이터 80포트와 호스트의 포트 중 하나를 바인딩
```shell
docker run -d \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_USER=root \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress \
--link wordpressdb:mysql \
-p 80
wordpress
```
- 호스트와 바인딩된 포트 확인
  - `컨테이너 포트 -> 호스트 포트`
```shell
docker port wordpress
80/tcp -> 0.0.0.0:57280
```

```shell
http://localhost:57280/
```
- `-d` 옵션
  - 컨테이너 내부에서 프로그램이 터미널을 차지하는 포그라운드로 실행된다.
  - 터미널을 차지하는 포그라운드로써 동작하는 프로그램이 없으면 컨테이너는 시작되지 않는다.
  - mysql 이미지는 컨테이너 실행시, 포그라운드로 실행되는 mysqld 가 동작하기 때문에 `-d` 를 준다.
```shell
docker run -d --name detach_test ubuntu:14.04
docker ps -a
CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS                      PORTS                   NAMES
829aa4dd7c58   ubuntu:14.04               "/bin/bash"              7 seconds ago    Exited (0) 6 seconds ago                            detach_test
```
- `-e` 옵션 : 컨테이너 환경변수 설정
```shell
...
-e MYSQL_ROOT_PASSWORD=password
...
```

```shell
# ... -e MYSQL_ROOT_PASSWORD=password ...
docker exec -i -t wordpressdb /bin/bash
root@ce22516465db:/# echo $MYSQL_ROOT_PASSWORD
password
```
- `-d`때문에 입출력이 불가능 -> `exec -i -t` 명령어로 내부에서 명령어를 실행하여 반환 받을 수 있다.
  - bash 에 `exit` 명령을 줘도 컨테이너가 정지되지 않은 채 빠져나온다. 
```shell
docker exec wordpressdb ls /
bin
boot
dev
docker-entrypoint-initdb.d
...
```

```shell
docker exec -i -t wordpressdb /bin/bash
root@ce22516465db:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36 MySQL Community Server (GPL)

...

mysql>
```
- `--link` 옵션
  - 내부 IP 알 필요 없이 항상 alias 로 접근하도록 설정.
```shell
...
--link wordpressdb:mysql
...
```

```shell
docker exec wordpress curl mysql:3306 --silent
```
- link 가 걸려있다는 것은 컨테이너 실행 순서의 의존성도 정의
```shell
docker stop wordpress wordpressdb
docker start wordpress
Error response from daemon: Cannot link to a non running container: /wordpressdb AS /wordpress/mysql
Error: failed to start containers: wordpress
```

## 도커 볼륨

- 볼륨 
  - mysql 컨테이너 등에 저장한 데이터는 컨테이너 삭제와 함께 사라진다.
  - 도커 볼륨을 통해 영속성을 줄 수 있다.
  - 볼륨을 통해 호스트와 볼륨을 공유하고, 볼륨 컨테이너를 활용하고, 도커가 관리하는 볼륨을 생성할 수 있다.

### 호스트 볼륨 공유

- DB, WAS 컨테이너 생성
  - `-v [hostDirectory]:[containerDirectory]` : 서로 디렉터리 공유
```shell
docker run -d \
--name wordpressdb_hostvolume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7
```

```shell
docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress_hostvolume \
--link wordpressdb_hostvolume:mysql \
-p 80 \
wordpress
```
- 컨테이너 삭제 후 호스트 확인
```shell
ls /home/wordpress_db
docker rm wordpress_hostvolume wordpressdb_hostvolume
ls /home/wordpress_db
```
- 역으로 호스트 디렉터리에 파일이 존재할 경우, 볼륨 생성시 모든 하위 디렉터리를 마운트한다.

### 볼륨 컨테이너

- 볼륨 공유하기
  - `--volumes-from`
```shell
docker run -i -t \
--name volumes_from_container \
--volumes-from volume_override \
ubuntu:14.04
```

```text
                        --volume           --volumes-from
[host-volume] <---> [volume-container] <---> [container1]
                                       <---> [container2]
```

### 도커 볼륨

- 도커 볼륨 생성
```shell
docker volume create --name myvolume
```
- 생성된 볼륨 조회
```shell
docker volume ls
DRIVER    VOLUME NAME
local     myvolume
```
- 볼륨을 사용하는 컨테이너 생성
  - `[볼륨명]:[컨테이너의 공유 디렉터리]`
  - `/root` 디렉터리에 파일쓰기
```shell
docker run -i -t --name myvolume_1 \
-v myvolume:/root/ \
ubuntu:14.04

root@a2f018f53171:/# echo hello, volume! >> /root/volume
```
- 다른 컨테이너 생성시에 볼륨을 공유하면 동일한 `volume`파일 존재
```shell
docker run -i -t --name myvolume_2 \
-v myvolume:/root/ \
ubuntu:14.04

root@b1ceef2356b4:/# cat /root/volume
hello, volume!
```

```shell
[myvolume] <---> [container1]
           <---> [container2]
```
- `docker inspect` : 볼륨 저장 위치 등 조회
  - `docker inspect --type [구성요소타입] [구성요소이름]`
```shell
docker inspect --type volume myvolume
# docker [volume | image | container] inspect myvolume
[
    {
        "CreatedAt": "2022-01-03T13:54:33Z",
        "Driver": "local", # 볼륨이 쓰는 드라이버
        "Labels": {}, # 볼륨을 구분하는 라벨
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data", # 볼륨이 저장된 호스트의 저장위치
        "Name": "myvolume",
        "Options": {},
        "Scope": "local"
    }
]

ls /var/lib/docker/volumes/myvolume/_data
volume
```
- `docker volume create` 명령없이 `-v` 옵션으로 수행 가능.
```shell
docker run -i -t --name volume_auto \
-v /root \
ubuntu:14.04

docker container inspect volume_auto
```
- 볼륨 전체 삭제
```shell
docker volume prune

WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
...
Total reclaimed space: 671.4MB
```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```