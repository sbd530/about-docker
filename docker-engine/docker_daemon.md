# 도커 데몬

## 도커의 구조

```shell
which docker
# for WindowsOs
# where docker
/usr/bin/docker
```
- 명령어는 /usr/bin/docker 에서 실행되지만 도커 엔진의 프로세스는 /usr/bin/dockerd 파일로 실행됨
  - docker 명령어는 엔진이 아니라 클라이언트로서의 도커
```shell
ps aux | grep docker
# for WindowsOs
# tasklist | find "docker"
```

- 도커 데몬 : 도커 프로세스가 실행되어 서버로서 입력 받을 준비가 된 상태
- 도커 클라이언트 : API를 사용할 수 있도록 CLI를 제공
- docker 명령시 클라이언트는 /var/run/docker.sock 에 위치한 소켓을 통해 도커 데몬의 API 를 호출한다.
- 아무런 설정이 없을 시 도커 데몬을 제어하는 순서

1. 사용자가 docker version 등의 도커 명령어를 입력한다.

2. /usr/bin/docker 는 /var/run/docker.sock 유닉스 소켓을 사용해 도커 데몬에게 명령어를 전달한다.

3. 도커 데몬은 명령어를 파싱하고 명령어에 해당하는 작업을 수행한다.

4. 수행결과를 도커 클라이언트에게 반환하고 사용자에게 결과를 출력한다.

## 도커 데몬 실행

- 도커 데몬 실행, 정지
```shell
service docker start
service docker stop
```

- 레드햇 계열 OS 에서도커 자동 실행 설정
```shell
systemctl enable docker
```

- `dockerd` 로 도커 데몬 실행 (foreground 로 실행)

```shell
service docker stop
dockerd
```

## 도커 데몬 설정

- `dockerd` 명령어의 옵션들은 설정 파일의 `DOCKER_OPTS` 에 입력하면 된다.

### 도커 데몬 제어: `-H`

- `-H` 옵션 : 도커 데몬의 API 를 사용할 수 있는 방법을 추가

```shell
# 위와 아래는 동일한 명령
dockerd
dockerd -H unix://var/run/docker.sock
```
- 호스트의 모든 네트워크 인터페이스 카드에 할당된 IP 주소와 특정 포트로 도커 데몬을 제어
```shell
dockerd -H unix://var/run/docker.sock -H tcp://0.0.0.0:2375
```
- 원격 호스트에서 Remote API 를 사용하도록 설정
```shell
dockerd -H tcp://192.168.99.100:2375
```
```shell
curl 192.168.99.100:2375/version --silent | python -m json.tool
{
  "ApiVersion: "1.39",
  ...
}
```
- `-H` 옵션으로 제어할 원격 도커 데몬을 설정
```shell
docker -H tcp://192.168.99.10:2375 version
```

### 도커 데몬에 보안 적용: `--tlsverify`

- 도커 데몬 측 파일 : ca.pem, server-cert.pem, server-key.pem
- 도커 클라이언트 측 파일 : ca.pem, cert.pem, key.pem

#### 1. 서버 측 파일 생성

- 인증서에 사용될 키를 생성
```shell
mkdir keys && cd keys
openssl genrsa -aes256 -out ca-key.pem 4906
Generating RSA private key, 4906 bit long modulus (2 primes)
............................................................++++
...............................................................................................................................................................................................................................................................................................................................++++
e is 65537 (0x010001)
Enter pass phrase for ca-key.pem:
Verifying - Enter pass phrase for ca-key.pem:
# 비밀번호 입력(docker)
```
- public key 를 생성한다.
```shell
openssl req -new -x509 -days 10000 -key ca-key.pem -sha256 -out ca.pem
Enter pass phrase for ca-key.pem:
# 위에서 입력한 비밀번호 입력
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
# 공백으로 엔터쳐도 된다.
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```
- 서버 측에서 사용될 키를 생성한다.
```shell
openssl genrsa -out server-key.pem 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
...................................................++++
...................................++++
e is 65537 (0x010001)
```
- 서버 측에서 사용될 인증서를 위한 인증 요청서 파일을 생성한다. $HOST 부분에 사용중인 도커 호스트의 IP 나 도메인을 입력한다.
```shell
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```
- 접속에 사용될 IP 주소를 extfile.cnf 파일로 저장한다.
```shell
echo subjectAltName = IP:$HOST,IP:127.0.0.1 > extfile.cnf
```
- 서버측의 인증서 파일을 생성한다.
```shell
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
Signature ok
subject=CN = 192.168.55.217
Getting CA Private Key
Enter pass phrase for ca-key.pem:
# 비밀번호 입력
```

#### 2. 클라이언트 측에서 사용할 파일 생성

- 클라이언트 측의 키 파일과 인증 요청 파일을 생성하고, extfile.cnf 파일에 extendedKeyUsage 항목을 추가한다.

```shell
openssl genrsa -out key.pem 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
......++++
..................................................................................................................++++
e is 65537 (0x010001)
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile.cnf
```

