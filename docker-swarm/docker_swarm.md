# 도커 스웜

## 도커 스웜을 사용하는 이유

- 하나의 호스트 머신에서 도커 엔진을 구동하는데, CPU나 메모리, 디스크같은 자원이 부족해질 수 있다.
- 이 때 더 좋은 성능의 장비를 새롭게 구매하는 것은 큰 비용이 따르고 비효율적인다.
- 여러 대의 서버를 클러스터로 만들어 자원을 병렬로 확장하는 방식이 사용된다.

## 스웜 클래식과 도커 스웜 모드

- 스웜 클래식과 스웜 모드는 여러대의 도커 서버를 하나의 클러스터로 만들어 컨테이너를 생성하는 기능을 제공한다.
- 도커 1.6버전 이후: 컨테이너로써의 스웜. 스웜 클래식. 여러 도커 서버에 단일 접근점 제공.
- 도커 1.12버전 이후: 스웜 모드. 마이크로서비스 아키텍쳐의 컨테이너를 다루기 위한 클러스터링 기능에 초점.

## 스웜 모드

- 별도 설치 없이 내장되어 있다.

```shell
docker info | grep Swarm
 Swarm: inactive
```

### 도커 스웜 모드 클러스터 구축

- 매니저 역할을 할 서버에서 스웜 클러스터를 시작한다.
  - `--advertise-addr` : 다른 서버가 매니저 노드에 접근하기 위한 IP 주소

```shell
docker swarm init --advertise-addr 172.18.0.1

Swarm initialized: current node (lu1bqj9fuixia65d89f13ihk6) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1dzuzmio3vq5c4y4y81wjn4grv9vdpdsf139ogewwiwsx22ew7-03kaztdkvmj3oqo6ggvyf112t 172.18.0.1:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

- 스웜 매니저는 기본적으로 2377 포트를 사용하고 노드 사이 통신에 7946 포트를 사용한다. 스웜이 사용하는 네트워크인 ingress 오버레이 네트워크에 4789 포트를 사용한다.

- 워커 노드를 추가한다.

```shell
docker swarm join --token SWMTKN-1-1dzuzmio3vq5c4y4y81wjn4grv9vdpdsf139ogewwiwsx22ew7-03kaztdkvmj3oqo6ggvyf112t 172.18.0.1:2377
```