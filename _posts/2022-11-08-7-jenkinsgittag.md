---
layout: post
title:  "Jenkins + Git 태그 기준으로 빌드하기"
date:   2022-11-08 23:00:08
categories: CICD
tags: jenkins git cicd tag
description: Jenkins Git 태그 기준으로 빌드하기
---

<i class="fa-solid fa-check"></i> *Git에 등록된 태그 기준으로 jenkins를 빌드하게 하는 과정 메모*

---

### 젠킨스 설정

**1. Git parameter 플러그인 설치**

Jenkins 관리 > 플러그인 항목에서 Git parameter 플러그인을 설치한다.

<a href="/assets/images/3.jpg" data-lightbox="falcon9-large" data-title="플러그인 설치">
  <img src="/assets/images/3.jpg" title="플러그인 설치">
</a>

**2. Job 설정 수정**

Job 설정에서 매개변수가 있는 빌드로 설정하고 다음과 같이 설정한다

> Name: tag   
> Parameter Type: Tag   
> Branch: */master   

<a href="/assets/images/4.jpg" data-lightbox="falcon9-large" data-title="매개변수 빌드로 설정 수정">
  <img src="/assets/images/4.jpg" title="매개변수 빌드로 설정 수정">
</a>

소스 코드 관리 부분에서도 Branches to Build를 위에서 설정한 Name 값으로 수정해준다.

<a href="/assets/images/5.jpg" data-lightbox="falcon9-large" data-title="태그 기준으로 빌드되도록 수정">
  <img src="/assets/images/5.jpg" title="태그 기준으로 빌드되도록 수정">
</a>

**3. 빌드 화면**

설정해놓은 Git repository에 Tag가 존재 시 빌드 화면에서 목록에 표기된다 (나의 경우는 태그가 없어서 안나옴)   
필요한 환경에 맞게 태그를 선택해서 빌드하면 된다.

<a href="/assets/images/6.jpg" data-lightbox="falcon9-large" data-title="빌드 선택 화면">
  <img src="/assets/images/6.jpg" title="빌드 선택 화면">
</a>
