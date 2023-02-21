---
layout: post
title:  "오라클 클라우드 서비스 맛보기"
date:   2023-02-21 00:00:01
categories: SERVER
tags: oracle cloud
cover: server.jpg
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*오라클 클라우드 가입 및 인스턴스 생성하여 가지고 놀기*

---

지금까지 **AWS, Naver, Tencent** 등 다양한 클라우드 서비스를 경험해봤었는데 정작 업무에만 사용하고 개인적으로 사용한 적은 없었던 것 같다. 
그나마 AWS가 **1년 무료**여서 가입했었는데, 초과 트래픽이나 해킹 등으로 요금폭탄을 맞을 게 두려워서 가입하고 얼마 안되서 탈퇴했었다. 무료하던 중 지인의 추천으로
<span class="text-warning">**오라클**</span>에도 클라우드 서비스가 있는 것을 알게 되었고, **무려 평생 무료**에 자동 과금 없는 시스템이어서 한번 맛보기로 사용해보았다.   

**상시 무료 서비스 및 한달 크레딧**을 제공해주는데, 한달 크레딧은 사실 관심 없고 상시 제공 항목만 봐도 가지고 노는데는 충분하다 생각했다.

---

**상시 제공 항목(프리 티어)**

|---|
|Oracle APEX 및 Oracle SQL Developer와 같은 강력한 도구가 포함된 Oracle Autonomous Database 2개|
|AMD 컴퓨팅 VM 2개|
|ARM 기반 Ampere A1 컴퓨팅 인스턴스 최대 4개 지원(매월 3,000 OCPU 시간과 18,000 기가바이트-시간 제공)|
|블록 스토리지, 오브젝트 스토리지, 아카이브 스토리지, 로드 밸런서 및 데이터 송신, Monitoring 및 Notifications|

여러 기능들이 제공되지만 개인적으로는 그냥 Remote DB나 API service 정도로 사용할 것 같긴 하다.   
프리티어 제공 설명 [사이트][free]를 보면 더 자세한 정보를 볼 수 있다.

---

#### 오라클 클라우드 가입

우선 해당 [링크][free]를 통해 무료로 시작하기를 누른다.

<a href="/assets/images/29_0.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_0.png" style="" title="">
</a>

국가 및 사용자 정보, 이메일을 입력한다.

<a href="/assets/images/29_7.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_7.png" style="" title="">
</a>

이메일 인증이 필요한데, 다음과 같이 verify 메일의 링크를 이용하면 인증 절차가 처리된다.

<a href="/assets/images/29_01.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_01.png" style="" title="">
</a>

그 다음은 회사 정보인데 생략해도 되기에 아무거나 입력해도 된다. 클라우드 계정 이름의 경우 로그인 할 때 입력하기 때문에 편한 아이디로 입력하면 된다.   
홈 영역의 경우 프리티어의 경우 <span class="text-danger">**변경이 불가능**</span>하므로 서울/춘천으로 제공되는 한국 리전 중에서 선택한다.

<a href="/assets/images/29_8.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_8.png" style="" title="">
</a>

주소는 한글 주소 대충 입력해도 무방한 것 같았다. 본인 주소를 입력하고 **해외 결제가 가능한 신용/직불 카드**를 이용하여 본인확인한다.   
이 과정이 처음에는 AWS 처럼 추후 자동결제 되거나 하는 걱정이 또 됬었는데, 본인인증 이후 정보는 사용되지 않으니 안심해도 될 것 같다. (추후 유료계정 전환 시 새로운 지불 정보를 입력하게 한다.)

<a href="/assets/images/29_9.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_9.png" style="" title="">
</a>

지불 정보까지 입력하고 나면 결제/환불이 되면서 카드를 통한 본인인증이 완료되고 가입 절차가 완료되어 계정정보가 담긴 이메일을 받게 된다.

---

#### VM 인스턴스 생성하기

계정 생성도 완료되었고 가장 기본적인 VM 인스턴스를 생성하여 원격 환경에 나만의 서버를 생성해보도록 하자.    
[로그인][login]을 진행한 후, 페이지 좌측 상단의 햄버거 메뉴 중 **컴퓨트 > 인스턴스**를 선택한다.

<a href="/assets/images/30_1.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/30_1.png" style="" title="">
</a>

인스턴스 생성 버튼을 누르면 아래와 같은 화면이 뜨게 된다. 이름에는 목록상 표기 될 해당 인스턴스를 식별 할만 할 이름을 적어준다.

<a href="/assets/images/29_2.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_2.png" style="" title="">
</a>

