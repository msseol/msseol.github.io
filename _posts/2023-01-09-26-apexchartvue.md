---
layout: post
title:  "Vue3에서 Apexchart 사용하기"
date:   2023-01-09 00:00:00
categories: FRONTEND
tags: vue vue3 chart apexchart
cover: vue.png
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Vue3에서 ApexChart 사용 기초 메모*

---

**Vue 기반 백오피스 페이지**를 개발하면서 <span class="text-info">**차트**</span>를 사용해야 할 업무가 생기게 되었다. 아주 예전에 *게임 관리자 통계* 등을 표현할 때 기본적인 Bar 차트는 직접 구현해서 사용하기도 했고, 파이 차트나 조금 복잡한 차트는 **Chart.js** 라이브러리를 사용해서 개발했었던 경험이 있다.   
   
그 당시에는 **Frontend Framework**도 발전되지 않은 시기라서 대충 **Jquery** 기반으로 차트를 개발해도 무난했는데 최근엔 Jquery를 잘 쓰지 않기도 하고, 프론트엔드와 호환되는 차트가 필요하여 찾아보게 되었고 여러가지 차트 라이브러리들이 개발되어 있었다. 그 중에서도 동료의 추천으로 사용하기도 편하고 <span class="text-success">**Vue와도 쉽게 호환**</span>되는 <span class="text-danger">**Apexchart**</span>를 사용하기로 하였다.

---

#### Vue 프로젝트 생성

먼저 Vue 기반의 환경에서 만들 것이므로 **Vue 프로젝트를 생성**한다. 아래의 명령어를 입력하여 **대화형**으로 프로젝트를 생성한다.

```bash
npm init vue@latest #node 설치과정은 생략한다.
```

<a href="/assets/images/26_1.png" data-lightbox="falcon9-large" data-title="Vue 프로젝트 옵션들">
  <img src="/assets/images/26_1.png" style="max-width:65%" title="Vue 프로젝트 옵션들">
</a>
<em>그림1. Vue 프로젝트 옵션들</em>

> vue 설치는 할때마다 옵션이 금방금방 다양해지는 것 같다. **Pinia**라는건 이번 설치하면서 처음 봤는데, **Vuex**와 비슷한 상태 저장 기능을 한다고 한다. (추후 별도로 정리)

#### Apex Chart 설치

Vue3버전을 기준으로 설치할 것이기 때문에 **Vue3용 chart**를 설치해야한다.

```bash
npm install --save apexcharts
npm install --save vue3-apexcharts # vue 2 버전은 vue-apexcharts를 사용한다.
```

위에서 생성했던 vue 프로젝트의 main.ts(혹은 main.js)에 ApexChart를 사용할 수 있도록 설정한다.

```javascript
import VueApexCharts from "vue3-apexcharts";

const app = createApp(App);
app.use(VueApexCharts);
...

```

이것으로 기본적으로 차트를 사용하기 위한 준비는 끝이 났다.

#### apexchart component

Vue기반의 차트로 개발되어서, 차트 역시 Vue component를 통해 관리되는데 관련되서 전달되는 Props 목록은 다음과 같다.

|Prop|Description|Type|Default|
|---|----|----|----|
|**type**|차트 유형(필수 Props)|String|‘line’|
|**series**|차트에 표시하려는 데이터|Array|undefined|
|**width**|차트의 너비|String or Number|‘100%’|
|**height**|차트의 높이|String or Number|‘auto’|
|**options**|기타 차트 설정 객체|Object|{}|

주로 기타 설정들을 정의해서 `:options`에 담아서 사용하는 형태로 이루어진다. option에는 색상 테마나 marker, legend등 차트에 표기되는 다양한 값을 설정할 수 있도록 되어있다. ([옵션 레퍼런스][options])

#### 사용법


```html
<!-- MyChart.vue -->
<template>
  <div>
    <apexchart width="500" type="bar" :options="options" :series="series"></apexchart> <!-- bar chart -->
  </div>
  <div>
    <apexchart width="500" type="line" :options="options" :series="series"></apexchart> <!-- line chart -->
  </div>
  <div>
    <apexchart width="380" type="donut" :options="donut_options" :series="donut_series"></apexchart> <!-- donut chart -->
  </div>
</template>
```

컴포넌트에 사용될 샘플 데이터들과 데이터 갱신을 위한 샘플 메소드 예시다.

```typescript
import { defineComponent } from 'vue'
export default defineComponent({
  data() {
    return {
      options: {
        chart: {
          id: 'vuechart-example'
        },
        theme: {
          palette: 'palette5'
        },
        xaxis: {
          categories: [1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998]
        }
      },
      series: [{
        name: 'series-1',
        data: [30, 40, 45, 50, 49, 60, 70, 91]
      }],
      donut_options: {},
      donut_series: [44, 55, 41, 17, 15]
    }
  },
  methods: {
    updateChart() {
      const max: number = 90;
      const min: number = 20;
      const newData: number[] = this.series[0].data.map(() => {
        return Math.floor(Math.random() * (max - min + 1)) + min
      })

      // Data를 변경하면 차트도 반응형으로 바로 반영되어 변경된다.
      this.series = [{
        name: 'series-2',
        data: newData
      }]
    }
  }
})
```

#### 결과 예시

<a href="/assets/images/26_2.png" data-lightbox="falcon9-large" data-title="세가지 차트 예시">
  <img src="/assets/images/26_2.png" style="max-width:65%" title="세가지 차트 예시">
</a>
<em>그림2. 위 샘플의 실제 화면</em>

아주 손쉽게 축소/확대등 반응형 차트 보기 옵션과 엑셀 다운로드 등등 여러가지 통계페이지나 관리에서 꼭 필요하던 추가 기능도 기본적으로 제곧되는 편리한 차트를 만들수 있었다.   
실제 라이브 동작을 확인해보려면 누군가가 만들어 놓은 [샌드박스][sendbox]에서 구동해보도록 하자.   

> Apex 차트에서 지원하는 차트 종류는 위의 3개를 포함하여 16가지나 되므로 자세한 건 [공식 홈페이지][apex]에서 도큐멘트를 확인해보자


[apex]: https://apexcharts.com/docs/installation/
[sendbox]: https://codesandbox.io/embed/qzjkzmzxoj
[options]: https://apexcharts.com/docs/options/annotations/