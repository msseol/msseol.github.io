---
layout: post
title:  "Docker를 이용한 Elasticsearch, Logstash, Kibana 환경 구축"
date:   2022-11-25 12:01:00
categories: SERVER
tags: elasticsearch logstash kibana filebeat
cover: 14_1.jpg
side: true
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Docker compose로 로컬에 ELK 환경을 구성하는 과정 메모*

---

처음에는 금칙어 관련 처리를 **Elasticsearch**를 사용해서 했던 경험을 메모하고 다른 방안들에 대해서 기록하려 했었다. 그러다 보니 ES 환경을 로컬에 구성했어야 했는데, 한 김에 **Kibana**, **Logstash**, 그리고 **Filebeat**도 같이 구성해 보게 되었다. 개인적으로 Docker도 공부할 겸 정리 겸 간단하게 구성해 보았다.

---

#### Docker compose

윈도우 환경을 사용중이라 *windows docker*로 진행하였다. 하나 하나 설치하는 것보다 **docker**와 **docker-compose**를 이용하면 손쉬운 구성과 손쉬운 제거가 가능해서 주로 로컬 테스트 서비스는 **docker**로 자주 사용해왔다.

> 윈도우 도커 설치 및 가상화 등등 구성에 필요한 관련 설정에 대한 부분은 생략한다.

최종 **docker-compose.yml**를 먼저 확인해보겠다.

**docker-compose.yml**

```yml
version: '3.9'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic
  logstash:
    container_name: logstash
    image: docker.elastic.co/logstash/logstash:7.14.0
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    volumes:
      - logstash:/usr/share/logstash/data
      - ./conf/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
      - ./conf/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
    networks:
      - elastic
    depends_on:
      - es01
      - es02
      - es03
  kib01:
    image: docker.elastic.co/kibana/kibana:7.14.0
    container_name: kib01
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://es01:9200
      ELASTICSEARCH_HOSTS: http://es01:9200
    networks:
      - elastic
  filebeat:
    image: docker.elastic.co/beats/filebeat:7.14.0
    container_name: filebeat
    command: filebeat -e -strict.perms=false
    volumes: 
      - ./conf/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - filebeat:/usr/share/filebeat/data/
      - ./log/:/logs/host/
    networks:
      - elastic
volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local
  logstash:
    driver: local
  filebeat:
    driver: local # == docker volume create --driver local --name filebeat

networks:
  elastic:
    driver: bridge
```

버전은 7버전 중 잡히는 것 아무거나 하나를 사용하였다. 8버전 부터는 기본적으로 **Security** 모드가 활성화 되었어서 초기 인증 설정이 좀 번거로운 부분이 있어서 <span class="text-secondary">**7.14.0**</span>버전으로 진행했다. ([8버전 변경사항 보기][es8])   

해당 파일에는 아래와 같이 3개의 설정 파일을 마운트 해놓은 상태이며, 테스트 구성이기에 메모리는 적게 할당했다.

**1. logstash.conf**

```conf
input {
  beats {
    port => "5044"
  }
}

filter {
  # 다양한 변환 조건을 넣을 수 있음
}

output {
  elasticsearch {
    hosts => ['es01:9200']
    index => 'test-%{+YYYY.MM.dd}'
  }
}
```

> filebeat에서 곧장 elasticsearch로 전송해도 되지만 logstash를 통하면 filter의 기능을 통해 다양하게 변환이 가능하다.

**2. logstash.yml**
```yml
http.host: "0.0.0.0"
xpack.monitoring.enabled: false
xpack.monitoring.elasticsearch.hosts: ["https://es01:9200"] #...
# https://github.com/elastic/logstash/blob/main/config/logstash.yml
```

**3. filebeat.yml**
```yml
# Input 설정
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /logs/host/*.log

# Kibana 설정 정보
setup.kibana:
  host: "kib01:5601"
  protocol: "http"
setup:
  dashboards:
    enabled: true
    always_kibana: true #Only talk to Kibana, which is important for the retry
    retry:
      enabled: true #Retry in case Kibana is not up yet (★)

client_inactivity_timeout: 86400

# ES로 출력
#output.elasticsearch:
#  hosts: ["es01:9200"]
#  username: "elastic"
#  password:
 
# 별도 가공이 필요하면 logstash를 이용하는게 좋은듯?
# Sprin boot에 별도의 logback 로그 가공기도 있음

# Logstash 사용시
output.logstash:
  hosts: ["logstash:5044"]
```

**beat**의 경우 별도의 설정 없이 그냥 동작시킬 경우 **kibana**가 늦게 뜨는 관계로 실패하고 종료되버리는데, 중간에 지정한 <span class="text-danger">setup</span>옵션을 통해 자동으로 재구동 되도록 하여 **kibana**의 구동을 확인하게 한다.   
만약 **logstash**를 사용하지 않을땐 <span class="text-danger">output.elasticsearch</span> 설정으로 바로 전송할 수 있다.

#### 구동

```bash
docker-compose up
```

첫 구동시에는 ES 관련 가상메모리 매핑 한계 설정 변경이 필요하여 구동이 되지 않는데, 관련해서 별도로 설정을 변경해주어야 한다.

```bash
open powershell
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

#### 동작 확인하기

**Docker desktop**에서 확인하면 **docker-compose**로 구동한 이미지들이 그룹화되어 동작하는 것을 확인할 수 있다.

<a href="/assets/images/14_1.jpg" data-lightbox="falcon9-large" data-title="Docker Desktop 화면">
  <img src="/assets/images/14_1.jpg" title="Docker Desktop 화면">
</a>
<em>그림1. Docker Desktop 화면</em>

**filebeat.yml**에서 설정했던 <span class="text-danger">filebeat.inputs</span> 항목에서 지정한 *logs* 폴더 경로에 **.log*로 실제 텍스트 파일을 저장해보자. 별도의 `filter` 설정이 없어서 저장된 텍스트가 그대로 `message`에 들어가지만 저장 자체는 정상적으로 잘 되는 것을 볼 수 있다.   

#### Kibana 확인

<a href="/assets/images/14_2.jpg" data-lightbox="falcon9-large" data-title="Kibana 화면">
  <img src="/assets/images/14_2.jpg" title="Kibana 화면">
</a>
<em>그림2. Kibana에 test-* 인덱스에 Log가 적재된 화면</em>

#### 기타

<span class="text-success">*client_inactivity_timeout: 86400*</span>

처음 구성했을 때 **filebeat -> logstash** 전송이 되지 않고 **connection** 관련 <span class="text-danger">오류</span>가 계속 발생했다.   
**network** 구성 문제인가 싶었지만 정상적이었고 이것 저것 해보다 보니 **timeout** 관련 문제였고 해당 설정값을 통해 해결하게 되었다.


**구성하면서 참조한 사이트**

[https://tommypagy.tistory.com/343][A]   
[https://imjeongwoo.tistory.com/113][B]   
[https://coding-start.tistory.com/187][C]   

사용한 설정 파일 샘플은 [여기][git]에서 볼 수 있다.

[A]: https://tommypagy.tistory.com/343
[B]: https://imjeongwoo.tistory.com/113
[C]: https://coding-start.tistory.com/187
[es8]: https://www.elastic.co/kr/blog/whats-new-elastic-8-0-0
[git]: https://github.com/seolminsu90/elk-docker-compose-example