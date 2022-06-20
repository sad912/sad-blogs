# Pinia 入门｜看文档写 Demo

还是贯彻以往的学习思路，先看文档写demo，再深入原理。

## Pinia vs Vuex

Pinia 原本是使用 Composition API 重新设计的 Vuex。也就是说，从文档来看，Pinia 已经替代了 Vuex。同时，官方文档是这样描述二者的区别的：

> Compared to Vuex, Pinia provides a simpler API with less ceremony, offers Composition-API-style APIs, and most importantly, has solid type inference support when used with TypeScript.

总结如下：

1. 更为精简的 API；
2. 组合式 API；
3. TypeScript 支持；

### 安装

```other
pnpm add pinia
```

本文直接使用 Vue3 和 Composition API `<script setup>` 的开发方式进行学习。

```other
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

app.use(createPinia()).mount('#app')
```

## 一言蔽之，什么是 Pinia?

> it hosts global state.

> It has **three concepts**, the state, getters and actions and it's safe to assume these concepts are the equivalent of `data`, `computed` and `methods` in components.

使用 `data`、`computed` 和 `methods` 三个概念类比 Pinia 的 `state`、`getters` 和 `action`，可以说是很形象了。

## 声明状态管理

使用 `defineStore()` 函数定义一个状态管理实例。第一个参数为实例名称字符串，第二个参数为实例选项组成的对象。

```other
// src/store/sad.ts

import { defineStore } from 'pinia'

export const sad = defineStore('SAD', {
  // other options...
})
```

我们可以在组件内引入所定义实例。

```other
// src/components/test.vue

<script setup>
import { sad } from 'pinia'
const sadStore = sad()
</script>
```

## State

State 即状态，作为 `defineStore()` 函数的第二个选项对象参数的一个选项，值为返回状态对象的函数。

声明一个 `mode` 状态，代码如下：

```other
// src/store/sad.ts

import { defineStore } from 'pinia'

export const sadState = defineStore('SAD', {
  state: () => {
    mode: 'learning',
    modeList: [],
  }
})
```

然后，我们在组件中，可以通过解构状态管理实例来访问这个 `mode` 状态。

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const { mode } in sadState()
</script>
<template>
  {{ mode }}
</template>
```

### 响应式

通过直接修改状态管理实例中的 `mode` 是可以生效的，但是这并不能使其保持响应式。如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const { mode } = sadState()
  setTimeout(() => {
    sadState().mode = 'sleeping'
  }, 500)
</script>
<template>
	<div>这里只会显示 learning:</div>
	<div>{{ mode }}</div>
</template>
```

当然，我们可以利用 `computed()` 函数实现其响应式。如下：

```other
// src/compnents/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  import { computed } from 'vue'
  const { mode } = sadState()
  setTimeout(() => {
    sadState().mode = 'sleeping'
  }, 500)
  const computedMode = computed(() => sadState().mode)
</script>
<template>
  <div>这里只会显示 learning：</div>
  <div>{{ mode }}</div>
  <div>但这里会在 500ms 后显示 sleeping：</div>
  <div>{{ computedMode }}</div>
</template>
```

但是，Pinia 提供了更方便的方法 `storeToRefs()`，使用这个方法可以使解构后的状态实例变成响应式的。如下：

```other
// src/compnents/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  import { storeToRefs } from 'pinia'
  const { mode } = storeToRefs(sadState())
  setTimeout(() => {
    sadState().mode = 'sleeping'
  }, 500)
</script>
<template>
  <div>这里先显示 learning，500ms 后显示 sleeping:</div>
  <div>{{ mode }}</div>
</template>
```

另外，状态实例化的返回值，是可以保持响应式的。如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  setTimeout(() => {
    store.mode = 'sleeping'
  }, 500)
</script>
<template>
  <div>先显示 learning，500ms 后显示 sleeping</div>
  <div>{{ store.mode }}</div>
</template>
```

### 重置

状态管理实例提供了重置状态函数 `$reset()` 。如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.mode = 'working'
  setTimeout(() => {
    store.$reset()
  }, 500)
</script>
<template>
  <div>先显示 working，500ms 后状态重置，显示 learning</div>
  <div>{{ store.mode }}</div>
</template>
```

### 修改

上述例子中，我们多次使用直接修改的方式来改变 `mode` 的状态，修改状态的方式一共有三种：

