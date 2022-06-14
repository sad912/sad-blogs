# Vue 2 和 Vue 3 实现响应式过程——数据劫持篇

## 什么是响应式

响应式，即数据会根据和此数据有关的数据的变化而响应变化。

下面是一个简单易懂的响应式例子：

```other
let bar = 1
let foo = bar
bar = 2
// 如果数据为响应式的，那么此时的foo = 2
```

## 响应式原理

实现上述响应式的关键，是如何在修改 `bar` 时，自动实现 `foo` 的响应式变化。

我把实现响应式的过程分为三步，一是我们可以监听 `bar` 的修改，这是触发响应式变化的时机，又称「数据劫持」；二是我们可以找出和 `bar` 有关的数据，又称「依赖收集」；三是对和 `bar` 有关的数据进行响应式更新，又称「发布订阅」。

本篇主要介绍第一步，也就是数据劫持。

## 数据劫持

### `Object.defineProperty()` API

Vue 2通过 `Object.defineProperty` API 作为数据劫持的工具。

O`bject.defineProperty` API 可以通过 d`escriptor` 对象参数中的 s`et` 属性和 g`et` 属性分别设置访问对象的属性时的拦截函数。

下面是一个简单的例子。

```other
let foo
const bar = {}
// bar 定义一个 foo 属性
Object.defineProperty(bar, 'foo', {
	get() {
		return foo
	},
	set(value) {
		foo = value
	}
})
// 此时，修改 bar.foo 的同时，也会修改 foo。
// 也就是说 foo 会根据 bar.foo 的更新，而发生响应式变化。 
bar.foo = 2
foo // 2
```

Vue 2 通过 `Object.defineProperty()` API ，可以比较方便地得到实现数据劫持，但 `Object.defineProperty()` API 实现响应式也有一些缺陷。

`Object.defineProperty()` API 是对响应式对象属性进行的数据劫持，每当添加一个属性，我们都需要对其进行数据劫持，这就导致了自动添加劫持这个行为无法通过 JavaScript 原生 API 实现。Vue 2 是通过遍历 Vue 实例的 data 选项中的对象的属性帮我们自动实现了 data 选项中的数据劫持，但有些场景需要在用户直接操作响应式数据时就要实现数据劫持的，这时 Vue2 就提供了一些用于实现数据劫持的 API。

1. 当你想在响应式对象或数组上添加属性时，Vue 2 提供了 `Vue.set(object, propertyName, value)` API 。
2. 使用 `delete` 删除对象属性时，数据劫持也随之消失了。故 Vue 2 提供了 `Vue.delete(target, propertyName/index)` API。

```other
delete bar.foo
foo // 值为 2，理想情况下 foo 的值应该为 undefined
```

1. Vue 2 重写了操作数组的原生 API。

Vue 2 在官方文档的深入响应式原理一节中，说明了 Vue 2 无法自动实现劫持的所有情况，并提供了 API 来处理这些情况。

## Proxy 对象

vue 3 `Rective` 选择 `Proxy` 对象作为数据劫持的工具，一个重要的原因就是`Proxy` 对象解决了 `Object.defineProperty()` 的一些缺陷。

`Proxy` 对象可以通过 `handler` 对象参数中的 `set` 、`get` 和 `deleteProperty` 属性来声明对象对应操作的拦截函数。

下面是一个简单的例子，功能和 `Object.defineProperty` API 实现的类似，但此例可以监听到属性的删除。

```other
let foo
// bar 定义一个 foo 属性
const bar = new Proxy({}, {
	get(target, property) {
		return target[property]
	},
	set(target, property, value) {
		target[property] = value
		if (property === 'foo')
			foo = value
	},
	deleteProperty(target, property) {
		if (property === 'foo')
			foo = undefined
		return true
	}
})
// 此时，修改 bar.foo 的同时，也会修改 foo。
// 也就是说 foo 会根据 bar.foo 的更新，而发生响应式变化。 
bar.foo = 2
foo // 2
delete bar.foo
foo // undefined
```

可以注意到，`Proxy` 对象是针对目标对象来监听，而不是针对对象的某个具体属性，也就是说，`Proxy` 对象可以代理那些定义时不存在的属性，同时可以通过 `deleteProperty` 实现对删除操作的代理。另外，`Proxy` 对象支持的拦截操作有13 种，这为不同情况的数据拦截提供了极大的便利。

### Object setter 和 getter

Vue 3 中的 `ref` 的实现也用到了对象属性的 `set` 和 `get` 函数，来实现对 value 属性的修改时的响应式监听。

```other
let foo
const bar = {
  get foo() {
    return _foo
  },
  set foo(value) {
    _foo = value
    foo = value
  }
}
// 此时，修改 bar.foo 的同时，也会修改 foo。
// 也就是说 foo 会根据 bar.foo 的更新，而发生响应式变化。 
bar.foo = 2
foo // 2
```

至此，关于 Vue 2 和 Vue3 是如何实现数据劫持的，我有了初步的思考的认识。

