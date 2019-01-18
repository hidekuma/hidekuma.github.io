---
title: "Docker swarm: 도커 스웜에서 워커/매니저 노드가 조인이 실패할 경우?"
excerpt_separator: <!--more-->
categories:
  - Docker
  - Swarm
tags: 
  - docker
  - server orchestration
---

## 도커 스웜 환경
- Versions
  1. Docker (18.09.0)
  2. Docker (1.23.2)

[저번 포스트](https://hidekuma.github.io/docker/swarm/docker-swarm/)에서 도커 스웜 사용후기에 대해 다뤘었는데, 이번에 실제로 클라우드 서비스를 이용해 도커 스웜을 구축할 때 마주했던 문제에 대해 공유하고자 한다.

---
# 답은 방화벽 문제이다. 답만 빠르게 알고 가고싶다면, 문서 맨 밑으로 가면 된다.

## 발생문제
1. 애초에 노드가 조인이 되지 않음.
2. 노드가 조인이 되었는데도 불구하고, 컨테이너 배포가 이루어지지 않음.

## 원인파악
사실 상기 발생 문제는 예상했던대로 방화벽의 문제였다. (AWS에서는 VPC와 Security group이 연관될 것이며, GCP에서는 방화벽규칙, NCP에서는 ACG에서 소정의 조치를 취해줘야한다.)

#### 스웜 설정
먼저 우리가 스웜을 설정할 때 하기와 같이 커맨드를 친다.
```bash
$ docker swarm init
```
이 경우 `advertise-addr`이 **내부 ip**로 자동으로 설정된다. 
각 노드(서버)가 같은 내부망 하에 있을경우는 문제없이 노드가 조인 될 것이나, 그렇지 않을경우 **퍼블릭 ip**로 해당 부분을 설정해주어야한다.
<!--more-->

보통 노드끼리 커넥션이 안 될경우, join 커맨드시 다음과 같은 에러가 발생한다.
```bash
$ docker swarm join --token SWMTKN-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx <private:ip>:2377

Error response from daemon: Timeout was reached before node joined. The attempt to join the swarm will continue in the background. Use the "docker info" command to see the current swarm status of your node.
Use the "docker info" command to see the current swarm status of your node.
```
여기까지 이해했다면, 자연스럽게 발생문제-1 (애초에 노드가 조인이 되지 않음)의 해결법이 감이 올 것이다.

#### 해결방법
1. 같은 내부망에 있을 경우
  - 방화벽에서 해당 서버의 tcp 2377 포트를 열어준다.

2. 같은 내부망에 있지 않을 경우
  - `advertise-addr`을 퍼블릭ip로 설정해준다.
  - 방화벽에서 해당 서버의 tcp 2377 포트를 열어준다.

#### 특정 ip로 도커 스웜 설정하기
스웜을 설정할 때, 아이피와 포트번호를 지정해주면 되며, 포트의 디폴트 값은 2377 포트이다. 
```bash
$ docker swarm init --advertise-addr <public_ip>
```
- advertise-addr
: Advertised address (format: <ip/interface>[:port])


**!! 주의 !!** <br/>해당 포트번호를 기본으로 가져가는것이 바람직하나, 피치못할 사정에 의해 바꿀경우에는 **2376** 포트를 제외하고 설정해야한다. **<U>2376은 docker-machine이 사용하는 포트</U>**이다. 
{: .notice--danger}

---

## 결과적으로
보통 1번 문제에 대해 인식을 하지 못했을 경우, 2번 문제(컨테이너 배포가 이루어지지 않을 경우)도 고스란히 발생한다.
해당 문제도 동일하게 방화벽 문제이다.

도커 공식 문서에 의하면 다음과 같이 방화벽을 열어줘야한다.
- TCP:2377
: 클러스터 매니지먼트(스웜 조인과 같은)에서 사용
- TCP/UDP:7946
: 노드간 통신
- UDP:4789
: 오버레이 네트워크 간 트래픽 통신

이렇게 해당 서버의 포트만 열어주면, 문제없이 스웜이 전개된다.

**Ingress network에서 암호화를 이용할 경우** <br/>즉, docker network에서 `--opt encrypted` 옵션을 사용할 경우에는 IP protocol 50 (ESP)가 열려있는지 확인해야한다.
{: .notice--info}
