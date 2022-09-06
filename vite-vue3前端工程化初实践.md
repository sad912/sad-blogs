本文详细介绍了使用 `Vite` 构建工具进行前端工程化配置的过程，其中：

- 核心技术栈包括 `Vue 3`、`TypeScript` 和 `TSX`；
- 使用 `Vue Router 4` 作为路由管理工具；
- 使用 `Pinia` 作为状态管理工具；
- 使用 `Unocss` 为项目提供 CSS 预设和创建原子化配置；
- 通过 `ESlint` 规范代码格式；

# Vite 构建

使用命令行搭建 `Vite` 项目，命令如下：

```Bash
npm create vite@latest
// or
yarn create vite
// or
pnpm create vite
```

此文以 `pnpm` 为例，根据提示完成项目的构建，日志如下：

```Bash
? project-name: > vite-starter-template
? select a framework: > vue
? select a variant: > vue-ts

Scaffolding project in /.../vite-starter-template...

Done. Now run:

  cd vite-starter-template
  pnpm install
  pnpm run dev
```

运行提示的命令后，项目就启动了。

至此，我们使用 `Vite` 构建了使用`TypeScript` 的 `Vue3` 项目。

# TSX

`Vite` 为 `Vue` 提供第一优先级支持，所以官方提供了 `@vitejs/plugin-vue-jsx` 插件，它提供了 `Vue 3` 特性的支持，包括 `HMR`，全局组件解析，指令和插槽。

键入以下命令安装 `@vitejs/plugin-vue-jsx` 插件：

```Bash
pnpm i @vitejs/plugin-vue-jsx -D
```

在 `Vite` 配置文件 `vite.config.ts` 中配置此插件，如下所示：

```other
// ./vite.conifg.ts

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsxPlugin from "@vitejs/plugin-vue-jsx"; //new

export default defineConfig({
    plugins: [
        vue(),
        vueJsxPlugin({}) //new
    ]
})
```

创建一个检查 `TSX` 是否引入的测试组件。命令如下：

```other
touch src/components/VueJsxTestComponent.tsx
echo 'export default () => <div>test</div>' > src/components/VueJsxTestComponent.tsx
```

并在 `App.vue` 中引入。

```other
// ./src/App.vue

<script setup lang="ts">
  import Test from './components/VueJsxTestComponent'
</script>

<template>
  <Test></Test>
</template>

<style scoped>
</style>
```

此时，浏览器中可以正常显示 `test` 字样，`TSX` 可以正常使用。

至此，我们完成了 `TSX` 的配置。

# Vue Router 4

键入以下命令安装 Vue Router 4：

```Bash
pnpm i vue-router@4
```

创建 `router` 目录，并创建配置路由的文件。如下所示：

```Bash
mkdir ./src/router
touch ./src/router/index.ts
```

创建 `pages` 目录，并创建一个简单的 `index` 页面。如下所示：

```Bash
mkdir ./src/pages/
touch ./src/pages/index.tsx
echo 'export default () => <div>index</div>' > ./src/pages/index.tsx
```

使用此 `index.tsx` 和先前测试用的 `VueJsxTestComponent.tsx` 两个文件配置路由进行测试。如下所示：

```other
// ./src/router/index.ts

import { createRouter, createWebHashHistory } from 'vue-router'
import Index from '../pages/index'
import Test from "../components/VueJsxTestComponent";

const routes = [
    {
        path: '/',
        name: 'index',
        component: Index
    },
    {
        path: '/test',
        name: 'Test',
        component: Test
    }
]

const router = createRouter({
    history: createWebHashHistory(),
    routes
})

export default router
```

在 `main.ts` 中引入路由配置文件，并挂载到实例上。如下：

```other
// ./main.ts

import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from './router/index'


createApp(App).use(router).mount('#app')
```

在 `App.vue` 中测试路由组件是否能正常使用。如下：

```other
// ./App.vue

<script setup lang="ts">
</script>

<template>
  <router-link to="/">Index</router-link>
  |
  <router-link to="/test">Test</router-link>
  <router-view></router-view>
</template>

<style scoped>
</style>
```

此时，浏览器中可以通过路由链接切换路由，并显示对应组件。

至此，我们完成了 `Vue Router 4` 的配置。

# Pinia

键入以下命令安装 `Pinia`：

```Bash
pnpm add pinia
```

创建 `store` 目录，并创建配置 `Pinia` 的文件。如下所示：

```Bash
mkdir ./src/store/
touch ./src/store/index.ts
```

此 `index.ts` 中配置如下：

```other
// ./src/store/index.ts

import { defineStore } from 'pinia'

export default defineStore('store', {
    // other options...
})
```

