---
layout: post
title:  "Frontend Backend 분리된 환경에서 Oauth 연동하기"
date:   2022-11-16 00:00:00
categories: SECURE
tags: frontend backend secure oauth sso
---


<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*백엔드와 프론트엔드가 분리된 환경에서 Oauth 연동하기*

---

과거에 주로 개발하던 구조는 보통 백엔드와 프론트엔드가 합쳐진 구조로 개발되어 왔다.    
프론트 페이지도 JSP/Thymeleaf 등의 서버 템플릿 엔진에 의해 개발되었는데
최근에는 프론트엔드 프레임워크나 라이브러리가가 발전하고 개발규모도 커지면서 프론트엔드 개발과 백엔드 개발을 분리해서 적용하는 경우가 많았다.    
   
예전 구조에서 카카오나 구글과 같은 외부 **Oauth**를 연동할 땐 **redirect** 처리라든지 **token** 처리라든지 처리 주체가 백 프론트 합쳐진 단일 주체라 큰 문제가 없었는데 분리된 구조에서는 각 처리 주체의 선정이 모호하여 이참에 정리하였다.

#### 대강의 프로세스 과정

<a href="https://www.plantuml.com/plantuml/png/TP9V2z9G6CRlzobUz6OR_RbU5w6BXBgA4hmUsn57jSKs8ReoMa7QXP6IsYOJGWeCvXzjKNwXdNVVeQDDt56wEXVFy_oUPo-hk9xcUEtdpuH7x3LryuLcUL3A1ZwTHLatmSy3-YaeWUP2xJwK5QLMKnJUYs62yn3zWAvy4SS9tNMaOw3w1ChDfc4GmWTel2nWrJDMO1KtxnvoTu3EWlHdmjY0yHwoKOAxG60AqvdhKiSeIDVR_-IZ8LrlsEFJtgA8O68QTt1_c2BAShNjjMp7VALvfct1FTiWiYpXk0FPy5lwUmIFYM4wHFVo5lKISMfGJoDucGq2LgBhn7MXRZybbYm3JWQu-i56dOZN5Xe7QVAUex9J__An-nLmsiSl_PmYDaolzIyc5OdNzl0ZxMr5yl8Mj5tYF32adAxoXxqDsUG32nm-OVeMS3MYyyMDCozx5sMlOy0t8VhB8-vXevwpyWlprhoPEo3lmE0FwozE3ZSu-IvKY6Mmd_Gl" data-lightbox="falcon9-large" data-title="플로우">
  <img src="https://www.plantuml.com/plantuml/png/TP9V2z9G6CRlzobUz6OR_RbU5w6BXBgA4hmUsn57jSKs8ReoMa7QXP6IsYOJGWeCvXzjKNwXdNVVeQDDt56wEXVFy_oUPo-hk9xcUEtdpuH7x3LryuLcUL3A1ZwTHLatmSy3-YaeWUP2xJwK5QLMKnJUYs62yn3zWAvy4SS9tNMaOw3w1ChDfc4GmWTel2nWrJDMO1KtxnvoTu3EWlHdmjY0yHwoKOAxG60AqvdhKiSeIDVR_-IZ8LrlsEFJtgA8O68QTt1_c2BAShNjjMp7VALvfct1FTiWiYpXk0FPy5lwUmIFYM4wHFVo5lKISMfGJoDucGq2LgBhn7MXRZybbYm3JWQu-i56dOZN5Xe7QVAUex9J__An-nLmsiSl_PmYDaolzIyc5OdNzl0ZxMr5yl8Mj5tYF32adAxoXxqDsUG32nm-OVeMS3MYyyMDCozx5sMlOy0t8VhB8-vXevwpyWlprhoPEo3lmE0FwozE3ZSu-IvKY6Mmd_Gl" title="플로우">
</a>
<em>그림1. 각 역할 별 플로우 정리</em>

[수정링크(PlantUML)][umllink]

위에서부터 차례대로 정리해보면   

**1. 로그인 창 호출**   

외부 인증서버(카카오, 구글 등)가 제공하는 서비스의 로그인 창을 띄우기를 요청한다.

**2. 로그인 창 제공**

각 서비스 제공사의 로그인 화면이 뜨는 단계다.

**3. 로그인 정보 전달**

해당 서비스로 연동된 로그인 정보를 인증 서버로 제공하는 과정이다.

**4. 인가코드(Authorization Code)와 함께 Redirect**

3의 과정을 거치면 인증 서버는 애플리케이션 등록시 설정한 redirect 정보로 authorization code를 파라미터로 전달해준다.   
각각의 서비스 별로 redirect 설정하는 설정 페이지가 있으니 찾아서 등록을 해줘야 한다.

**5. 인가코드 전달**

인증 서버로부터 받은 code를 백엔드로 전달해주는 과정이다. **POST** 방식으로 노출을 최소화 한다.

**6. 인가코드로 Access Token 요청**

