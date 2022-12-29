---
layout: post
title:  "Vue siblings component간 데이터 전달하기"
date:   2022-12-29 00:01:07
categories: FRONTEND
tags: vue frontend component
---


아래와 같이 구성된 동일한 계층 컴포넌트가 있을 때 **childOneComponent**의 데이터가 **childTwoComponent**에도 필요한데, 전달할 방법이 마땅치 않았다.

```html
<template>
    <childOneComponent/>
    <childTwoComponent/>
</template>
```

이전에 사용하던 <span class="text-success">**Vue2**</span>버전에서는 아래와 같은 방식으로 쉽게 할 수 있었는데, 안티 패턴인지 <span class="text-secondary">**Vue3**</span>에서는 사라졌다.

**Component 1:**
```javascript
this.$root.$emit('eventing', data);

```

**Component 2:**
```javascript
mounted() {
    this.$root.$on('eventing', data => {
        console.log(data);
    });
}
```

<span class="text-secondary">**Vue3**</span>버전에서는 위 기능들이 제거됨에 따라, 동일한 기능을 하려면 *mitt* 라이브러리를 통해 동일 구현 가능하다고 한다. 

> Vue.js 버전 3에서는 타사 라이브러리를 사용하거나 게시자-구독자(PubSub 개념) 프로그래밍 패턴으로 작성된 기능을 사용할 수 있습니다. ([참고 내용][link])

**event.js**
```javascript
//events - a super-basic Javascript (publish subscribe) pattern

class Event{
    constructor(){
        this.events = {};
    }

    on(eventName, fn) {
        this.events[eventName] = this.events[eventName] || [];
        this.events[eventName].push(fn);
    }

    off(eventName, fn) {
        if (this.events[eventName]) {
            for (var i = 0; i < this.events[eventName].length; i++) {
                if (this.events[eventName][i] === fn) {
                    this.events[eventName].splice(i, 1);
                    break;
                }
            };
        }
    }

    trigger(eventName, data) {
        if (this.events[eventName]) {
            this.events[eventName].forEach(function(fn) {
                fn(data);
            });
        }
    }
}

export default new Event();

```

**index.js**
```javascript
import Vue from 'vue';
import $bus from '.../event.js';

const app = Vue.createApp({})
app.config.globalProperties.$bus = $bus;

```

#### 기타 

[svelte][bonus]도 보너스로 살펴보자

[link]: https://stackoverflow.com/questions/63471824/vue-js-3-event-bus
[bonus]: https://svelte.dev/docs