---
title: "Docker swarm: 도커 스웜 사용후기 > 복수개의 컨테이너 손쉽게 관리하기"
excerpt_separator: <!--more-->
categories:
  - Docker
  - Swarm
tags: 
  - docker
  - server orchestration
---

### 도커 스웜 환경
- Versions
  1. Docker (18.09.0)
  2. Docker (1.23.2)
  
#### 많은 서버 오케스트레이션 툴 중에서 도커 스웜을 선정한 이유? 
1. 다른 툴을 설치할 필요가 없다.
2. 도커 명령어를 그대로 사용할 수 있다.
3. 쉽고 편리했다.
4. 안정성이 검증되어있다.
5. 입문으로 좋다.

### 알아야할 개념
- service
: 배포단위, 큰 틀

- node
: 스웜에 속해있는 서버
  - manager node
  : worker node를 관리하며, worker node에게 명령
  - worker node
  : 매니저 노드의 명령을 받아 일함
<!--more-->

---
### 원하는 결과 (해보려는 것)
- AS-IS
: 현재 사용중인 API서버가 있는데, 스펙이 나쁘지 않다.(AWS EC2 r5.large) 현재 아키텍처는 `EC2 target group -> ELB -> AutoScaling`으로 되어있는데, API서버에서 특정 데이터베이스를 커넥션할 때, 데이터베이스 서버가 Pending이 되는경우 timeout까지 커넥션을 물고있어서 WAS가 받아들이는 서버 커넥션수가 부족할 때가 있다. 그래서 컴퓨팅 자원을 전부 이용하지 못하고 오토스케일링이 전개되어 요금에 다소 영향력이 있는 상황이다.

- TO-BE
: 다 필요없고, 같은 스펙의 서버 1대로 버틸때까지 버텨보자.(물론 언젠간 스케일아웃 해야하겠지만)

**잠시만!** <br/>근본적인 데이터베이스쪽 팬딩을 해결하거나, Auto Scaling이 전개되는 설정을 바꾸거나, WAS단에서 커넥션 수를 조절(이미 진행)할 수도 있지만, 필자는 굳이 **`Docker swarm`**을 사용해보려고 한다.
{: .notice--danger}

| 항목 | AS-IS | TO-BE |
|-|-|-|
|컴퓨팅 자원사용률|20%|100%|
|서버 수|Auto scaling|1~n|
|상황|쓸데 없이 인스턴스가 늘어나고 줄어가고를 반복함|서버 한대, 복수컨테이너로 버텨보자|

---
### 도커 스웜/노드 설정하기
1. 스웜 초기설정
<br/>
컨픽서버로 사용할 서버에 도커를 설치해준 후, 하기 커멘드를 입력하면 해당 서버가 `manager node`인 swarm이 설정이 된다.
```bash
$ docker swarm init
Swarm initialized: current node (xxxxxxxxxxxxxxxxxx) is now a manager.
```

2. 스웜에 노드 추가
<br/>
나같은 경우는 서버 1대에 복수개의 컨테이너를 띄울 생각이나, 추후 서버 트레픽을 분산하기 위해 매니저노드와 자식노드를 추가하는 방법도 소개하겠다. 노드를 추가하려면 매니저노드에서 `join token`을 발급 받아야하는데, `--help`를 하면 알수 있겠지만 간단히 설명하겠다. 
- 매니저노드 토큰
: ```bash
$ docker swarm join-token manager
```
- 워커노드 토큰
: ```bash
$ docker swarm join-token worker
```
- 노드 확인
: ```bash
$ docker node ls
ID        HOSTNAME     STATUS    AVAILABILITY    MANAGER STATUS    ENGINE VERSION
xxxxx *   server-01     Ready     Active          Leader            18.09.0
```

