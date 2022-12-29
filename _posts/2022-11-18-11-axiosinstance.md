---
layout: post
title:  "Axios 동적으로 Global headers 처리하기"
date:   2022-11-18 00:00:01
categories: FRONTEND
tags: axios frontend vuejs
---


<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*axios Instance의 config headers를 동적으로 처리하는 방법*

---

최근에 회사 내부툴을 개발하면서 **axios** 관련된 문제를 겪었다.   
   
프론트엔드에서 **Backend API**와의 통신으로 **accessToken** 형태의 API key 값을 사용하고 있었고 **refreshToken**로 토큰이 자동 갱신되는 형태로 만들었다.   
   
그런데 토큰 갱신 처리 이후에도 **store**의 **accessToken**값은 바뀌었지만 비동기 요청 시 axios에 설정된 **Authorization header**는 변화가 없는 현상이 발생했다.   

<a href="/assets/images/11_1.png" data-lightbox="falcon9-large" data-title="플로우">
  <img style="width:50%; min-width:200px" src="/assets/images/11_1.png" title="플로우">
</a>
<em>그림1. 문제의 과정</em>

그림에서 보듯이 토큰을 B로 갱신 후에도 A로 보내게 되는 현상이었다.

   
JWT기반의 토큰 인증 방식이라 과거의 토큰이 살아있는 한 오류가 발생하지 않기에 그동안 찾을 수 없었던 문제였다. 30분의 삽질 끝에 원인과 해결 방법을 찾아서 메모로 남긴다.

---


#### 구조

프로젝트는 **vue3** 기반으로 개발되어있다. [composition-api][setup]의 **setup**방식이 익숙하지 않아서 개발방식은 **vue2** 버전의 형태로 진행하고 있어 **vue2**라고 봐도 무방하다. 아무튼 axios를 사용하기 위한 공용 설정으로 별도의 js 파일로 분리하여 사용하고 있다.

**axios-instance.js**

```javascript
import axios from 'axios'

const createAxios = (env, store) => {
    let options = {
        baseURL: '',
        timeout: 60000,
        withCredentials: true,
        headers: {
            'Access-Control-Allow-Origin': '*',
            'Authorization': ...access token from store...
        }
    }

    const axiosInstance = axios.create(options)

    axiosIns = axiosInstance
    return axiosInstance
}

let axiosIns
const getAxios = () => axiosIns

export { createAxios, getAxios } 

```

다른 일반 js 파일에서 참조할 수 있도록 **getter**를 제공하고 있으며, **axios**를 생성하는 함수를 **export** 하고 **main.js**에서 store를 넘겨주는 형태로 활용하고 있다.

**main.js**
```javascript
import { createAxios } from '@/common/axios-instance.js'
...
const app = createApp(App)
app.config.globalProperties.$axios = createAxios(process.env.NODE_ENV, store)
```

---

#### 문제점
 
여튼 결론부터 말하면 refreshToken을 통해 갱신처리가 안된 이유는 <span class="text-danger">**createAxios** 하면서 설정했던 **options**이 초기에 한번만</span> 설정되었던게 문제였다. *(어쩐지 새로고침 안하면 토큰이 안먹는 경우가 종종 있었다.)*     
   
당연히 동적으로 생성될거라고 잘못 생각하고 있었다. 문제를 해결하기 위해 처음에는 매번 axios 호출시마다 header를 다음과 같이 설정해 주어야 겠다고 생각했으나, 너무나도 귀찮고 번거로운 일이였다.

```javascript
this.$axios.get(URL, {
    headers: {
        Authorization: ACCESS_TOKEN
    }
})
```
~~(상당히 귀찮다)~~

   

#### 해결방법

해결 방법은 생각보다 간단한데, <span class="text-primary">**axios interceptor**</span>를 활용하는 것이다. 우리의 기존 소스에도 이미 **Error Handling** 및 **config 제어**를 위해 적용되어 있던 항목이다. 
위의 소스에서 axiosInstance에 interceptor를 설정 해보자.

```javascript
import axios from 'axios'

const createAxios = (env, store) => {
    let options = {
        baseURL: '',
        timeout: 60000,
        withCredentials: true,
        headers: {
            'Access-Control-Allow-Origin': '*'
        }
    }

    const axiosInstance = axios.create(options)

    // 여기
    axiosInstance.interceptors.request.use(
        function (config) {

            let authorization = config.headers['Authorization']
            if (authorization == null || authorization == undefined) config.headers['Authorization'] = ...access token from store...

            return config;
        },
        function (error) {
            return Promise.reject(error);
        }
    )

    axiosIns = axiosInstance
    return axiosInstance
}

let axiosIns
const getAxios = () => axiosIns

export { createAxios, getAxios } 
```

생성한 **axiosInstance**에 <span class="text-primary">**axios interceptor**</span>를 생성했다. axios를 통해 ajax 호출 시 호출 전에 **먼저** 실행될 수 있는 영역이다. 매개변수로 받는 config는 axios 호출 파라미터 인자 중 config에 해당된다. (*config를 console에 찍어보면 더 자세히 알 수 있다.*)    
   
여기서는 호출 시 명시적으로 **Authorization**을 설정해서 보내는 경우를 제외하고는 **store**에서 자동으로 설정될 수 있도록 설정했다.

> interceptors.response를 사용하면 응답시의 처리도 가능하다.


#### 꿀팁

axios 호출 시 다음과 같이 지정된 값이 아니더라도 넘겨서 보낼 수 있는데 이것을 활용하면 request / response inteceptor에서 별도의 액션에 활용할 수 있다.   
예를 들면 응답시 별도로 `행동 로그`를 남긴다거나, 호출시에는 `loading spinner`를 자동으로 on하고 응답에 `spinner`를 자동으로 off하는 기능 등으로 활용할 수 있다.

```javascript
this.$axios.get(URL, {
    headers: {
        Authorization: ACCESS_TOKEN
    },
    params: {

    },
    spinner: true // custom config
})
```

전체 Vue 프로젝트 예시 소스는 [여기][source]에서 볼 수 있다.

[source]: https://github.com/seolminsu90/vue3-scaffold/blob/master/vue-scaffold-after/src/common/axios-instance.js
[setup]: https://vuejs.org/api/composition-api-setup.html