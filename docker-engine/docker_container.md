# 컨테이너 생성, 실행, 정지, 삭제

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

# 컨테이너 노출

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

```shell
http://127.0.0.1:80/
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
