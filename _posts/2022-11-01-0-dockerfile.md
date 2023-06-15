---
layout: post
title:  "Dockerfile 명령어 메모"
date:   2022-11-01 00:01:03
categories: ETC
desc: "Dockerfile에 관련된 명령어들을 참고하여 기억할 만한 것들 메모 모아보기"
tags: dockerfile
cover: docker.png
---

#### FROM

```
FROM <이미지>
FROM <이미지>:<태그>
```

하나의 Docker 이미지는 base 이미지부터 시작해서 기존 이미지위에 새로운 이미지를 중첩해서 여러 단계의 이미지 층(layer)을 쌓아가며 만들어지며 최상단의 **FROM**절이 이미지의 base가 된다.

- Ubuntu 최신 버전을 base 이미지로 사용
```
FROM ubuntu:latest
```

- NodeJS 12를 base 이미지로 사용
```
FROM node:12
```

- Python 3.8 (alpine 리눅스 기반)을 base 이미지로 사용
```
FROM python:3.8-alpine
```

#### WORKDIR 

```
WORKDIR <이동할 경로>
```

쉘(shell)의 cd 명령문처럼 컨테이너 상에서 작업 디텍토리로 전환을 위해서 사용되며 **WORKDIR** 명령문으로 작업 디렉터리를 전환하면 그 이후에 등장하는 모든 *RUN, CMD, ENTRYPOINT, COPY, ADD* 명령문은 해당 디렉터리를 기준으로 실행된다.

#### RUN 

```
RUN ["<커맨드>", "<파라미터1>", "<파라미터2>"]
RUN <전체 커맨드>
```

**RUN**은 이미지 빌드 과정에서 필요한 커맨드를 실행하기 위해서 사용되며, 보통 npm등 패키지 설치에 사용하게 된다.

- npm 패키지 설치
```
RUN npm install --silent
```

- pip 패키지 설치
```
RUN pip install -r requirements.txt
```

#### ENTRYPOINT 

```
ENTRYPOINT ["<커맨드>", "<파라미터1>", "<파라미터2>"]
ENTRYPOINT <전체 커맨드>
```

이미지 실행시 항상 실행되야 하는 커맨드를 지정할 때 사용한다.

- npm start 스크립트 실행
```
ENTRYPOINT ["npm", "start"]
```

- Django 서버 실행
```
ENTRYPOINT ["python", "manage.py", "runserver"]
```

#### CMD 

```
CMD ["<커맨드>","<파라미터1>","<파라미터2>"]
CMD ["<파라미터1>","<파라미터2>"]
CMD <전체 커맨드>
```

이미지 실행시 default 명령어나, *ENTRYPOINT*에 넘길 기본 값을 정의한다.

```
ENTRYPOINT ["node"]
CMD ["index.js"]
```

- node index.js 실행
```bash
$ docker run test
```

- node main.js 실행
```bash
$ docker run test main.js
```

#### EXPOSE 

```
EXPOSE <포트>
EXPOSE <포트>/<프로토콜> (default : TCP)
```

네트워크 상에서 컨테이너로 들어오는 트래픽(traffic)을 리스닝(listening)하는 포트와 프로토콜를 지정하기 위해서 사용된다. 지정하더라도 호스트에서 접근시에는 -p를 이용하여 publish 해주어야 한다.

#### COPY/ADD

```
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

호스트 컴퓨터 파일을 이미지로 복사할 때 사용한다. WORKDIR 지정 시 상대경로로는 WORKDIR에 위치한 상태로 지정되는 것을 유의해야 한다.   
**ADD** 명령문은 **COPY**와 동일하나, 일반 파일 뿐만 아니라 압축 파일이나 네트워크 상의 파일도 사용할 수 있다.

### ENV

```
ENV <키> <값>
ENV <키>=<값>
```

이미지 환경 변수를 설정할 때 사용한다. 빌드 할 때와, 컨테이너 내에서도 사용 가능하다.

- NODE_ENV 환경 변수를 production으로 설정
```
ENV NODE_ENV production
```

### ARG

```
ARG <이름>
ARG <이름>=<기본 값>
```

docker build 커맨드로 이미지를 빌드 시, --build-arg 옵션을 통해 넘길 수 있는 인자를 정의하기 위해 사용한다.   

예를 들어, Dockerfile에 다음과 같이 ARG 명령문으로 port를 인자로 선언해주면,

```
ARG port # port=8080 (기본값 지정시)
```

다음과 같이 docker build 커맨드에 --build-arg 옵션에 port 값을 넘길 수가 있다.

```
$ docker build --build-arg port=8080 .
```

설정된 인자 값은 다음과 같이 ${인자명} 형태로 읽어서 사용할 수 있다.

```
CMD start.sh -h 127.0.0.1 -p ${port}
```

*ENV*와의 차이점은 빌드 내에서만 유효한 값이라는 점이다.

---

#### 요약

|명령어|용도|
|----|----|
|**FROM**|base 이미지 설정|
|**WORKDIR**|작업 디렉터리 설정|
|**RUN**|이미지 빌드 시 커맨드 실행|
|**ENTRYPOINT**|이미지 실행 시 항상 실행되야 하는 커맨드 설정|
|**CMD**|이미지 실행 시 디폴트 커맨드 또는 파라미터 설정|
|**EXPOSE**|컨테이너가 리스닝할 포트 및 프로토콜 설정|
|**COPY/ADD**|이미지의 파일 시스템으로 파일 또는 디렉터리 복사|
|**ENV**|환경 변수 설정|
|**ARG**|빌드 시 넘어올 수 있는 인자 설정|

   
[원본 참고 사이트][link]   
[공식 레퍼런스][link2]

---

#### 기타

**Window Docker Desktop과 WSL2 통합하기**


```powershell
wls -l -v # 사용중인 버전 확인

ex)
  NAME                   STATE           VERSION
* Alpine                 Running         1
  docker-desktop         Running         2
  Ubuntu-22.04           Running         1
  docker-desktop-data    Running         2

wsl --set-version Alpine 2 # 2버전으로 변경
wsl --set-version Ubuntu-22.04 2 # 2버전으로 변경
```

`Docker-desktop > General > Use the WSL 2 based engine`   
`Docker-desktop > Resources > WSL Integration > Enable intergration with my defauilt distro`

위에서 변경한 2버전 항목들이 목록으로 노출되며, 설정 포함시 통합된다.


[link]: https://www.daleseo.com/dockerfile/
[link2]: https://docs.docker.com/engine/reference/builder/