다음은 ssh 접속을 위해 인스턴스에 **퍼블릭 키 정보**를 등록해준다. 개인적으로 사용하고 있는 키가 없으면 **자동으로 키 쌍 생성**을 통해 생성한 후 퍼블릭 키를 업로드한다.
이외의 나머지 값은 우선 기본값으로 설정하고 생성 버튼을 누른다. 
> DB 등 내부통신 전용으로 생성하려면 네트워크 설정 중 public ip 항목을 수정한다.

<a href="/assets/images/29_3.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_3.png" style="" title="">
</a>

다음과 같이 **인스턴스 목록**이 추가된 것을 확인할 수 있다. 초기에는 ssh **22 포트**만 접근이 가능한데, 기타 다른 서비스(http,  https, mysql) 등을 이용하기 위해 Inbound/Outbound 포트 설정이 필요하다. (AWS의 Security Group 같은 개념)   
생성된 **인스턴스의 상세정보**를 누르거나 **네트워킹 > 가상 클라우드 네트워크** 을 접근하여 Default로 생성된 항목을 수정한다.

> 보안그룹? 서브넷? 설정 차이(AWS 기준) [링크][link]


<a href="/assets/images/29_4.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_4.png" style="" title="">
</a>

인스턴스에 기본으로 설정되어 있는 **서브넷의 보안 목록**을 선택한다.

<a href="/assets/images/29_5.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_5.png" style="" title="">
</a>

**수신 규칙 추가** 를 클릭하여 인스턴스에서 수신할 포트 정보를 기입하여 추가한다.

<a href="/assets/images/29_6.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_6.png" style="" title="">
</a>

나는 mysql 및 api 서비스 정도를 관리할 예정이라 다음과 같이 몇개만 추가해 놓았다.

<a href="/assets/images/29_10.png" data-lightbox="falcon9-large" data-title="">
  <img src="/assets/images/29_10.png" style="" title="">
</a>

---

#### 인스턴스 접속

외부에서 **telnet**을 해보면 22port를 제외하고는 반응이 없는 것을 확인할 수 있었다. AWS를 사용할 때에는 위의 설정정도만 해도 접근이 되었던 것으로 기억하는데 오라클 리눅스의 경우 **별도의 인스턴스 방화벽**도 처리해줘야 하는 것 같다.   
우선 인스턴스 생성할 때 사용한 키를 이용하여 인스턴스에 접속한다.

```bash
ssh -i {인스턴스 생성할 때 사용한 키} opc@{instance public ip}
```
> 오라클 리눅스 기본 계정은 opc로 되어있는데 root유저를 사용하려면 `sudo passwd root` 를 통해 비밀번호를 설정해준다.

현 시점 기준 Oracle Linux 8 버전으로 설치가 되어있는데, 방화벽이 firewalld service로 관리되고 있으므로 전부 종료해주었다.

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```
> 모든 port가 개방되므로 모든 port 개방을 원치 않으면 세부적으로 방화벽을 설정해서 사용한다. ([참조][firewalld])

이제 다시 telnet을 시도해보면 **Resource temporarily unavailable**로 나오던 메시지가 **Connection refuse**로 정상접근 되는 것을 알 수 있다. (포트에 할당된 서비스가 없어서 refuse로 발생된다)

---

#### Mysql 설치하기

무엇을 해볼까 하다가 간단하게 DB 정도만 설치해보기로 했다.

mysql 설치는 여기저기 많으니 간단하게 과정만 기록한다.

**설치**

```bash
sudo dnf module list mysql

sudo dnf install @mysql:8.0

sudo systemctl enable --now mysqld
```

**Root 비밀번호 설정**

```bash
sudo mysql_secure_installation
```
대화형 프롬프트를 통해 기호에 맞게 설정해준다.

**접속**
```bash
mysql -u root -p
{위에서 설정한 비밀번호}
```

**원격 접속용 계정 설정**
```
CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '{pwd}';
```

이제 로컬 컴퓨터 등 외부 환경에서도 인스턴스에 설치된 DB로 접근해보면 정상적으로 접근되는 것을 확인할 수 있다.

***여기까지 완료하면 오라클 클라우드의 아주 쉽게 나만의 Mysql DB가 생성된다.***

---

#### 참조

[외부 접속이 가능한 MariaDB 서버 구축기][ref1]   
[How To Install MySQL 8.0 on Oracle Linux 8][ref2]   

[free]: https://www.oracle.com/kr/cloud/free/
[login]: https://www.oracle.com/kr/cloud/sign-in.html
[firewalld]: https://team-okitoki.github.io/getting-started/compute-firewall/
[ref1]: https://devwithpug.github.io/database/oci-mariadb/
[ref2]: https://techviewleo.com/how-to-install-mysql-on-oracle-linux-server/
[link]: https://medium.com/awesome-cloud/aws-difference-between-security-groups-and-network-acls-adc632ea29ae

