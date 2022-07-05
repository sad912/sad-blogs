# Vue 3 中 v-if 和 v-for 混用时优先级的变化

本文通过一个简单的例子，从范例、渲染函数、源码和实践的角度，深入探讨Vue 2 和 Vue 3 中 v-if 和 v-for 混用时优先级的变化。

# Vue 2

## 例子

Vue 2 时，指令`v-for` 的优先级高于指令 `v-if`，我们经常会有如下代码场景：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vue 2 demo</title>
</head>

<body>
  <ul id="app">
    <li v-for="item in items" v-if="item.state" :key="item.id">
      {{item.value}}
    </li>
  </ul>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

  <script>
    const app = new Vue({
      el: '#app',
      data() {
        return {
          items: [
            { id: 1, value: 'v-for', state: true },
            { id: 2, value: 'v-if', state: false }
          ]
        }
      },
    })
    console.log(app.$options.render);   
  </script>
</body>

</html>
```

## 渲染函数

例子中的浏览器控制台打印的渲染函数如下：

```javascript
(function anonymous(
) {
with(this){return _c('ul',{attrs:{"id":"app"}},_l((items),function(item){return (item.state)?_c('li',{key:item.id},[_v("\n      "+_s(item.value)+"\n    ")]):_e()}),0)}
})
```

通过渲染函数可以看出，Vue 2 是先进行 item 的遍历，再进行  state 的判断。

## 源码

查找浏览器资源中的 vue.js 源代码，如下：

```javascript
function genElement(el, state) {
      if (el.parent) {
          el.pre = el.pre || el.parent.pre;
      }
      if (el.staticRoot && !el.staticProcessed) {
          return genStatic(el, state);
      }
      else if (el.once && !el.onceProcessed) {
          return genOnce(el, state);
      }
      // 二者优先级
      else if (el.for && !el.forProcessed) {
          return genFor(el, state);
      }
      else if (el.if && !el.ifProcessed) {
          return genIf(el, state);
      }
      else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
          return genChildren(el, state) || 'void 0';
      }
      else if (el.tag === 'slot') {
          return genSlot(el, state);
      }
      else {
          // component or element
          var code = void 0;
          if (el.component) {
              code = genComponent(el.component, el, state);
          }
          else {
              var data = void 0;
              if (!el.plain || (el.pre && state.maybeComponent(el))) {
                  data = genData(el, state);
              }
              var tag 
              // check if this is a component in <script setup>
              = void 0;
              // check if this is a component in <script setup>
              var bindings = state.options.bindings;
              if (bindings && bindings.__isScriptSetup !== false) {
                  tag =
                      checkBindingType(bindings, el.tag) ||
                          checkBindingType(bindings, camelize(el.tag)) ||
                          checkBindingType(bindings, capitalize(camelize(el.tag)));
              }
              if (!tag)
                  tag = "'".concat(el.tag, "'");
              var children = el.inlineTemplate ? null : genChildren(el, state, true);
              code = "_c(".concat(tag).concat(data ? ",".concat(data) : '' // data
              ).concat(children ? ",".concat(children) : '' // children
              , ")");
          }
          // module transforms
          for (var i = 0; i < state.transforms.length; i++) {
              code = state.transforms[i](el, code);
          }
          return code;
      }
}
```

`genElement` 方法用于创建 DOM 元素，不难看出，执行 `genFor()` 方法是比执行 `genIf()` 的优先级更高的。

# Vue 3

## 例子

在 Vue 3 中，如果混用指令 `v-for` 和指令 `v-if` 且使用遍历项的属性作为 `v-if` 指令的判断条件，浏览器控制台会报错，这是因为 Vue 3 中的指令`v-if` 的优先级是高于指令 `v-for` 的，还没有创建遍历项自然就访问不到遍历项的属性了。

下面是一个简单的例子，我们通过 `virtualState` 来尝试将两个指令混用，打印其渲染函数。

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <!-- 过滤列表中项目 -->
  <!-- 浏览器控制台会报错：Uncaught TypeError: Cannot read properties of undefined (reading 'state') -->
  <!-- <ul id="app">
    <li v-for="item in items" v-if="item.state" :key="item.id">
      {{ item.value }}
    </li>
  </ul> -->

  <!-- 避免渲染应该被隐藏的列表 -->
  <ul id="app">
    <li v-for="item of items" v-if="virtualState" :key="item.id">
      {{ item.value }}
    </li>
  </ul>
  <script src="http://unpkg.com/vue@3"></script>
  <script>
    const app = Vue.createApp({
      data() {
        return {
          virtualState: true,
 		   items: [
            { id: 1, value: 'v-for', state: true },
            { id: 2, value: 'v-if', state: false }
          ]
        }
      },
    }).mount('#app')
    console.log(app.$options.render);
  </script>
</body>

</html>
```

