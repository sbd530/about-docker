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

