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

docker container inspect volume_auto # 볼륨 이름은 임의로 생성
```
- `-v` 대신, 키밸류로 표현하는`--mount` 옵션 사용가능
```shell
docker run -i -t --name mount_option_1 \
--mount type=volume,source=myvolume,target=/root \
ubuntu:14.04
```
- `--mount`로 호스트 디렉터리를 내부에 마운트하는 경우 `type=bind`로 지정
```shell
docker run -i -t --name mount_option_2 \
--mount type=bind,source=/home/wordpress_db,target=/home/testdir \
ubuntu:14.04
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

- ***Stateless Container*** 
  - 컨테이너가 삭제돼도 데이터는 보존
  - 컨테이너 자체는 영속적 데이터와 무관 -> 상태가 없음 (Stateless)
  - 상태를 결정하는 데이터는 외부로부터 제공받는다.
  - 도커 사용시에 반드시 선택해야할 아키텍쳐
  - 컨테이너 자체에 데이터를 보관 -> Stateful

## 도커 네트워크

### 브리지 네트워크 : 컨테이너 생성시 자동으로 연결되는 bridge0 브리지를 활용하도록 설정됨.

- 172.17.0.x IP 대역을 순차적으로 컨테이너에 할당
```shell
docker network ls
ETWORK ID     NAME                  DRIVER    SCOPE
86a77a687a23   bridge                bridge    local
393f89059b36   host                  host      local
6f5994cf9b86   none                  null      local
```

```shell
docker network inspect bridge
[
    {
        "Name": "bridge",
        ...
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        ...
        "Containers": {},
        ...
    }
]
```
- 새로운 브리지 네트워크 생성
```shell
docker network create --driver bridge mybridge
```
- `--net` 옵션 : 컨테이너가 이 네트워크를 사용하도록 설정
  - 새로운 IP 대역 할당 (172.20.0.2)
```shell
docker run -i -t --name mynetwork_container \
--net mybridge \
ubuntu:14.04

root@6192af63fffd:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:14:00:02
          inet addr:172.20.0.2  Bcast:172.20.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:13 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1102 (1.1 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
- 네트워크 해제 및 연결
```shell
docker network disconnect mybridge mynetwork_container
docker network connect mybridge mynetwork_container
```
- 서브넷, 게이트웨이, IP 할당 범위 등을 임의로 설정
```shell
docker network create --driver=bridge \
--subnet=172.72.0.0/16 \
--ip-range=172.72.0.0/24 \
--gateway=172.72.0.1 
my_custom_network
```

### 호스트 네트워크
- 호스트의 네트워크 환경을 그대로 사용 가능.
```shell
docker run -i -t --name network_host \
--net host \
ubuntu:14.04

root@docker-desktop:/#
```
- 호스트와 동일한 네트워크
  - 아파치 웹서버를 컨테이너에 띄우면 기본 포트인 80으로 바로 접근 가능해진다.
```shell
root@docker-desktop:/# ifconfig
br-37d8ffa457c0 Link encap:Ethernet  HWaddr 02:42:a7:44:c3:6c
          inet addr:172.19.0.1  Bcast:172.19.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
...
```
### 논 네트워크
- 네트워크를 사용하지 않음.
```shell
docker run -i -t --name network_none \
--net none \
ubuntu:14.04
```
- 로컬호스트 외에는 네트워크가 존재하지 않음.
```shell
root@437fc7345616:/# ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
### 컨테이너 네트워크
- `--net container:[다른 컨테이너 ID]` : 다른 컨테이터의 네트워크 환경 공유 
  - 내부 IP를 새로 할당받지 않는다.
  - 호스트에 veth 로 시작하는 가상 네트워크 인터페이스도 생성되지 않는다.
```shell
# -i -t -d : 내부에서 셸을 실행하지만 내부로 들어가지 않음.
docker run -i -t -d --name network_container_1 ubuntu:14.04
75eba45311220df6ec66f36f5000bbae88d34bdf7508e693d70440d05e37e4e6

docker run -i -t -d --name network_container_2 \
--net container:network_container_1 \
ubuntu:14.04
7b6fe649f30e5390261812909fa8bfb4ab7ba4285e1d1f64d3b066af741f84cf
```
- 두 컨테이너의 eth0 정보가 완전히 동일
```shell
docker exec network_container_1 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1382 (1.3 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

docker exec network_container_2 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1382 (1.3 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
### 브리지 네트워크와 --net-alias
- 특정 호스트 이름으로 컨테이너 여러 개에 접근 가능
```shell
docker run -i -t -d --name network_alias_container_1 \
--net mybridge \
--net-alias alicek106 \
ubuntu:14.04