프론트엔드로부터 받은 코드를 통해 인증서버에 AccessToken 발급을 요청한다.(***보통의 oauth 표준은 /oauth/token***)   
백엔드 서비스 내에서 HttpClient로 내부 호출하여 응답을 받는다.

**7. Access Token 발급**

인증서버는 유효한 code를 통해 자원을 사용할 범위 또는 권한을 갖는 AccessToken을 발급해준다.

**8. 서비스 자원 요청(with Token)**

서비스에서 사용할 유저 인증에 필요한 인증서버 고유의 유저 자원을 요청하는 과정이다. (ex. Kakao Id, Google Id, Email 등)

**9 ~ 10. 서비스 자원 응답(ex. userId) 및 로그인 / 회원가입 처리, 서비스토큰 생성**   

백엔드는 인증서버로 부터 자원 정보를 응답받아 로그인/회원가입을 처리하고 고유 서비스에서 사용할 별도의 인증 정보를 생성한다.   
Token 자체로 유저를 식별할 수 있는 가벼운 키라 JWT를 보통 자주 사용한다.   
인증 서버는 로그인 할 유저를 식별해주는 역할만 수행하기 때문에 이후의 처리는 일반적인 로그인/회원가입 처리와 같다고 보면 된다.

**11. 서비스토큰 응답**

생성한 서비스 전용 토큰을 프론트엔드로 응답해준다. 이때 access + refresh 두가지 토큰 기반의 룰을 적용한다면 좀 더 토큰 탈취 등의 위험에서 조금은 회피할 수 있다.   
짧은 만료시간을 갖는 access token와 비교적 긴 만료시간을 갖는 refresh toekn을 이용하여 노출되는 access token의 사용성을 최소화 하는 전략이다.   
위에서 사용한 외부 인증 서버(Oauth) 의 경우도 AccessToken 발급 시 별도의 RefreshToken도 발급되는 것을 확인 할 수 있다.

**12. 서비스토큰으로 서비스 이용**

응답받은 서비스 token을 적절한 위치(Vue store)에 저장하고 백엔드 호출 시 사용한다.   
긴 만료시간을 갖는 refresh token과 같은 경우엔 저장소에 대한 의견이 분분한데 나는 보통 localstorage나 cookie를 선호한다.(어짜피 털리면 다 털린다고 생각)

> 플로우만 적당히 인지하고 있으면 내부의 로직을 구현하는건 크게 어렵지 않을 것이다.

---

#### Refresh token은 언제 갱신할까?

서비스 정책마다 다르지만 자동으로 갱신해주느냐 수동으로 갱신해주느냐로 구분할 수 있다.

**자동으로 갱신할 때**

백엔드 호출 시 만료된 토큰일 경우 별도의 익셉션 응답을 프론트엔드에 전달해주고, 이후 refresh token을 통해 access token을 갱신하는 방법이다.   
만약 서비스에서 ajax library로 axios를 사용한다면 request interceptor 설정을 통해 요청 시 만료여부를 체크하고 갱신처리할 수 있다.   
혹은 access token의 만료일을 주기적으로 체크하여 만료에 근접하였을 때 백그라운드에서 자동으로 갱신해주는 방법이 있다. 전자의 경우 retry가 필요하기 때문에
개인적으로는 후자를 선호하는 편이다. 

**수동으로 갱신할 때**

이력서 사이트나 은행, 관공서 등을 보면 로그인 세션 시간을 서비스 화면에 표기하는 서비스들이 있는데, 이런 것처럼 해당 세션의 만료 시간을 서비스에 노출해주고 
사용자가 수동으로 갱신할 수 있는 버튼을 제공하는 방식이다.

> 서비스 정책에 따라 수동 혹은 자동으로 갱신될 수 있도록 개발하면 된다.

[umllink]: https://www.plantuml.com/plantuml/uml/TP9V2z9W6CNlzoaUz6OR_RbU5w6BXBgA4hmUsokEQejjGdHbj8Aq2oCbjKqcX1GOp3_Qeln2UvzzXrvjubwbgmtEERzpzh2AwscErtx-J10UiHTJDrzebWUbRE3JMPHr4_mye9-24cWkrEv3ML5gDKNXlHWcE0_H3-ZA5t6Sq5r7EWQgJw3SR1e68No0nii2LZrZ1LPny-uXTmVeB4X_9emD6EyX6oMu2mIcC9svBdMCWdIz_Ky-6jBrXZrysbiK4KnCumxk3rD4MQwslPRjcAyqpxGjk6SRH9Ob77SW6tvBVu_WCJ5CX-XUtg9U8auDwca4BxD1m4fqdRWkrEsdn99bmCa0Lp_Og1FnsaBZ84tUSrHswby-Thy2JllufPzJn8RfjVx5f2BnsXv-fBsjI5xUWhPBd0U6bDDL_h2t8PjyO8739qpV0gv6TBwuSUPbxwBiTGpu9aH_USGzRDHpLdvXtfetSmVa7HZyOVt5oU469zy5eK8iW_scVm40