在 `main.ts` 中，将创建的 `store` 挂载到实例上，如下所示：

```other
// ./main.ts

import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from './router/index'
import store from './store/index'

createApp(App).use(router, store).mount('#app')
```

至此，我们完成了 `Pinia` 的简单配置。

# UnoCSS

键入以下命令安装 `UnoCSS`：

```Bash
pnpm i -D unocss
```

在 `Vite` 配置文件 `vite.config.ts` 中配置此插件，如下所示：

```other
// ./vite.conifg.ts

import {defineConfig} from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsxPlugin from "@vitejs/plugin-vue-jsx";
import Unocss from "unocss/vite"                    //new
import {presetAttributify, presetUno} from 'unocss' //new

export default defineConfig({
    plugins: [
        vue(),
        vueJsxPlugin(),
        Unocss({								          //new
            presets: [
                presetAttributify({ /* preset options */}),
                presetUno(),
                // ...custom presets
            ],
            rules: [
                ['m-1', { margin: '0.25rem' }],
            ]
        })
    ]
})

```

上例中使用了 `UnoCSS` 官方文档中推荐的预设，同时添加了 `m-1` 规则用于测试。

将`uno.css` 添加 `main.ts` 中。

```other
// ./main.ts

import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from './router/index'
import store from './store/index'
import 'uno.css'

createApp(App).use(router, store).mount('#app')
```

在 `App.vue` 中将 `router-view` 标签添加 `m-1` 属性，测试是否生效。

至此，我们完成了 `UnoCSS` 的基础配置。

# ESLint

键入以下命令初始化 ESLint 配置文件：

```Bash
pnpx eslint --init
```

控制台打印如下：

```Bash
Need to install the following packages:
  @eslint/create-config@0.3.1
Ok to proceed? (y) y
? How would you like to use ESLint? · style
? What type of modules does your project use? · esm
? Which framework does your project use? · vue
? Does your project use TypeScript? · Yes
? Where does your code run? · browser
? How would you like to define a style for your project? · guide
? Which style guide do you want to follow? · standard-with-typescript
? What format do you want your config file to be in? · JavaScript
Checking peerDependencies of eslint-config-standard-with-typescript@latest
Local ESLint installation not found.
The config that you've selected requires the following dependencies:

eslint-plugin-vue@latest eslint-config-standard-with-typescript@latest @typescript-eslint/eslint-plugin@^5.0.0 eslint@^8.0.1 eslint-plugin-import@^2.25.2 eslint-plugin-n@^15.0.0 eslint-plugin-promise@^6.0.0 typescript@*
✔ Would you like to install them now? · No / Yes
✔ Which package manager do you want to use? · pnpm
Installing eslint-plugin-vue@latest, eslint-config-standard-with-typescript@latest, @typescript-eslint/eslint-plugin@^5.0.0, eslint@^8.0.1, eslint-plugin-import@^2.25.2, eslint-plugin-n@^15.0.0, eslint-plugin-promise@^6.0.0, typescript@*
Packages: +158
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Progress: resolved 478, reused 310, downloaded 58, added 158, done

devDependencies:
+ @typescript-eslint/eslint-plugin 5.33.1
+ eslint 8.22.0
+ eslint-config-standard-with-typescript 22.0.0
+ eslint-plugin-import 2.26.0
+ eslint-plugin-n 15.2.4
+ eslint-plugin-promise 6.0.0
+ eslint-plugin-vue 9.3.0

The integrity of 1502 files was checked. This might have caused installation to take longer.
Successfully created .eslintrc.cjs file in /vite-starter-template
```

此时 `package.json` 文件已经发生变化，同时，根目录多了一个 `.eslintrc.cjs` 文件，对其进行如下配置：

```Bash
// ./eslintrc.cjs

module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: [
    'plugin:vue/vue3-essential',
    'standard-with-typescript'
  ],
  overrides: [],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: ['./tsconfig.json', './tsconfig.node.json'] //new
  },
  plugins: [
    'vue'
  ],
  rules: {}
}
```

同时，修改 `tsconfig.node.json` 文件：

```bash
// ./tsconfig.node.json

{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true,
    "strictNullChecks": true	//new
  },
  "include": [
    "vite.config.ts"
  ]
}
```

在 `package.json` 中添加 `lint` 脚本，如下所示：

```bash
// ./package.json

{
  ...
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.vue,.tsx --fix"      //new
  },
  ...
}
```

在终端运行如下命令实现命令 `lint` 操作：

```bash
pnpm lint
```

至此，我们完成了 `ESLint` 的基本配置。