docker run -i -t -d --name network_alias_container_2 \
--net mybridge \
--net-alias alicek106 \
ubuntu:14.04

docker run -i -t -d --name network_alias_container_3 \
--net mybridge \
--net-alias alicek106 \
ubuntu:14.04
```

```shell
docker inspect network_alias_container_1 | grep IPAddress
"IPAddress": "172.20.0.3"
docker inspect network_alias_container_2 | grep IPAddress
"IPAddress": "172.20.0.4"
docker inspect network_alias_container_3 | grep IPAddress
"IPAddress": "172.20.0.5"
```
- 컨테이너 3개의 IP로 각각 ping 전달됨
  - 라운드로빈 방식으로 IP를 결정
  - alicek106라는 설정한 호스트 이름을 --net-alias 옵션으로 alicek106로 설정한 컨테이너로 resolve 한다.
```shell
docker run -i -t --name network_alias_ping \
--net mybridge \
ubuntu:14.04

root@01fa5f357f1f:/#  ping -c 1 alicek106
...
64 bytes from network_alias_container_1.mybridge (172.20.0.2): icmp_seq=1 ttl=64 time=0.072 ms

root@01fa5f357f1f:/#  ping -c 1 alicek106
...
64 bytes from network_alias_container_2.mybridge (172.20.0.4): icmp_seq=1 ttl=64 time=0.071 ms

root@01fa5f357f1f:/#  ping -c 1 alicek106
...
64 bytes from network_alias_container_3.mybridge (172.20.0.3): icmp_seq=1 ttl=64 time=0.087 ms
```

```shell
[도커내장DNS] -> [ping alicek106] -> [--net-alias=alicek106]
                                -> [--net-alias=alicek106]
                                -> [--net-alias=alicek106]
```
- dig : DNS 로 도메인 이름에 대응하는 IP 조회하는 툴
```shell
# network_alias_ping 내부
apt-get update
apt-get install dnsutils
# dig 명령어 반복하면 반환되는 IP 리스트 순서가 바뀐다
dig alicek106

alicek106.              600     IN      A       172.20.0.3
alicek106.              600     IN      A       172.20.0.4
alicek106.              600     IN      A       172.20.0.2
```

### MacVLAN 네트워크

- MacVLAN : 호스트 네이워크 인터페이스 카드를 가상화하여 물리 네트워크를 컨테이너에게 동일하게 제공
```shell
공유기 네트워크 : 192.168.0.0/24
서버 1 (node01) : 192.168.0.50
서버 2 (node02) : 192.168.0.51
```

```shell
# node01
docker network create -d macvlan --subnet-192.168.0.0/24 \
--ip-range=192.168.0.64/28 --gateway=192.168.0.1 \
-o macvlan_mode=bridge -o parent=eth0 my_macvlan

#node02
docker network create -d macvlan --subnet-192.168.0.0/24 \
--ip-range=192.168.0.128/28 --gateway=192.168.0.1 \
-o macvlan_mode=bridge -o parent=eth0 my_macvlan
```
- MacVLAN 네트워크를 사용하는 컨테이너 생성
```shell
docker run -it --name c1 --hostname c1 \
--network my_vacvlan ubuntu:14.04

ip a
```

```shell
docker run -it --name c2 --hostname c2 \
--network my_vacvlan ubuntu:14.04

ip a
```

## 컨테이너 로깅
### json-file 로그
```shell
# -d 백그라운드 모드로 컨테이너 생성
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
```
- 컨테이너 로그 확인
```shell
docker logs mysql
```

```shell
# -e 환경변수 옵션 없이 생성
docker run -d --name no_passwd_mysql mysql:5.7
docker ps
# no_passwd_mysql 조회되지 않음
docker start no_passwd_mysql
docker ps
# 마찬가지로 no_passwd_mysql 조회되지 않음
docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"
CONTAINER ID   STATUS         PORTS                 NAMES
1ed09db73aa1   Up 8 minutes   3306/tcp, 33060/tcp   mysql
```
- docker logs 로 문제를 확인
  - `--since [UNIX TIME]`
  - `-t` : 타임스탬프
  - `-f` : 로그를 스트림으로 출력 (실시간 로그)
```shell
# docker logs --since 1474765879 mysql
# docker logs no_passwd_mysql
# docker logs --tail 5 no_passwd_mysql
# docker logs --tail 5 -t no_passwd_mysql
docker logs --tail 5 -f -t no_passwd_mysql