1. 直接修改状态值；
2. 通过 `$patch()` 分发要修改的状态组成的对象值，实现修改；
3. 通过 `$patch()` 分发用于修改值的函数，实现修改；

给 `$dispatch()` 函数传递一个多个状态属性和状态值组成的对象，可以实现多个状态值的同时修改。下面是一个传递对象的例子：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$patch({
    mode: 'working',
  })
</script>
<template>
  <div>此时显示 working：</div>
  <div>{{ store.mode }}</div>
</template>
```

但对于状态值为数组类型的状态的修改，出于代价和简易程度的考虑，往往采用给 `$dispatch()` 函数传递一个修改状态的回调函数，回调函数的参数为状态实例。下面是一个传递回调函数的例子：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$patch((store) => {
    store.modeList.push('learning')
    store.modeList.push('working')
    store.modeList.push('sleeping')
  })
</script>
<template>
  <div v-for="mode in store.modeList" :key="mode">{{ mode }}</div>
</template>
```

### 替换

状态管理实例还提供了 `$store` 属性，可以用于替换修改实例的状态。如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  setTimeout(() => {
    store.$state = { mode: 'sleeping' }
  }, 500)
</script>
<template>
  <div>先显示 learning，500ms 后显示 sleeping：</div>
  <div>{{ store.mode }}</div>
</template>
```

### 发布订阅

状态管理实例还提供了 `$subscribe()` 方法，可以用于监听状态的修改。

```other
store.$subscribe((mutation, state) => {
  mutation.type // 'direct' | 'patch object' | 'patch function'
  mutation.storeId // 'SAD'
  // only available with mutation.type === 'patch object'
  mutation.payload // patch object passed to store.$patch()
  state.mode //'learning'
})
```

当直接对状态进行修改时，`mutation.type` 为 `direct`。代码如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe((mutation, state) => {
    console.log(mutation.type) // 'direct'
    console.log(state.mode) // 'working'
  })
  store.mode = 'working'
</script>
<template></template>
```

```other
// src/components/test.vue
<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe((mutation, state) => {
    console.log(mutation.type)  // 'direct'
    console.log(state.modeList[0]) // 'sleeping'
  })
  store.modeList.push('sleeping')
</script>
<template></template>
```

当通过 `$patch()` 方法传递状态对象时，`mutation.type` 应为 `patch object`。测试代码如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe((mutation, state) => {
    console.log(mutation.type)
    console.log(state.mode)
  })
  store.$patch({
    mode: 'sleeping',
  })
</script>
<template></template>
```

我期望发布订阅的回调函数执行一次，但是浏览器控制台显示回调函数被执行了两次，一次修改类型为 `patch object`，一次为 `direct`。光看文档找不出其中缘由，看完文档再读源码吧。

```other
// console.log
patch object
sleeping
direct
sleeping
```

接着，尝试给 `$patch()` 方法传递函数，`mutation.type` 应为 `patch function`。测试代码如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe((mutation, state) => {
    console.log(mutation.type)
    console.log(state.mode)
  })
  store.$patch((state) => {
    state.mode = 'reading'
  })
</script>
<template></template>
```

同样的问题，控浏览器控制台显示发布订阅的回调函数被执行两次，日志如下：

```other
// console.log
patch funtion
reading
direct
reading
```

另外，当同时使用直接修改和 `$patch()` 方式修改的时候，也测试出了一些问题。测试代码如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe((mutation, state) => {
    console.log(mutation.type)
    console.log(state.mode)
  })
  store.modeList.push('sleeping')
  store.$patch({
    mode: 'sleeping',
  })
  store.mode = 'working'
  store.$patch((state) => {
    state.mode = 'reading'
  })
</script>
<template></template>
```

我期望回调函数调用四次，但浏览器控制台显示前两次修改调用了一次，后两次修改调用了一次，还莫名其妙调用了一次且 `mutation.type` 为 `direct`。控制台日志如下：

```other
// console.log

