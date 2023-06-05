---
layout: post
title:  "Docker로 RabbitMQ Mirror 클러스터 구성하기"
date:   2023-06-05 00:01:00
categories: SERVER
tags: docker rabbitmq cluster docker-compose dockerfile
cover: rabbit.png
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Docker를 이용하여 RabbitMQ 클러스터를 쉽게 구성해보자*

---

과거 *크롤러*를 개발할 때 **RabbitMQ**를 처음 사용해봤었던 것 같다. 순차적으로 작업을 수행하고 부하컨트롤을 할 필요가 있어 도입했는데 안정성 측면에서 큰 효과를 받았었다. 예전 *인앱푸시 서비스*를 개발할 때도 비슷한 목적으로 **푸시매니저**(메시지 적재) -< **푸시작업서버**(메시지 읽어 작업 수행) 형태로 분할하여 개발하기 위해 사용하기도 했다.   
요새는 MQ관련해서 사용할 일이 없어 기억이 가물가물 해지고 있어, 기억할 겸 다시 찾아보고 정리하려고 한다. 요즘에는 **Docker** 예시가 흔하니 Docker를 이용하여 클러스터링을 구성하는 예시를 작성해보겠다. (사실 돌려보는 거에 의의를 뒀다)

> Docker 설치 설정 등은 생략한다.

---

#### 1. 폴더 구조

```bash
ROOT
├─rabbitmq-node1
│   ├─data
│   │  └─ ...               # 데이터 볼륨 마운트
│   ├─logs
│   │  └─ ...               # 로그 볼륨 마운트
│   └─rabbitmq.conf         # 설정파일
├─rabbitmq-node2
│   ├─data
│   │  └─ ...               # 데이터 볼륨 마운트
│   ├─logs
│   │  └─ ...               # 로그 볼륨 마운트
│   └─rabbitmq.conf         # 설정파일
├─rabbitmq-node3
│   ├─data
│   │  └─ ...               # 데이터 볼륨 마운트
│   ├─logs
│   │  └─ ...               # 로그 볼륨 마운트
│   └─rabbitmq.conf         # 설정파일
└─docker-compose.yml
```

한개의 서버에서 3개의 Node로 구성하기 위해 **각각 설정 및 데이터 및 로그 폴더를 마운트**해줬다. 각각의 설정파일을 인스턴스에 적용하기 위해 설정 파일도 별도로 구분했다.

#### 2. 주요 설정 파일

<span class="text-primary">**docker-compose.yml**</span>

```yml
version: '3'

services:
  rabbitmq-node1:
    image: rabbitmq:3.11-management
    hostname: rabbitmq-node1
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      - RABBITMQ_ERLANG_COOKIE=my-erlang-cookie
    volumes:
      - ./rabbitmq-node1/data:/var/lib/rabbitmq
      - ./rabbitmq-node1/logs:/var/log/rabbitmq
      - ./rabbitmq-node1/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
    networks:
      - rabbitmq-cluster

  rabbitmq-node2:
    image: rabbitmq:3.11-management
    hostname: rabbitmq-node2
    ports:
      - 5673:5672
      - 15673:15672
    environment:
      - RABBITMQ_ERLANG_COOKIE=my-erlang-cookie
    volumes:
      - ./rabbitmq-node2/data:/var/lib/rabbitmq
      - ./rabbitmq-node2/logs:/var/log/rabbitmq
      - ./rabbitmq-node2/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
    networks:
      - rabbitmq-cluster

  rabbitmq-node3:
    image: rabbitmq:3.11-management
    hostname: rabbitmq-node3
    ports:
      - 5674:5672
      - 15674:15672
    environment:
      - RABBITMQ_ERLANG_COOKIE=my-erlang-cookie
    volumes:
      - ./rabbitmq-node3/data:/var/lib/rabbitmq
      - ./rabbitmq-node3/logs:/var/log/rabbitmq
      - ./rabbitmq-node3/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
    networks:
      - rabbitmq-cluster

networks:
  rabbitmq-cluster:
    driver: bridge

```

**volumes** 에서 각각 생성한 파일 및 폴더를 마운트했고, 동일한 **network**로 묶어놨다.   
`RABBITMQ_ERLANG_COOKIE=my-{erlang-cookie}`은 노드간 동일한 값으로 지정해야한다. (추후 버전에서 deprecated된다고 경고가 뜨고 있긴 한데..)

<span class="text-primary">**rabbitmq.conf**</span>

```yml
# 클러스터 구성
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@rabbitmq-node1
cluster_formation.classic_config.nodes.2 = rabbit@rabbitmq-node2
cluster_formation.classic_config.nodes.3 = rabbit@rabbitmq-node3

# 클러스터 유지
cluster_partition_handling = autoheal

# 동기화 관련
queue_master_locator=min-masters

loopback_users.guest=false
```

#### 구동

```
docker-compose up
```

아무 노드나 매니지 웹으로 접근해보자. (ex. localhost:15672)   
초기 게스트 아이디 비밀번호는 `guest/guest` 이다

> 접근이 안된다면 `docker exec {node} rabbitmq-plugins enable rabbitmq_management `를 입력하여 매니지웹을 활성화하자.


<a href="/assets/images/36_1.png" data-lightbox="falcon9-large" data-title="노드 확인">
  <img src="/assets/images/36_1.png" style="max-width:100%" title="노드 확인">
</a>
<em>그림1. 노드의 연결 상태 확인</em>

위와 같이 아무 노드에 접근하더라도 다른 클러스터 노드가 보인다면 성공이다.   
다만 이 상태에서는 Exchange나 Queue 등 데이터들이 복제되어 **동기화 되지 않는다.** Mirror 설정을 추가로 해야한다.


```bash
docker exec {allNode} rabbitmqctl set_policy ms-ha-all "^ha." '{"ha-mode":"all"}' # 1. ha.으로 시작하는 패턴
docker exec {allNode} rabbitmqctl set_policy ms-all "^" '{"ha-mode":"all"}' # 2. 모든 패턴
```

>  --apply-to exchanges.. queues.. 를 통해 적용할 대상을 설정할 수 있다.

각각의 노드에 원하는 큐 패턴에 맞게 HA 설정을 추가한다.  Management Web의 **Admin -> Policies -> Add / Update policies**에서도 설정할 수 있다. 아래를 참조하자

|Virtual Host|Name|Pattern|Apply to|Definition|Priority|
|---|---|---|---|---|---|
|/	|ha-all|^ha.	all	|ha-mode:	all|**blahblah**|0|

> docker exec로 할 경우 JSON오류가 나는 경우가 있는데 Container에 직접 접근해서 명령어를 입력하면 문제없이 잘 된다.

이제 Exchange나 Queue 등을 생성하거나, Data를 Publish 해보면 각각의 노드에 정상적으로 동기화 되는 것을 볼 수 있다.   
이렇게 Docker를 이용하여 간단하게 클러스터 환경을 구성해봤다.

#### Tip

클러스터 노드들의 상태를 모니터링 하기 위해서는 Grapana/Graphite 등 다양한 도구와 연동하는데 아래와 같은 Endpoint를 통해 Node 상태를 확인하고 장애알림 등을 연동하면 된다.

```bash
curl -u guest:guest -i http://localhost:15672/api/healthchecks/node
```
