---
layout: post
title:  "Gitlab과 Jenkins 테스트 환경 구축하기"
date:   2022-12-02 00:01:00
categories: CICD
tags: jenkins gitlab docker deploy
---

내부적으로 젠킨스를 테스트 해야 할 일이 있었는데, 마음대로 만들고 수정하고 다운받고 하기에는 사내 사용하고 있는 젠킨스에서 하기에는 조금 꺼리는 부분이 있었다. 
아무래도 시스템 부하적인 부분도 그렇고 권한도 별로 없어, 플러그인 조차도 마음대로 설치하기가 조금 그런 환경이라 별도의 개발공간이 필요했다.   
   
각각 별도로 설치파일을 통해 설치하기에는 금방 사용하고 지우기가 번거로울 것으로 생각되어 Docker를 이용하여 간단하게 로컬 환경을 구축해서 테스트에 사용하기로 하였다.
   
> Windows Docker 기준으로 메모했으며, Docker를 설치하는 부분은 제외한다.

---

#### Docker Container 실행

**Jenkins 시작**

```bash
docker run --name jenkins-docker -d 
  -p 8080:8080 
  -p 50000:50000 
  -v C:/Users/minsu/test/gittest/jenkins:/var/jenkins_home 
  -u root 
  jenkins/jenkins:lts
```

**Gitlab 시작**

```bash
docker run --name gitlab --detach --hostname localhost 
  -p 443:443 
  -p 80:80 
  -p 22:22 
  --restart always 
  -v C:/Users/minsu/test/gittest/gitlab/config:/etc/gitlab 
  -v C:/Users/minsu/test/gittest/gitlab/logs:/var/log/gitlab 
  -v C:/Users/minsu/test/gittest/gitlab/data:/var/opt/gitlab 
  gitlab/gitlab-ce:latest
```

> 네이밍이나 볼륨 마운트 경로, 퍼블리싱 포트는 편한 것으로 지정하면 된다.

**네트워크 설정**

Jenkins에서 Gitlab에 대한 통신이 필요하여 컨테이너 간 통신을 위한 네트워크도 설정해준다. 위 구동 시 --network 옵션으로 설정할 수도 있다.

```bash

docker network create --driver=bridge jenkins-gitlab # 네트워크 생성
docker network ls # 네트워크 목록 보기
docker network connect jenkins-gitlab CONTAINTER_NAME # 네트워크에 연결

```

각 개방한 포트로 접근하여 정상적으로 접근되는 것을 확인한다.

#### Gitlab 및 Jenkins 로그인

**Gitlab**

root 계정의 초기비밀번호가 말이 다 다른데, 그냥 마음 편하게 직접 초기 비밀번호를 설정하는게 편하다. Gitlab이 실행된 컨테이너로 접속하여 초기 비밀번호를 직접 변경한다.

```bash
docker exec -it GITABL_CONTAINER_ID # 컨테이너 진입

gitlab-rails console -e production
> user = User.where(id: 1).first
> user.password='변경할비밀번호' # 8자 이상
> user.password_confirmation='변경할비밀번호'
> user.save
```

**Jenkins**

젠킨스의 경우 초기 구동하면서 logs를 확인하면 초기 비밀번호를 바로 알 수 있었다.   
만약 로그를 확인하지 않았다면 컨테이너 내에서 아래 경로의 InitialAdminPassword를 확인하면 된다.

<a href="/assets/images/15_6.jpg" data-lightbox="falcon9-large" data-title="Jenkins 초기 비밀번호 화면">
  <img src="/assets/images/15_6.jpg" title="Jenkins 초기 비밀번호 화면">
</a>
<em>그림1. Jenkins 초기 비밀번호 화면</em>

> 여기까지 하면 우선 서비스 설정은 끝난다.

---

#### Jenkins Gitlab 빌드 연동

이제 젠킨스 JOB에서 소스 코드 관리를 Gitlab으로 하기 위한 설정을 하는 단계이다.   
   
**1. Gitlab AccessToken 발급**

먼저 Gitlab에 접속해서 상단 프로필의 **User Settings**으로 접근한다.   
좌측 메뉴 중 **Access Token** 메뉴를 통해 신규 `Access Token`을 생성한다. *Select scopes*는 토큰에 담을 권한을 설정하는 영역인데, 테스트이므로 전체 다 체크해도 된다.

<a href="/assets/images/15_1.jpg" data-lightbox="falcon9-large" data-title="Gitlab Access Token 발급">
  <img src="/assets/images/15_1.jpg" title="Gitlab Access Token 발급">
</a>
<em>그림2. Gitlab Access Token 발급</em>

> 해당 Access Token으로 Jenkins에서 Gitlab 유저 인증을 처리한다.

**2. Jenkins Credentials 등록**

위에서 발급한 `Access Token`을 젠킨스에 등록하는 과정이다.   
대시보드에서 **Jenkins 관리**로 이동하여 스크롤해보면 Security 항목에 Manage 하는 영역이 있다.

<a href="/assets/images/15_2.jpg" data-lightbox="falcon9-large" data-title="Jenkins Credentials1">
  <img src="/assets/images/15_2.jpg" title="Jenkins Credentials1">
</a>
<em>그림3. Jenkins Credentials 1</em>

**Manage Credentials**로 이동하여 *global scope*를 선택하고, 아래와 같이 인증 정보를 생성한다.

<a href="/assets/images/15_3.jpg" data-lightbox="falcon9-large" data-title="Jenkins Credentials2">
  <img src="/assets/images/15_3.jpg" title="Jenkins Credentials2">
</a>
<em>그림4. Jenkins Credentials 2</em>

**3. Gitlab plugin 설치**

동일한 **Jenkins 관리** 영역에서 Plugin 관리 항목으로 이동하여 Gitlab 연동시 필요한 플러그인들을 설치한다.   
Git 연동에 Parameter를 사용할 예정이면 **Git Parameter**와 같은 플러그인을 추가로 설치해도 된다. 보통은 **Gitlab Plugin**만 설치해도 될 것이다. ([깃 태그 기준으로 젠킨스 빌드하기][tagbuild])


<a href="/assets/images/15_4.jpg" data-lightbox="falcon9-large" data-title="Jenkins gitlab 플러그인 설치">
  <img src="/assets/images/15_4.jpg" title="Jenkins gitlab 플러그인 설치">
</a>
<em>그림5. Jenkins gitlab 플러그인 설치</em>

**4. Gitlab 연동**

젠킨스 잡 생성 후 소스 코드 관리를 **Git**으로 선택하자. 여기서 *Repository URL*의 경우 보통은 사내 또는 외부에 설정된 Git 레포지토리의 주소지만, 테스트용 환경으로 도커에 구축했기 때문에 <span class="color1">Gitlab 도커 컨테이너 이름</span>(위에서 생성했던 이름)으로 호스트를 적으면 된다. *Credentials* 영역에는 위에서 생성한 인증 정보가 리스트업 되며 이제부턴 **Git**을 통해 소스코드를 관리하고 빌드할 수 있게 됬다.   


<a href="/assets/images/15_5.jpg" data-lightbox="falcon9-large" data-title="Job 설정">
  <img src="/assets/images/15_5.jpg" title="Job 설정">
</a>
<em>그림6. Jenkins Job 설정</em>


[tagbuild]: /cicd/2022/11/09/7-jenkinsgittag.html
