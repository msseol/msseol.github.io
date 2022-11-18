---
layout: post
title:  "Jenkins + Git 태그 기준으로 빌드하기"
date:   2022-11-08 23:00:08
categories: CICD
tags: jenkins git cicd tag
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Jenkins Git 태그 기준으로 빌드하기*

---

젠킨스를 Git에 연동해서 사용하는 경우 개발/QA/라이브 등 동일한 소스로 연동하게 되는데, 동일한 소스로 작업하다 보면 특정 환경에 배포할 때
불필요하거나, 아직 미개발 상태인 개발 작업물이 라이브에 배포될 수 있는 우려가 있다. 이럴 때를 대비하여 각 환경을 버전 태그로 관리하면 보다
쉽게 환경에 맞는 소스를 배포시킬 수 있다.

---

### 젠킨스 설정

Git + 젠킨스 연동 및 배포 설정은 미리 되어있다고 가정하고 설정을 시작한다.

**1. Git parameter 플러그인 설치**

Jenkins 관리 > 플러그인 항목에서 Git parameter 플러그인을 설치한다.

<a href="/assets/images/3.jpg" data-lightbox="falcon9-large" data-title="플러그인 설치">
  <img src="/assets/images/3.jpg" title="플러그인 설치">
</a>

**2. Job 설정 수정**

Job 설정에서 매개변수가 있는 빌드로 설정하고 다음과 같이 설정한다
Name은 아무 값이나 입력해도 된다. 태그 기준으로 배포할 것이기 때문에 Parameter Type은 Tag로 설정해준다.   
(Parameter Type에 보면 Branch, Revision 등 여러가지 옵션이 있음을 알 수 있다.)

> Name: tag   
> Parameter Type: Tag   
> Branch: */master   

<a href="/assets/images/4.jpg" data-lightbox="falcon9-large" data-title="매개변수 빌드로 설정 수정">
  <img src="/assets/images/4.jpg" title="매개변수 빌드로 설정 수정">
</a>


소스 코드 관리 부분에서도 Branches to Build를 위에서 설정한 Name 값으로 수정해준다. (해당 이름의 Git Parameter로 빌드하겠다는 의미)

<a href="/assets/images/5.jpg" data-lightbox="falcon9-large" data-title="태그 기준으로 빌드되도록 수정">
  <img src="/assets/images/5.jpg" title="태그 기준으로 빌드되도록 수정">
</a>

**3. 빌드 화면**

설정해놓은 Git repository에 Tag가 존재 시 빌드 화면에서 목록에 표기된다 (나의 경우는 태그가 없어서 안나옴)   
필요한 환경에 맞게 릴리즈한 태그를 선택해서 빌드하면 된다.

<a href="/assets/images/6.jpg" data-lightbox="falcon9-large" data-title="빌드 선택 화면">
  <img src="/assets/images/6.jpg" title="빌드 선택 화면">
</a>


**4. 빌드 확인 (jenkins console)**   

실제 샘플 태그를 릴리즈 한 뒤 젠킨스 잡을 돌려보면 정상적으로 태그를 패치해서 빌드하는 것을 볼 수 있다.   
(여기에선 v1.0.0으로 태깅)

```bash
Fetching upstream changes from https://github.com/seolminsu90/build-jenkins-sample.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
 > git fetch --tags --force --progress -- https://github.com/seolminsu90/build-jenkins-sample.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse origin/v1.0.0^{commit} # timeout=10
 > git rev-parse v1.0.0^{commit} # timeout=10
Checking out Revision bf1d6c48ef938fcd283f31583babd963553d3baa (v1.0.0)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f bf1d6c48ef938fcd283f31583babd963553d3baa # timeout=10
```

