# Vue 单向绑定全解

# 什么是单向绑定

在开发实践中，我们经常会将 Model 绑定到 View，当使用代码来更新 Model 时，View 就会自动更新，这是单向绑定。而对应到 Vue 开发实践中，当我们将 Vue 实例中的数据渲染到 DOM 元素中，当我们修改 Vue 实例中的数据时，对应 DOM 元素中的内容也会发生变化，这就是 Vue 的单向绑定。

# 单向绑定实践

Vue 通过 v-bind 指令实现单向绑定，动态的绑定一个或多个 attribute，同时 v-bind 也可以绑定组件的 prop。

下面是一些简单的双向绑定实例。

```other
<!-- 绑定 attribute -->
<img v-bind:src="imageSrc" />

<!-- 动态 attribute 名 -->
<button v-bind:[key]="value"></button>

<!-- 缩写 -->
<img :src="imageSrc" />

<!-- 缩写形式的动态 attribute 名 -->
<button :[key]="value"></button>

<!-- 内联字符串拼接 -->
<img :src="'/path/to/images/' + fileName" />
```

当用于绑定 `class` 或 `style` attribute 时，`v-bind` 支持额外的值类型如数组或对象。

```other
<!-- class 绑定 -->
<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, { classB: isB, classC: isC }]"></div>

<!-- style 绑定 -->
<div :style="{ fontSize: size + 'px' }"></div>
<div :style="[styleObjectA, styleObjectB]"></div>
```

当用于组件 props 绑定时，所绑定的 props 必须在子组件中已被正确声明。

```other
<!-- prop 绑定。"prop" 必须在子组件中已声明。 -->
<MyComponent :prop="someThing" />

<!-- 传递子父组件共有的 prop -->
<MyComponent v-bind="$props" />
```

当不带参数使用时，可以用于绑定一个包含了多个 attribute 名称-绑定值对的对象。

```other
<!-- 绑定对象形式的 attribute -->
<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

<!-- 传递父组件传递进来的 attrs -->
<div v-bind="$attrs"></div>
```

同时，v-bind 指令还提供了一些修饰符

`.camel` 修饰符可以驼峰化 `v-bind` attribute 的名称。

```other
<svg :view-box.camel="viewBox"></svg>
```

`.prop` 修饰符可以强制绑定为 DOM property，同时提供了其缩写符 `.`。

```other
<div :someProperty.prop="someObject"></div>

<!-- 等同于 -->
<div .someProperty="someObject"></div>
```

`.attr` 修饰符可以强制绑定为 DOM attribute。

```other
<div :someAttribute.attr="someAttribute"></div>
```

---

完。