**토큰을 발급 받으면?** <br/> command를 리턴해주는데 노드로 추가할 서버에 가서 해당 커맨드를 복붙해주면 된다. 또한 join이 실패할 경우에는, [다음 포스트](https://hidekuma.github.io/docker/swarm/docker-swarm-fail-join-node-as-worker/)를 참고해보면 도움이 될 수 있다.
{: .notice--info}

---
### 도커 서비스 생성하기
1. 서비스 생성
  - 노드들이 생성되었으니, swarm의 큰 틀인 서비스를 구축해보자. `name`과 `image`부분만 바꿔서 사용하면 된다. (`-p 80:80`은 80번 외부 포트를 컨테이너의 80번 포트로 포워딩 시켜준다는 뜻이다.)
  ```bash
  $ docker service create --name api -p 80:80 api:latest
  $ docker service create --name <name> -p 80:80 <image>:<image_version>>
  ```
  - 서비스 확인
  ```bash
  $ docker service ls
  ID      NAME    MODE           REPLICAS    IMAGE        PORTS
  xxxx    api     replicated     1/1         api:latest   *:80->80/tcp
  ```
2. 서비스 내 컨테이너 복제
  - 현재 한 서비스에 1개의 컨테이너가 떠 있는 상황인데, 복수개를 띄워보자.
  ```bash
  $ docker service scale api=3
  api scaled to 3 
  $ docker service ps api
  ID       NAME     IMAGE          NODE       DESIRED STATE  CURRENT STATE            
  xxx1     api.1    api:latest     server-01  Running        Running 3 minutes ago
  xxx2     api.2    api:latest     server-01  Running        Preparing 4 seconds ago
  xxx3     api.3    api:latest     server-01  Running        Preparing 4 seconds ago
  ```

3. 하나의 서버에서 3개의 컨테이너가 전개되었다.

**만약 노드가 복수개였다면?** <br/> 예를 들어, 노드(서버)가 3개였다면, 3개의 서버에 각각 하나의 컨테이너가 전개된다. 꼭 하나의 노드에 한 개의 컨테이너일 필요는 없다.
{: .notice--success}

---
### 도커 서비스 생성시, 컴포저 파일을 이용하고 싶다면?
1. stack을 이용한다.
  ```bash
  $ docker stack deploy -c ./docker-compose.yml --with-registry-auth swarm
  $ docker stack deploy -c <docker-compose.yml path> --with-registry-auth <service_name>
  ```
  - --with-registry-auth
  : 이 파라미터는 매우중요하다. 복수 노드가 있을경우, worker node에서 이미지 정보를 pull 할 때, image가 private라면 worker node에서도 docker login을 진행해야하는데 그러한 부분을 매니저노드에서 전달해주는 것(대신 해주는것)이라고 생각하면 된다.

2. stack update
  ```bash
  $ docker stack deploy/up -c ./docker-compose.yml --with-registry-auth swarm
  $ docker stack deploy/up -c <docker-compose.yml path> --with-registry-auth <service_name>
  ```
3. docker-compose.yml 예시
```yml
version: '3.7'
services:
    api:
      container_name: api
      image: api:latest
      sysctls:
        - net.core.somaxconn=4096
      ports:
        - '80:80'
        - '443:443'
      command: ./entrypoint.sh
      deploy:
        mode: replicated
        replicas: 10
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost"]
        interval: 1m30s
        timeout: 10s
        retries: 3
        start_period: 40s
```
여기서 중요한 것은, docker-compose 에서는 적용되던 container_name과 sysctls 인자는 swarm에서 이용할 수 없다. 따라서, **<U>connection수를 늘리려고 하면 container를 복수개 띄워주면 된다.</U>**
- `stack`을 이용한 서비스를 생성할 경우, 서비스 명은 나의 경우 swam_api로 생성된다.(1번 `<service_name>`이 프리픽스된다.)
- `healthcheck`를 통해, 각 컨테이너의 상태를 체크해줘야 다음에 말하는 Rolling Update가 문제없이 적용된다.
- `replicas: 10`을 통해 container를 10개 복제하였다.
- 서비스를 새롭게 업데이트 할 때, 자동으로 컨테이너별로 순차적업데이트를 진행한다.(도커 네트워크에서 컨테이너에 포워딩을 해줄 때, healthy한 컨테이너에만 포워딩을 해준다.)
- 즉 배포를 다운타임이 없이 진행할 수 있다. 하지만 모든 컨테이너에 배포되기까지, 배포 전/후 버전이 공존하는 상태가 된다.

**테스트결과**<br/>worker node에서는 sysctls인자가 먹는데, 커맨드라인을 입력한 manager node에서만 안먹는다. 도커에서 아직 지원하지 않는다고...
{: .notice--success}

---
### 결과적으로 나의 API서버는 어떻게 되었나?

#### AS-IS
![image-center]({{ '/images/post-imgs/api-before.png' | absolute_url }}){: .align-center}

최근 3일 중에서 가장 많이 쳤던 것은 5분에 12,000 정도였다. 오토 스케일링이 전개되면 많이 뜰 때는 약 6대까지 뜨니, 약 5분에 72,000정도 트래픽이 발생한다고 가정하자.

#### TO-BE
![image-center]({{ '/images/post-imgs/jmeter-api-test.png' | absolute_url }}){: .align-center}

Jmeter를 이용해서 스트레스 테스트를 진행해보았다.
- 테스트환경
: 500명의 유저(500 Threads)가 0.1 초당 한번씩 롱 커넥션을 무는 API를 1시간동안 접속한다.

![image-center]({{ '/images/post-imgs/jmeter-api-test2.png' | absolute_url }}){: .align-center}

계산해보면 5분에 약 300,000회 호출하는 셈인데, CPU 80%에서 잘 버텨주고있다.

![image-center]({{ '/images/post-imgs/api-vmstat.png' | absolute_url }}){: .align-center}

서버에 접속해서 실시간으로 찍어보니, 꽤 열심히 일하는 모습을 보여주었다.

**결과적으로** <br/> 동일 스펙 서버 1대를 혹사시켰지만, 충분히 현재 트래픽의 약 4배의 달하는 트래픽을 잘 버텨주었고, 서버가 더 힘들어하면, worker node를 붙여줘 손쉽게 scale out할 수 있다.
{: .notice--success}