patch object
sleeping
patch function
reading
direct
reading
```

另外，我继续分别测试了使用 `$reset()` 和`$state` 修改 state 的情况，两种情况和使用 `$patch()` 传递函数修改 state 时一样。

面对测试的疑问，我开始尝试寻找答案，在官方 issues 中，有这么一个问题。

[`$subscribe` miss mutations with type of `direct` immediately after`patch` mutations · Issue #992 · vuejs/pinia](https://github.com/vuejs/pinia/issues/992)

我尝试在发布订阅函数的第二个选项对象参数中，加入 `flush: sync`。代码如下：

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
  store.$subscribe(
    (mutation, state) => {
      console.log(mutation.type)
      console.log(state.mode)
    },
    { flush: 'sync' }
  )
<template></template>
```

此时上述的期望打印都可以满足了。

而至于为什么会这样。还需要深入研究。本文目的是入门 Pinia，记录看文档时学习和思考的过程，先不做深入研究。

## Getters

Getters 的定义和使用类似于计算属性。

`getters` 作为 `defineStore()` 函数第二个参数的选项属性，用于声明 `Getters` ，值为带有 `state` 参数的函数。

下面是一个简单的例子。

```other
// src/store/sad.js

import { defineStore } from 'pinia'
export const sadState = defineStore('SAD', {
  state: () => {
    return {
      mode: 'learning',
      modeList: [],
    }
  },
  getters: {
    myState: (state) => `I am ${state.mode}.`,
  },
})
```

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
</script>
<template>{{ store.myState }}</template>
```

浏览器显示 `I am learning.`

同时，可以使用 `this` 在一个 `getter` 里调用另一个 `getter`。

```other
// src/store/sad.js

import { defineStore } from 'pinia'
export const sadState = defineStore('SAD', {
  state: () => {
    return {
      mode: 'learning',
      modeList: [],
    }
  },
  actions: {
    setModeToWorking() {
      this.mode = 'working'
    },
  },
  getters: {
    myState: (state) => `I am ${state.mode}.`,
    noTime() {
      return `I have no time because ${this.myState}`
    },
  },
})
```

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
</script>
<template>{{ store.noTime }}</template>
```

浏览器显示 `I have no time because I am learning.`

由于 `getters` 默认传递参数 `state`，如果想为 `getters` 传递其他参数，可以选择给 `getters` 返回带参数的函数。下面是一个简单的例子：

```other
// src/store/sad.js

import { defineStore } from 'pinia'
export const sadState = defineStore('SAD', {
  state: () => {
    return {
      mode: 'learning',
      modeList: ['learning', 'working', 'sleeping', 'reading', 'writing'],
    }
  },
  getters: {
    getModeByIndex: (state) => {
      return (index) => {
        return state.modeList[index]
      }
    },
  },
})
```

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
</script>
<template>modeList 的第一个 mode 值为：{{ store.getModeByIndex(0) }}</template>
```

此时浏览器显示 `modeList 的第一个 mode 值为：learning`。

要注意，此时的 `getters` 是不被缓存的，但是你可以在 `getters` 内部缓存一些数据，用于返回函数中进行处理，来提高性能。

另外，在 `getters` 中还可以使用其他 `store` 的 `getters`。例子如下：

```other
// src/store/test.js

import { defineStore } from 'pinia'

export const testState = defineStore('TEST', {
  state: () => {
    return {
      test: '测试',
    }
  },
  getters: {
    valueOfTest(state) {
      return state.test
    },
  },
})
```

```other
// src/store/sad.js

import { defineStore } from 'pinia'
import { testState } from '@/store/test.ts'
export const sadState = defineStore('SAD', {
  state: () => {
    return {
      mode: 'learning',
    }
  },
  getters: {
    contactStoreState(state) {
      return state.mode + '|' + testState().valueOfTest
    },
  },
})
```

```other
// src/components/test.vue

<script setup>
  import { sadState } from '@/store/sad'
  const store = sadState()
</script>
<template>{{ store.contactStoreState }}</template>
```

浏览器显示 `learning|测试`。

## Actions

`Actions` 相当于组件内的 `methods`。

`actions` 作为 `defineStore()` 函数第二个参数的选项属性，用于声明 `Actions` ，值为用于逻辑操作的函数。

通过 `this` 可以访问 `state` 和 `getters`。

同时，可以在 `actions` 中使用异步操作。

另外，还可以访问其他 `store` 中的 `actions`。

由于 `actions` 和 `methods` 几乎一致，不再用代码展示。

## Plugins

Plugins 的功能，在之后进阶 Pinia 博客中再详细介绍。

