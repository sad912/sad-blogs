# Vue 3 组件通信全解

Vue 的一大特点就是组件化，而组件通信又是组件化最为重要的开发知识，本文对组件通信的方式和使用场景的分类做一个总结。

> 需要注意的是，本文探讨的对象是使用 Composition API 和 `<script setup>` 的组件。

## 组件通信的方式

### Props 声明

Props 声明的方式是 Vue 中最简单的通信方式。

下面是一个简单的例子：

```other
// @components/propsExample/PartentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {ref} from "vue";

const myMode = ref('learning')
</script>

<template>
  <ChildComponent :my-mode="myMode"/>
</template>
```

```other
// @/components/propsExample/ChildComponent.vue

<script setup>
const props = defineProps({
  myMode: {
    type: String,
    require: true
  }
})
</script>

<template>
  <span>My mode: {{ props.myMode }}</span>
</template>
```

注意事项：

1. 最好使用详细的定义 Props 的结构，便于维护和开发。
2. 定义 Props 名时使用 camelCase 方式命名，而在祖先对 Props 传参时，使用 kebab-case 方式命名。
3. Props 遵循单向数据流的原则，会因组件组件的更新而变化，后代组件原则上不能修改 prop 的值。
4. 可以利用引用类型的 Props 来实现后代组件修改 prop，算是打破单向数据流原则的一个技巧，但不推荐这样做。
5. 可以配合 `v-model` 进行隐式传递 prop。

### Emits 事件

监听和触发事件的方式也是比较常用的组件通信的方式。

下面是一个简单的例子：

```other
// @/components/emitsExample/ParentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {ref} from "vue";

const myMode = ref('learning')
const changeMyMode = mode => {
  myMode.value = mode
}
</script>

<template>
  <ChildComponent :my-mode="myMode" @change-my-mode="changeMyMode"/>
</template>
```

```other
// @/components/emitsExample/ChildComponent.vue

<script setup>
const props = defineProps({
  myMode: {
    type: String,
    require: true
  }
})
const emit = defineEmits({
  changeMyMode: mode => {
    // 只做简单的字符串类型判断用于示例
    if (typeof mode === "string") {
      console.log('The type of mode is legal.')
      return true
    } else return false
  }
})
</script>

<template>
  <span>My mode: {{ props.myMode }}</span>
  <div>change my mode</div>
  <button @click="$emit('changeMyMode', 'working')">Working</button>
  <button @click="emit('changeMyMode','sleeping')">Sleeping</button>
</template>
```

注意事项：

1. 最好使用详细的定义 Emits 的结构，同时尽可能的进行校验，便于维护和开发。
2. 定义 Emits 名时使用 camelCase 方式命名，而在祖先组件对 Emits 监听时，使用 kebab-case 方式命名。
3. Emits 的校验函数无论是使用 `$emit()` ，还是使用`defineProps()` 方式触发事件，都会执行校验函数。
4. 可以配合 `v-model` 进行隐式传递事件。

### $parent API

Vue 提供了 `$parent` API，用于访问祖先组件实例。

下面是一个简单的例子：

```other
// @components/ParentExample/ParentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {ref} from "vue";

const myMode = ref('learning')
const changeMyMode = mode => {
  myMode.value = mode
}
defineExpose({myMode, changeMyMode})
</script>

<template>
  <ChildComponent/>
</template>
```

```other
// @components/ParentExample/ChildComponent.vue

<script setup>
import {getCurrentInstance} from 'vue'

const currentInstance = getCurrentInstance().proxy
const parent = currentInstance.$parent
</script>

<template>
  <span>My mode: {{ $parent.myMode }}</span>
  <div>So, I am {{ parent.myMode }}.</div>
  <button @click="$parent.changeMyMode('working')">Working</button>
  <button @click="parent.changeMyMode('sleeping')">Sleeping</button>
</template>
```

注意事项：

1. 使用 `<script setup>` 的组件需要使用 `defineExpose()` API 暴露属性，这样才可以配合 `$parent` 实现组件通信。
2. `$parent` 可以在后代组件的模版中直接使用。
3. 使用 `<script setup>` 的组件没有 `this`，所以无法直接调用 `$parent`，可以使用 `getCurrentInstance` 方法来调用 `$parent`。
4. 根组件的 `$parent` 为 `null`。

### $attrs API

Vue 提供了 `$attrs` API，用于访问祖先组件的透传属性。

下面是一个简单的例子：

```other
// @/components/attrsExample/ParentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {ref} from "vue";

const myMode = ref('learning')
const changeMyMode = mode => {
  myMode.value = mode
}
</script>

<template>
  <ChildComponent :myMode="myMode" :changeMyMode="changeMyMode"/>
</template>
```

```other
// @/components/attrsExample/ChildComponent.vue

<script setup>
import {useAttrs} from 'vue'

const attrs = useAttrs()
</script>

<template>
  <span>My mode: {{attrs.myMode}}</span>
  <div>So I am {{$attrs.myMode}}.</div>
  <button @click="attrs.changeMyMode('sleeping')">Sleeping</button>
  <button @click="$attrs.changeMyMode('working')">Working</button>
</template>
```

注意事项：

1. 上例中，是因为使用了 `:myMode=“myMode”` ，所以实现了 `$attrs` 的响应式效果，`$attrs` 并不是响应式的。
2. 利用单根节点的透传属性，可以实现多代的组件通信。

### $refs API

Vue 提供了 `$refs` API，用于访问后代组件实例。

下面是一个简单的例子：

