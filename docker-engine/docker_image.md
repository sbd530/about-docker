# 도커 이미지

- 도커 허브라는 중앙 이미지 저장소에서 이미지를 내려 받을 수 있다.
- 누가나 도커 허브에 이미지를 등록할 수 있으며, 이미지 저장소를 비공개로 설정할 수도 있다.

```shell
# 이미지 검색
docker search ubuntu
NAME                                                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   13460     [OK]
# STARS : 즐겨찾기 수
```

## 도커 이미지 생성

- 이미지로 만들 컨테이너 생성

```shell
docker run -i -t --name commit_test ubuntu:14.04
# 파일 생성
root@641dfbf65167:/# echo test_first! >> first
```
- 해당 컨테이너를 이미지로 만들기
  - 컨테이너 이름과 이미지 이름
  - `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
  - `-a` : author
  - `-m` : commit message  
    
- 저장소 이름은 없어도 되지만, TAG 가 없으면 자동으로 latest 로 설정 
```shell
docker commit \
-a "alicek106" -m "my first commit" \
commit_test \
commit_test:first

docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED         SIZE
commit_test                        first                   446d2f365dc0   7 minutes ago   197MB
ubuntu                             14.04                   13b66b487594   9 months ago    197MB
```
- 이미지로 새로운 이미지 생성
```shell
docker run -it --name commit_test2 commit_test:first

root@9acd9d96fcf2:/# echo test_second! >> second

docker commit \
-a "alicek106" -m "my second commit" \
commit_test2 \
commit_test:second

docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED          SIZE
commit_test                        second                  e9e6beade970   14 seconds ago   197MB
commit_test                        first                   446d2f365dc0   12 minutes ago   197MB
```

## 도커 이미지 구조

- 이미지를 새롭게 생성하여도 같은 사이즈의 이미지가 복사되어 저장되지 않고, 변경된 히스토리만 레이어에 추가하여 저장하게 된다.
- `history` 명령어로 레이어를 확인할 수 있다.
```shell
docker history commit_test:second

IMAGE          CREATED             CREATED BY                                      SIZE      COMMENT
e9e6beade970   About an hour ago   /bin/bash                                       13B       my second commit
446d2f365dc0   About an hour ago   /bin/bash                                       12B       my first commit
13b66b487594   9 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      9 months ago        /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>      9 months ago        /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B
<missing>      9 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   195kB
<missing>      9 months ago        /bin/sh -c #(nop) ADD file:276b5d943a4d284f8…   196MB
```
- 컨테이너에서 사용중인 이미지를 삭제하려 하면 에러 발생
```shell
docker rmi commit_test:first
Error response from daemon: conflict: unable to remove repository reference "commit_test:first" (must force) - container 9acd9d96fcf2 is using its referenced image 446d2f365dc0
```
- commit_test:first 를 삭제했었도, second 에서 해당 레이어를 참조 중이므로 레이어에 부여된 이름만 삭제한다.
```shell
docker stop commit_test2 && docker rm commit_test2
commit_test2
commit_test2
docker rmi commit_test:first
Untagged: commit_test:first # Untagged -> 이미지에 부여된 이름만 삭제
```
- Deleted 는 실제로 삭제된 레이어를 의미
- ubuntu 이미지는 삭제되지 않음.
```shell
docker rmi commit_test:second

Untagged: commit_test:second
Deleted: sha256:e9e6beade97095be044b73b7b83bf02133e33a52ead06d56b4371c0276ae3f37
Deleted: sha256:8b9d9f8ef014e9df7281f183c6fa2d934ff69aaf20d656d4c14d62d31874c9ad
Deleted: sha256:446d2f365dc0644dcfa8d5a84361b0f0e8d40a2f4ecb7e061f3642cd13d366c2
Deleted: sha256:dcb5605321d88f248f5b2d47aed140c8fd24458818312466026f310882178ceb
```
### 이미지 추출

- 이미지의 모든 메타데이터를 포함하여 하나의 파일로 추출 가능
  - `save -o [추출될 파일명]` 
```shell
docker save -o ubuntu_14_04.tar ubuntu:14.04
```

- 추출된 이미지를 로드하여, 이미지가 도커 엔진에 생성
  - `load -i [파일명]`
```shell
docker load -i ubuntu_14_04.tar
```

- `export` 로 이미지를 추출하면 이미지에 대한 설정 정보를 저장하지 않는다.
- `import` 로 로드하면 단일 파일로 저장. 이미지 용량을 버전별로 따로 차지하여 비효율적이다.
```shell
docker export -o rootFS.tar mycontainer
docker import rootFS.tar myimage:0.0
```

### 이미지 배포

- `save`같은 방식으로 추출 후 배포할 수 있지만, 파일 크기가 너무 크거나 도커 엔진이 많으면 파일로 배포하는데 한계가 있다.
- 도커의 이미지구조(레이어 형태)를 이용하지 않으므로 비효율적이다.
- 도커 허브 이미지 저장소로 push, pull 하여 간단하게 배포할 수 있다.
- 도커 사설 레지스트리에 직접 이미지 저장소를 만들 수 있다.

#### 도커 허브

```shell
docker run -i -t --name commit_container1 ubuntu:14.04
root@90ca70ee97b6:/# echo my first push >> test

# 이미지 이름의 접두어는 이미지가 저장되는 저장소 이름
docker commit commit_container1 my-image-name:0.0
sha256:1ef16a5dfe67ddcb51f16f098ee44f2cdf3e9511428dd18f269b3f8cc71de4de
# docker tag [기존이름] [새이름] : 이미지 이름 추가
docker tag my-image-name:0.0 alicek107/my-image-name:0.0

docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED         SIZE
alicek107/my-image-name            0.0                     1ef16a5dfe67   2 minutes ago   197MB
my-image-name                      0.0                     1ef16a5dfe67   2 minutes ago   197MB
# IMAGE ID 동일 : 같은 이미지를 가리키는 또다른 이름을 추가
```
- Docker Hub 에 로그인
```shell
docker login

Authenticating with existing credentials...
Login Succeeded
```
- push 하면 기존 ubuntu 의 레이어는 올라가지 않고, 추가적인 레이어만 push 됨
```shell
docker push sbd530/my-image-name:0.0
The push refers to repository [docker.io/sbd530/my-image-name]
584dd3955d27: Pushed
83109fa660b2: Pushed
30d3c4334a23: Pushed
f2fa9f4cf8fd: Pushed
0.0: digest: sha256:7a00f20b5cbc1152012181a6cce84d2e055c23fa02e1b23e9ab07a9faedae678 size: 1152
```
- 로그인 없이도 이미지를 내려받을 수 있다.
```shell
docker pull sbd530/my-image-name:0.0
```

### 도커 사설 레지스트리

- 개인 서버에 이미지 저장하는 레지스트리. 컨테이너로서 구현됨
