---
layout: post
title:  "Elasticsearch와 Mongodb 동기화하기"
date:   2022-11-20 00:00:01
categories: DATABASE
tags: elasticsearch mongodb monstache database
---


<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Mongodb + Elasticsearch로 구성된 프로젝트에서 데이터 동기화를 위해 설정했단 것들*

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem; color:red"></i><span class="color1">*오래전에 작성된 메모임을 감안*</span>

---


#### Monstache

Elasticsearch와 mongodb를 동기화하기 위해 몇가지 찾아본 것들을 먼저 나열해 보겠다.

|이름|방식|비고|
|---|---|---|
|**Logstash & logstash-input-mongodb**|insert데이터 동기화|수정/삭제도 동기화가 필요해서 제외|
|**mongolastic**|배치방식|java jar형태로 동작하는데 <span class="color1">실시간성이 떨어짐</span>|
|**mongo-connector**|?|es v1.3 수준의 <span class="color1">낮은버전만 지원</span>|
|**river-mongodb**|?|es v1.x 까지만 지원<span class="color1">(리뉴얼 전 사용하던 것)</span>|
|**monstache**|query log방식|<span class="color3">실시간, CUD도 가능하여 서비스에 적합하다 판단.</span>|

표에서 보듯이 기존의 사용하던 river를 더이상 사용할 수 없게 되면서 **monstache**가 **전체읽기/계속읽기/문서내용기반의 인덱스분리** 등 다양한 기능을 지원하여 가장 적합한 솔루션이라고 판단했다.   


[Monstache 홈페이지][monstache]

---

**실행** 

```bash
monstache <inline options>
# or
monstache -f <config toml file>
nohup ./monstache -f config.toml &
```

설정 자체도 설정 파일 내의 각 인스턴스 정보만 기입하면 되서 매우 심플한 편이다. monstache를 사용함으로서 mongodb에 저장된 커뮤니티 게시글을 es로 실시간 동기화하여, 검색이 필요한 항목만 별도로 es를 통한 검색으로 처리하는 식의 전략을 사용할 수 있었다.   
   
> 요새는 개인적으로 느끼기엔 Mongodb의 인기가 시들시들해진 것 같은데 그당시에만 해도 떠오르는 샛별마냥 자주 사용됬던 것 같다. 만약 다시금 비슷한 서비스를 구축해야 한다면 **RDB->MondoDB->ES**의 구조가 아닌 **RDB->ES** 구조로 사용할 것 같다. ES가 단순 검색엔진 보단 저장소 역할도 충분히 수행하게 됐다고 판단이 되어서다.


---

아래는 관련해서 es를 구성하면서 설정했던 몇가지 항목들에 대한 메모다.

#### Analyzer 플러그인 설치

**프록시 설정 ( 플러그인 다운 시 방화벽 문제 시)**

```bash
export 
ES_JAVA_OPTS="
-Dhttp.proxyHost=OTHER_HOST
-Dhttp.proxyPort=OTHER_PORT
-Dhttps.proxyHost=OTHER_HOST
-Dhttps.proxyPort=OTHER_PORT"
```

**설치 아날라이저 목록**

```bash
analysis-kuromoji # jp
analysis-nori # kr
analysis-smartcn # cn
```

> 그당시에 사용하던 아날라이저인데, 요새는 다른 아날라이저를 사용할수도 있다.
   
**데이터 경로, 로그 경로 설정**

설정 파일 중 elasticsearch.yml 내의 path.data, path.log 수정

*예시)*

```bash
path.data: /var/data
path.logs: /var/logs
```

가상메모리 설정 및 file descriptor

*예시)*

```bash
/etc/sysctl.conf
fs.file-max = 65536
vm.max_map_count = 262144
sudo sysctl -p #적용
```
 
/etc/security/limits.conf (추가)

```bash
{user} hard nofile 65536
{user} soft nofile 65536
/etc/pam.d/su (추가)
session    required   pam_limits.so
```

[참조사이트][essetup]

---

#### JVM 설정

**설정 파일 중 jvm.options 내의 heap space 영역 수정**

예시) - 보통 시스템 메모리의 50% 적용

```bash
-Xms16g
-Xmx16g
```

**클러스터 이름 설정**

설정 파일 중 **elasticsearch.yml** 내의 **cluster.name** 수정   
예시) 유니크한 값

```bash
cluster.name: hello-my-cluster
```

**지속성을 유지하기 위한 의미있는 노드 값 설정**

설정 파일 중 **elasticsearch.yml** 내의 **node.name** 수정   
예시) 의미있는 값

```bash
node.name: alphanode-1
```

**접속 호스트 설정**

설정 파일 중 **elasticsearch.yml** 내의 **network.host** 수정   
예시) 외부접속 IP 설정

```bash
network.host: 0.0.0.0
```


[essetup]: http://kugancity.tistory.com/entry/elasticsearch-51-관련-설정-변경
[monstache]: https://rwynn.github.io/monstache-site/