```other
// @/components/refsExample/ParentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {onMounted, ref} from "vue"

const child = ref(null)
const showState = ref(false)
onMounted(() => {
  showState.value = true
})
</script>

<template>
  <ChildComponent ref="child"/>
  <template v-if="showState">
    <span>My mode: {{ child.myMode }}</span>
    <div>So I am {{ $refs.child.myMode }}.</div>
    <button @click="child.changeMyMode('sleeping')">Sleeping</button>
    <button @click="$refs.child.changeMyMode('working')">Working</button>
  </template>
</template>
```

```other
// @/components/refsExample/ChildComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {onMounted, ref} from "vue"

const child = ref(null)
const showState = ref(false)
onMounted(() => {
  showState.value = true
})
</script>

<template>
  <ChildComponent ref="child"/>
  <template v-if="showState">
    <span>My mode: {{ child.myMode }}</span>
    <div>So I am {{ $refs.child.myMode }}.</div>
    <button @click="child.changeMyMode('sleeping')">Sleeping</button>
    <button @click="$refs.child.changeMyMode('working')">Working</button>
  </template>
</template>
```

注意事项：

1. 使用 `<script setup>` 的组件没有 `this`，所以无法直接调用 `$refs`，通过创建与后代组件 ref 同名的 `ref(null)`，实现对后代组件实例的访问。
2. 通过 `$refs` 访问后代组件的实例的时机需要在后代组件挂载到祖先组件上之后，也就是说，一般是在 `onMounted()` 中，对后代组件进行访问。

### $root API

Vue 提供了 `$root` API，用于访问根组件实例。

下面是一个简单的例子：

```other
// @/App.vue

<script setup>
import {ref} from "vue";

const myMode = ref('learning')
const changeMyMode = mode => {
  myMode.value = mode
}
defineExpose({myMode, changeMyMode})
</script>

<template>
//...
</template>
```

```other
// @/components/rootExample/ChildComponent.vue

<script setup>
import {getCurrentInstance} from "vue";

const currentInstance = getCurrentInstance().proxy
const root = currentInstance.$root
</script>

<template>
  <span>My mode:{{$root.myMode}}</span>
  <div>So I am {{root.myMode}}</div>
  <button @click="$root.changeMyMode('working')">Working</button>
  <button @click="root.changeMyMode('sleeping')">Sleeping</button>
</template>
```

注意事项：

1. 由于此例中通过路由系统进行演示，故根组件应为 `App.vue`。
2. 使用 `<script setup>` 的组件需要使用 `defineExpose()` API 暴露属性，这样才可以配合 `$root` 实现组件通信。
3. `$root` 可以在后代组件的模版中直接使用。
4. 使用 `<script setup>` 的组件没有 `this`，所以无法直接调用 `$root`，可以使用 `getCurrentInstance` 方法来调用 `$root`。
5. 根组件的 `$root` 为根组件本身。

### provide/inject

除了 Props 和 Emits 的方式外，Vue 同时推荐我们使用 provide/inject 的方式进行跨组件的通信。

下面是一个简单的例子：

```other
// @/components/provideAndInjectExample/ParentComponent.vue

<script setup>
import ChildComponent from './ChildComponent.vue'
import {provide, ref} from "vue"

const myMode = ref('learning')
const changeMyMode = mode => {
  myMode.value = mode
}
provide('modeObject', {
  myMode,
  changeMyMode
})

</script>
<template>
  <ChildComponent></ChildComponent>
</template>
```

```other
// @/components/provideAndInjectExample/ChildComponent.vue

<script setup>
import {inject} from "vue"

const {myMode, changeMyMode} = inject('modeObject')
</script>

<template>
  <div>My mode: {{ myMode }}</div>
  <button @click="changeMyMode('working')">Working</button>
</template>
```

注意事项：

1. `provide()` 方法的第一个参数的类型不仅可以是字符串，还可以是 Symbol，使用 Symbol 类型数据作为 key，配合单独管理键名的文件，可以避免潜在的命名冲突异常。
2. 可以使用 `app.provide()` 在 app 层级注入数据。

### Pinia 状态管理

Pinia 是官方替代 Vuex 的状态管理工具。

下面是一个简单的例子：

```other
// @/components/pages/PiniaExample.vue

<script setup>
import {storeToRefs} from 'pinia'
import {useModeStore} from '@/stores/mode.js'

const modeStore = useModeStore()
const {myMode} = storeToRefs(modeStore)
const mutateState = (mode) => {
  modeStore.$patch(state => {
    state.myMode = mode
  })
}
const changeMyMode = (mode) => {
  modeStore.changeMyMode(mode)
}
const resetState = () => {
  modeStore.$reset()
}
</script>

<template>
  <div>My mode: {{ myMode }}</div>
  <div>So my mode is {{ modeStore.getMyMode }}.</div>
  <button @click="mutateState('working')">Working</button>
  <button @click="changeMyMode('Sleeping')">Sleeping</button>
  <button @click="resetState()">Reset</button>
</template>
```

具体用法可以看 Pinia 文档。

我在读文档的时候写了这么一篇博客「[Pinia 入门](https://github.com/sad912/sad-blogs/blob/main/Pinia%20入门｜看文档写%20Demo.md)」。

## 使用场景的分类

### 后代组件与祖先组件通信

1. Props 声明
2. Emit 事件
3. $attrs API
4. $parent API
5. $root API
6. $ref API
7. provide/inject API
8. Pinia

### 同代组件通信

1. $parent API
2. provide/inject API
3. Pinia

## 跨层级组件通信

1. provide/inject API
2. $root API
3. Pinia

以上。

