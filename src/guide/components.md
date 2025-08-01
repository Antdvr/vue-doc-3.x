---
aside: false
---

# Component 组件使用

Vue 单文件组件一般有如下 3 种加载引用方式

- 按需加载组件
- 全局注册组件
- 局部注册组件

## 按需加载组件

在使用 Ant Design Vue 官方组件时，**按需加载** 是我们比较推荐的一种方式。而在 **Antdv.pro** 中，则主要借助如下 vite 插件来实现：

::: details unplugin-vue-components => 自动按需引用 Ant Design Vue 组件 {open}

::: code-group

```vue:line-numbers [范例 - AButton]
<template>
  <div class="section-container">
    <AButton>Button</AButton>
  </div>
</template>

<script setup lang="ts">
// 这个例子无需 import 导入, 就可以使用 AButton 组件
// 但是若需要在 script 引用，则还是需要 import 导入 (例: Tag)

import ATag from "ant-design-vue/es/tag"
const ATagName = ATag.name
</script>
```

```typescript:line-numbers{10-17} [插件配置 - vite.config.ts]
import { defineConfig } from "vitest/config"
import { AntDesignVueResolver } from "unplugin-vue-components/resolvers"
import AutoComponents from "unplugin-vue-components/vite"
import VueJsx from "@vitejs/plugin-vue-jsx"
import Vue from "@vitejs/plugin-vue"

export default defineConfig(() => {
  return {
    plugins: [
      AutoComponents({
        resolvers: [
          AntDesignVueResolver({
            resolveIcons: true,
            importStyle: 'less',
          }),
        ],
      }),
      VueJsx(),
      Vue(),
    ],
  }
})
```

```typescript:line-numbers{4} [TS支持 - components.d.ts]
/* eslint-disable */
/* prettier-ignore */
// @ts-nocheck
// Generated by unplugin-vue-components
// Read more: https://github.com/vuejs/core/pull/3399
export {}

declare module "vue" {
  export interface GlobalComponents {
    AButton: typeof import("ant-design-vue/es")["Button"];
  }
}
```

:::

::: details unplugin-auto-import => 自动按需导入 Vue、Vue-Router 等 API

::: code-group

```vue:line-numbers [范例 - Vue API]
<template>
  <div class="section-container">
    <AButton>{{ button }}</AButton>
  </div>
</template>

<script setup lang="ts">
// 下面两个例子:
//    --- 无需从 Vue 导入 ref
//    --- 无需从 Vue-Router 导入 useRouter

const button = ref("Button")
const router = useRouter()

const back = () => {
  router.go(-1)
}
</script>
```

```typescript:line-numbers{9-18} [插件配置 - vite.config.ts]
import { defineConfig } from "vitest/config"
import AutoImport from "unplugin-auto-import/vite"
import VueJsx from "@vitejs/plugin-vue-jsx"
import Vue from "@vitejs/plugin-vue"

export default defineConfig(() => {
  return {
    plugins: [
      AutoImport({
        include: [/\.[tj]sx?$/, /\.vue\?vue/, /\.vue$/],
        imports: ["vue", "pinia", "vue-router"],
        eslintrc: {
          enabled: true,
          filepath: "./.eslintrc-auto-import.json",
          globalsPropValue: true
        },
        dts: true
      }),
      VueJsx(),
      Vue(),
    ],
  }
})
```

:::

## 全局注册组件

除了 Ant Deisgn Vue v4.x 官方组件，Antdv.pro 还提供了 **Library-3.x 组件库** (提供了 SForm、STable、STree 等高阶组件)。

::: details 在使用 Library-3.x 组件时，我们采用全局注册方式来声明定义组件 {open}

::: code-group

```vue:line-numbers [范例 - SForm]
<template>
  <section class="form-container">
    <SForm
      v-model="model"
      :groups="groups"
    />
  </section>
</template>

<script setup lang="ts">
/**
 * Now you don't need to manually import
 * When using @antdvr/library-3.x/resolver by unplugin-auto-import
 */
// import { formGroupsDefiner } from '@antdvr/library-3.x'

defineOptions({
  name: 'TestForm',
  inheritAttrs: false,
})

const model: Ref<any> = ref({})
const groups = formGroupsDefiner([
  {
    type: 'AInput',
    slot: 'title',
    field: 'title',
    label: '名称',
    props: {
      placeholder: '请输入名称',
    },
  },
])
</script>
```

```typescript:line-numbers{3,9} [全局注册 - main.ts]
import App from "./App.vue"
import DirectivePlugin from "@/configure/presetDirective"
import ComponentPlugin from "@antdvr/library-3.x"
import PiniaterPlugin from "@/plugin/pinia"
import RouterPlugin from "@/router"

createApp(App)
  .use(DirectivePlugin)
  .use(ComponentPlugin)
  .use(PiniaterPlugin)
  .use(RouterPlugin)
  .mount("#app")
```

```typescript:line-numbers [Vite配置 - vite.config.ts]
import { defineConfig } from 'vitest/config'
import { AntdLibraryResolver } from '@antdvr/library-3.x/resolver' // [!code highlight]
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig(({ mode }) => {
  return {
    plugins: [
      AutoImport({
        resolvers: [
          AntdLibraryResolver(), // [!code highlight]
        ],
        eslintrc: {
          enabled: true,
          filepath: './.eslintrc-auto-import.json',
          globalsPropValue: true,
        },
        dts: true,
      }),
    ],
  }
})
```

```typescript:line-numbers [TS配置 - tsconfig.json]
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "types": [
      "@antdvr/library-3.x/globals", // [!code highlight]
      "vite/client"
    ]
  }
}
```

```typescript:line-numbers [ESLint配置 - eslint.config.mjs]
import eslint from '@eslint/js'
import tseslint from 'typescript-eslint'
import pluginVue from 'eslint-plugin-vue'
import stylistic from '@stylistic/eslint-plugin'
import templater from '@antdvr/eslint-config-3.x' // [!code highlight]

const flatArray = options => {
  return !Array.isArray(options)
    ? [options]
    : options
}

export default tseslint.config(
  {
    extends: [
      ...flatArray(eslint.configs.recommended),
      ...flatArray(tseslint.configs.recommended),
      ...flatArray(stylistic.configs['recommended']),
      ...flatArray(pluginVue.configs['flat/recommended']),
      ...flatArray(templater.configs['flat/recommended']), // [!code highlight]
    ],
  },
)
```

:::

## 局部注册组件

Antdv.pro 在开发 **业务页面** 时，往往需要封装定义 **业务级组件**。而在使用这些组件时，我们通常会使用 import 导入，进行局部注册。

::: details 组件局部注册 {open}

::: code-group

```vue:line-numbers{3,8} [范例 - 父组件]
<template>
  <section class="page-container">
    <BusinessTable>
  </section>
</template>

<script setup lang="ts">
import BusinessTable from './BusinessTable.vue'

defineOptions({
  name: 'BusinessPage',
  inheritAttrs: false,
})
</script>
```

```vue:line-numbers [范例 - 子组件]
<template>
  <section class="table-container">
    <STable>
  </section>
</template>

<script setup lang="ts">
defineOptions({
  name: 'BusinessTable',
  inheritAttrs: false,
})
</script>
```

:::