2022-01-08T08:14:19.217116400Z 2022-01-08 08:14:19+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
2022-01-08T08:14:19.217136000Z     You need to specify one of the following:
2022-01-08T08:14:19.217138700Z     - MYSQL_ROOT_PASSWORD
2022-01-08T08:14:19.217140700Z     - MYSQL_ALLOW_EMPTY_PASSWORD
2022-01-08T08:14:19.217142700Z     - MYSQL_RANDOM_ROOT_PASSWORD

# ^C로 탈출
```
- `-i -t` 옵션으로 입출력 설정한 컨테이너의 입출력 내용도 확인 가능
```shell
docker run -i -t --name logstest ubuntu:14.04
# internal env
root@785bb4031c07:/# echo test!
test!
```
```shell
docker logs logstest
root@785bb4031c07:/# echo test!
test!
```
- 기본적으로 컨테이너 로그는 JSON 형태로 도커 내부에 저장된다.
```shell
cat /var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```
- 로그 파일이 커지면 호스트 저장공간을 계속 차지하게 된다.
  - 파일 최대 크기와 파일 개수 지정
```shell
docker run -it \
--log-opt max-size=10k --log-opt max-file=3 \
--name log-test ubuntu:14.04
```
- 기본 로깅 드라이버는 json-file
- 도커 데몬 시작 옵션에서 --log-driver 옵션으로 로깅 드라이버 변경 가능
```shell
DOCKER_OPTS="--log-driver=syslog"

DOCKER_OPTS="--log-opt max-size=10k --log-opt max-file=3"
```

### syslog 로그

```shell
docker run -d --name syslog_container \
--log-driver=syslog \
ubuntu:14.04
echo syslogtest
```
- syslog 로깅 드라이버는 기본적으로 로컬호스트의 syslog 에 저장
```shell
# ubuntu OS example
tail /var/log/syslog
```
- syslog 를 리모트 서버에 설치하고 로그 옵션을 추가하여 로그를 리모트 서버로 전송 가능
  - `rsyslog`
```shell
# 서버 호스트 : 192.168.55.217
# 클라이언트 호스트 : 192.168.55.217
docker run -i -t \
-h rsyslog \
--name rsyslog_server \
-p 514:514 -p 514:514/udp \
ubuntu:14.04
```
- rsyslog 설정에서 주석 처리를 없애고 rsyslog 재시작
```shell
root@rsyslog:/etc# vi /etc/rsyslog.conf
...
# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
...
```
```shell
root@rsyslog:/# service rsyslog restart
 * Stopping enhanced syslogd rsyslogd                               [ OK ]
 * Starting enhanced syslogd rsyslogd                               [ OK ]
```
```shell
docker run -i -t \
--log-driver=syslog \
--log-opt syslog-address=tcp://192.168.55.217:514 \
--log-opt tag="mylog" \
ubuntu:14.04
```
```shell
root@rsyslog:/# tail /var/log/syslog
```
- --log-opt 옵션으로 syslog-facility 를 쓰면 로그 저장 파일을 변경할 수 있다.
  - 기본적으로는 daemon 으로 설정
```shell
docker run -i -t \
--log-driver syslog \
--log-opt syslog-address=tcp://192.168.55.217:514 \
--log-opt tag="maillog" \
-- log-opt syslog-facility="mail" \ 
ubuntu:14.04
```
- rsyslog 서버 컨테이너에 새로운 로그 파일이 생성됨.
```shell
root@rsyslog:/# cd /var/log/
mail.log
```

### fluentd 로깅

- JSON 데이터 포맷
- AWS S3, HDFS, MongoDB 등 다양한 저장소에 저장 가능

```shell
# 도커 서버 : 192.168.55.217
# fluentd 서버 : 192.168.55.217
# MongoDB 서버 : 192.168.55.217
# docker pull mongo
docker run --name mongoDB -d -p 27017:27017 mongo
```
- fluent.conf 파일 저장 : 로그 설정 정보
```shell
<source>
  @type forward
