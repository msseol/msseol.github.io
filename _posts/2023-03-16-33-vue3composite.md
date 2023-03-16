---
layout: post
title:  "Vue3 composite api 예시 메모"
date:   2023-02-28 00:00:02
categories: FRONTEND
tags: vue3 vue composite optional
cover: vue.png
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Vue3 CompositeAPI 사용 방식 메모*

---


#### OptionalAPI와 큰 차이 몇개 정리

**1. refs 주입**
```javascript
import { ref } from 'vue';
const myRef = ref(null); // ref="myRef" 으로 생성한 템플릿 참조 마운트 단계에서 주입된다.

// myRef.value 로 접근하여 사용한다. html코드영역에서는 myRef로 바로 접근이 가능함.
```

**2. props 받기**
```javascript
import { defineProps } from 'vue'
const props = defineProps(['foo'])
const props =  defineProps({
      disabled: Boolean,
   })
const props = defineProps({
  foo: { type: String, required: true },
  bar: Number
})
```

**3. 라우트 접근**
```javascript
import { useRoute } from 'vue-router'
const route = useRoute();
```

**4. Store(Vuex) 사용**
```javascript
import { computed } from 'vue'
import { useStore } from 'vuex'
const store = useStore()

const storeAction = {
    count: computed(() => store.state.count),
    double: computed(() => store.getters.double),
    increment: () => store.commit('increment'),
    asyncIncrement: () => store.dispatch('asyncIncrement')
}
```

**5. LifeCycle (mounted().. created()..)**
```javascript
onMounted(()=>{ 
    // doSometing... 
}) 
```

**6. Watch**
```javascript
watch(() => route.name, (name) => state.routeName = name)
```

**7. 자식 컴포넌트 접근**     
부모 -> 자식 호출 ([참조][ref2])
```javascript
/* Parent.vue */
...
<Children ref="child"/>
...
<script setup>
...
const child = ref(null)
const childCall = () => {
      child.childFunction()
}
</script>
```
```javascript
/* Children.vue */
<script setup>
...
const childFunction = () => {
      console.log("Hello World")
}

defineExpose({
  childFunction
})
</script>
```
**8. 부모 컴포넌트 Emit**   
자식 -> 부모 호출 ([참조][ref3])   

- Event Emit 방식

```javascript
// App.vue (Parent component)
...
<Todos :todos="todos" @complete-todo="updateTodos" />
...
<script setup>
const todos = ref([
...
]);

const updateTodos = (updatedTodo) => { /* do.. */};
</script>
```
```javascript
// Todos.vue (Child component)
...
<button @click="emit('completeTodo', todo)">Done</button>
...

<script setup>
const props = defineProps(['todos']);
const emit = defineEmits(['completeTodo']);
</script>
```

- Function 전이 방식

```javascript
// App.vue (Parent component)
...
<Todos :todos="todos" :complete-todo="updateTodos" />
...

<script setup>
const todos = ref([
...
]);

const updateTodos = (updatedTodo) => { /* do.. */};
</script>
```
```javascript
// Todos.vue (Child component)
...
<button @click="completeTodo(todo)">Done</button>
...

<script setup>
const props = defineProps(['todos', 'completeTodo']);
</script>
```

[ref2]: https://velog.io/@violet_yang/TIL-Vue.js-defineEmits-defineExpose
[ref3]: https://www.webmound.com/pass-data-child-to-parent-vue/