## 渲染函数

例子中的浏览器控制台打印的渲染函数如下：

```javascript
(function anonymous(
) {
const _Vue = Vue

return function render(_ctx, _cache) {
  with (_ctx) {
    const { renderList: _renderList, Fragment: _Fragment, openBlock: _openBlock, createElementBlock: _createElementBlock, toDisplayString: _toDisplayString, createCommentVNode: _createCommentVNode } = _Vue

    return virtualState
      ? (_openBlock(true), _createElementBlock(_Fragment, { key: 0 }, _renderList(items, (item) => {
          return (_openBlock(), _createElementBlock("li", { key: item.id }, _toDisplayString(item.value), 1 /* TEXT */))
        }), 128 /* KEYED_FRAGMENT */))
      : _createCommentVNode("v-if", true)
  }
}
})
```

通过渲染函数可以看出，先判断了 `virtualState` 的值，再进行了遍历创建 DOM 元素，故当使用遍历项的属性作为判断条件的值会产生报错，同时这也表明指令 `v-if` 的优先级是高于指令 `v-for` 的。

## 源码

查找浏览器资源中的 vue.js 源代码，如下：

```javascript
function genNode(node, context) {
      if (isString(node)) {
          context.push(node);
          return;
      }
      if (isSymbol(node)) {
          context.push(context.helper(node));
          return;
      }
      switch (node.type) {
          case 1 /* ELEMENT */:
		   // 二者优先级
          case 9 /* IF */:
          case 11 /* FOR */:
              assert(node.codegenNode != null, `Codegen node is missing for element/if/for node. ` +
                      `Apply appropriate transforms first.`);
              genNode(node.codegenNode, context);
              break;
          case 2 /* TEXT */:
              genText(node, context);
              break;
          case 4 /* SIMPLE_EXPRESSION */:
              genExpression(node, context);
              break;
          case 5 /* INTERPOLATION */:
              genInterpolation(node, context);
              break;
          case 12 /* TEXT_CALL */:
              genNode(node.codegenNode, context);
              break;
          case 8 /* COMPOUND_EXPRESSION */:
              genCompoundExpression(node, context);
              break;
          case 3 /* COMMENT */:
              genComment(node, context);
              break;
          case 13 /* VNODE_CALL */:
              genVNodeCall(node, context);
              break;
          case 14 /* JS_CALL_EXPRESSION */:
              genCallExpression(node, context);
              break;
          case 15 /* JS_OBJECT_EXPRESSION */:
              genObjectExpression(node, context);
              break;
          case 17 /* JS_ARRAY_EXPRESSION */:
              genArrayExpression(node, context);
              break;
          case 18 /* JS_FUNCTION_EXPRESSION */:
              genFunctionExpression(node, context);
              break;
          case 19 /* JS_CONDITIONAL_EXPRESSION */:
              genConditionalExpression(node, context);
              break;
          case 20 /* JS_CACHE_EXPRESSION */:
              genCacheExpression(node, context);
              break;
          case 21 /* JS_BLOCK_STATEMENT */:
              genNodeList(node.body, context, true, false);
              break;
          // SSR only types
          case 22 /* JS_TEMPLATE_LITERAL */:
              break;
          case 23 /* JS_IF_STATEMENT */:
              break;
          case 24 /* JS_ASSIGNMENT_EXPRESSION */:
              break;
          case 25 /* JS_SEQUENCE_EXPRESSION */:
              break;
          case 26 /* JS_RETURN_STATEMENT */:
              break;
          /* istanbul ignore next */
          case 10 /* IF_BRANCH */:
              // noop
              break;
          default:
              {
                  assert(false, `unhandled codegen node type: ${node.type}`);
                  // make sure we exhaust all possible types
                  const exhaustiveCheck = node;
                  return exhaustiveCheck;
              }
      }
  }
```

`genNode` 方法用于创建 DOM 节点，不难看出，当同时具有两个 IF  类型和 FOR 类型的节点在创建时，会按 IF 类型进行处理，故在 Vue 3 中，指令 `v-if` 的优先级是高于指令 `v-for` 的。