</source>

<match docker.**>
  @type mongo
  database nginx
  collection access
  host 192.168.55.217
  port 27017
  flush_interval 10s
  # user alicek106
  # password mypw
</match>
```
- fluent.conf 파일이 저장된 디렉터리에서 실행
```shell
# Dockerhub 의 fluentd 이미지에는 mongo 에 연결하는 플러그인이 없음
# -> alicek106/fluentd:mongo
docker run -d --name fluentd -p 24224:24224 \
-v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf \
-e FLUENTD_CONF=fluent.conf \
alicek106/fluentd:mongo
```
- 로그를 남기는 임의의 컨테이너 생성
```shell
docker run -p 80:80 -d \
--log-driver=fluentd \
--log-opt fluentd-address=192.168.55.217:24224 \
--log-opt tag=docker.nginx.webserver \
nginx
```
- MongoDB 에 저장된 로그 조회
```shell
docker exec -it mongoDB mongo
# MongoDB shell
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
nginx   0.000GB
> use nginx
switched to db nginx
> show collections
access
> db['access'].find()
...
{
  "_id" : ObjectId("61d962bdc98aa9000a539942"), 
  "log" : "2022/01/08 10:08:51 [notice] 1#1: using the \"epoll\" event method", 
  "container_id" : "1ad5872b94a1dbd997be9f3bc015b28d23d6f32e5c57f60f13f986408373dcbd", 
  "container_name" : "/priceless_payne", "source" : "stderr", 
  "time" : ISODate("2022-01-08T10:08:51Z")
}
...
```
### 아마존 클라우드워치 로그 (필요시)

1. 클라우드워치에 해당하는 IAM 권한 생성
2. 로그 그룹 생성
3. 로그 그룹에 로그 스트림 생성
4. 클라우드워치의 IAM 권한을 사용할 수 있는 EC2 인스턴스 생성과 로그 전송

## 컨테이너 자원 할당 제한

```shell
docker inspect rsyslog_server
...
"HostConfig": {
  ...
  "KernelMemory": 0,
  "KernelMemoryTCP": 0,
  "MemoryReservation": 0,
  "MemorySwap": 0,
  "MemorySwappiness": null,
  "OomKillDisable": false,
  "PidsLimit": null,
  "Ulimits": null,
  "CpuCount": 0,
  "CpuPercent": 0,
  "IOMaximumIOps": 0,
  "IOMaximumBandwidth": 0,
  ...
}
...
```
- `run` 명령에서 설정한 자원 제한 변경 
```shell
docker update [변경할 자원제한] [컨테이너 이름]
```

### 컨테이너 메모리 제한

- 최소 메모리는 4MB. 
- 너무 작게 잡으면 메모리가 부족하여 컨테이너가 실행되지 않는다.

```shell
# m = megabyte, g = gigabyte
docker run -d --memory="1g" --name memory_1g nginx
docker inspect memory_1g | grep \"Memory\"
# (windows) docker inspect memory_1g | find "Memory"
            "Memory": 1073741824,
```
- 기본적으로 Swap 메모리는 메모리의 2배로 설정.
- 사용자 지정도 가능
```shell
docker run -it --name swap_500m \
--memory=200m \
--memory-swap=500m
ubuntu:14.04
```

### 컨테이너 CPU 제한

#### --cpu-shares

- 시스템에 존재하는 CPU 를 어느 비중만큼 share 할 것인지 명시
  - `1024` : CPU 할당에서 1의 비중
```shell
docker run -i -t --name cpu_share \
--cpu-shares 1024 \
ubuntu:14.04
docker run -i -t --name cpu_share \
--cpu-shares 512 \
ubuntu:14.04
# 2 : 1 의 비율. 66%, 33%
ps aux
```

#### --cpuset-cpu

- 호스트에 CPU 가 복수개 있을 때 컨테이너가 특정 CPU 만 사용하도록 설정

```shell
docker run -d --name cpuset_2 \
--cpuset-cpus=2 \
alicek106/stress \
stress --cpu 1
```