- 클라이언트 측의 인증서를 생성한다.

```shell
openssl x509 -req -days 30000 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
```

- 다음같이 파일이 만들어졌느지 확인한다.

```shell
ls
ca-key.pem  ca.srl    client.csr   key.pem          server-key.pem
ca.pem      cert.pem  extfile.cnf  server-cert.pem  server.csr
```

- 생성된 파일의 쓰기 권한을 삭제해 읽기 전용 파일로 만든다.

```shell
chmod -v 0400 ca-key.pem key.pem server-key.pem ca.pem server-cert.pem cert.pem
```

- 도커 데몬의 설정파일이 존재하는 ~/.docker 로 도커 데몬 측에서 필요한 파일을 옮긴다. 필수는 아니지만 필요한 파일을 모아두면 관리가 용이하다.

```shell
cp {ca,server-cert,server-key,cert,key}.pem ~/.docker
```

- TLS 보안을 적용하여 도커를 실행한다.

```shell
dockerd --tlsverify \
-- tlscacert=/root/.docker/ca.pem \
--tlscert=/root/.docker/server-cert.pem \
--tlskey=/root/.docker/server-key.pem \
-H=0.0.0.0:2376 \
-H unix:///var/run/docker.sock
```

- 다른 도커 호스테이서 도커 클라이언트에 -H 를 추가해 보안이 적용된 도커를 제어한다.

```shell
docker -H 192.168.55.217:2376 \
-- tlscacert=/root/.docker/ca.pem \
--tlscert=/root/.docker/cert.pem \
--tlskey=/root/.docker/key.pem \
--tlsverify verison
```

- 모든 명령어마다 위같은 설정을 하긴 귀찮으므로, 환경변수를 저장한다.
  - `export` 는 셸이 종료되면 사라진다.

```shell
# 도커 데몬 인증 파일 위치
export DOCKER_CERT_PATH="/root/.docker"
# TLS 인증을 사용할지를 설정
export DOCKER_TLS_VERIFY=1

docker -H 192.168.55.217:2376 version
```

```shell
# 환경변수 항상 적용
vi ~/.bashrc
...
export DOCKER_CERT_PATH="/root/.docker"
export DOCKER_TLS_VERIFY=1
...
```

- curl 로 Remote API 를 사용하려면 다음같은 플래그를 추가한다.

```shell
curl https://192.168.55.217/version \
--cert ~/.docker/cert.pem \
--key ~/.docker/key.pem \
--cacert ~/.docker/ca.pem
```

### 도커 스토리지 드라이버 변경: ---storage-diver

## 도커 데몬 모니터링

### 도커 데몬 디버그 모드

- 도커 데몬을 디버그 옵셥으로 실행

```shell
dockerd -D
```

### events stats, system df

**events**

- 도커 데몬에 어떤 일이 일어나고 있는지 실시간 로그를 조회

```shell
docker events
docker system events
```

- 이벤트 발생

```shell
docker pull ubuntu:14.04
```

```shell
docker events
2022-01-15T18:08:41.991529400+09:00 image pull ubuntu:14.04 (name=ubuntu)

```

- events 명령어에 필터 추가

```shell
docker events --filter 'type=image'
```

**stats**

- 실행 중인 모든 컨테이너의 자원 사용량을 스트림으로 출력
  - 컨테이너들의 CPU, 메모리 제한 및 사용량, 네트워크 입출력, 블록 입출력 정보

```shell
docker stats
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT   MEM %     NET I/O       BLOCK I/O   PIDS
1e78c2ed39f5   myregistry   0.01%     5.246MiB / 25GiB    0.02%     1.66kB / 0B   0B / 0B     10
```

- 스트림이 아닌, 한 번만 출력

```shell
docker stats --no-stream
```

**system df**

- 사용중인 이미지, 컨테이너, 로컬 볼륨의 총 개수 및 사용 중인 개수, 크기, 삭제 후 확보 가능한 공간을 출력

```shell
docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          26        13        8.125GB   5.209GB (64%)
Containers      40        1         49.9MB    49.9MB (100%)
Local Volumes   12        12        999.5MB   0B (0%)
Build Cache     84        0         396.7MB   396.7MB
```

### CAdvisor

- 구글이 만든 컨테이너 모니터링 도구
- 컨테이너로서 설치하고, 컨테이너 별 실시간 자원 사용량 및 도커 모니터링 정보를 시각화
- 오픈소스. 깃허브, 도커허브.
- 도커데몬의 정보를 가져올 수 있는 호스트의 모든 디렉터리를 CAdvisor 컨테이너에 볼륨으로서 마운트.

```shell
docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker.ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
--name=cadvisor \
google/cadvisor:latest
```

### Remote API 라이브러리를 이용한 도커 사용

- 도커를 제어하고 싶을 경우 Remote API 를 래핑해서 사용할 수 있다.
- 다양한 언어의 라이브러리를 오픈소스로 사용할 수 있다.

