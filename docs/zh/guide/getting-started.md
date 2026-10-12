---
description: '安装 vue-qs，并使用 useQueryState 和 useQueryStates 将 Vue 3 状态保存到 URL 中：可写 ref、类型化解析器以及 v-model 支持。'
---

# 快速开始

vue-qs 是一个面向 Vue 3 的类型安全搜索参数状态管理库，是
[nuqs](https://nuqs.dev) 的 Vue 移植版本：它提供相同的解析器、选项和行为，
并通过可写 ref 和响应式对象暴露这些功能。

```ts
const page = useQueryState('page', parseAsInteger.withDefault(1));
page.value++; // ?page=2
```

## 安装

::: code-group

```sh [npm]
npm install vue-qs
```

```sh [pnpm]
pnpm add vue-qs
```

```sh [bun]
bun add vue-qs
```

:::

需要 Vue 3.3 或更高版本。`vue-router`（4.2+ 或 5）是可选的 peer dependency。

## 安装插件

vue-qs 通过 **adapter** 读取和写入 URL。请根据应用使用的路由方案安装对应的插件：

::: code-group

```ts [Vue Router]
import { createApp } from 'vue';
import { createVueQs } from 'vue-qs';
import { createVueRouterAdapter } from 'vue-qs/adapters/vue-router';
import { router } from './router';

createApp(App)
  .use(router)
  .use(createVueQs({ adapter: createVueRouterAdapter(router) }))
  .mount('#app');
```

```ts [无路由器]
import { createApp } from 'vue';
import { createVueQs } from 'vue-qs';
import { createHistoryAdapter } from 'vue-qs/adapters/history';

createApp(App)
  .use(createVueQs({ adapter: createHistoryAdapter() }))
  .mount('#app');
```

:::

使用 Nuxt？请参阅[适配器 → Nuxt](../../guide/adapters#nuxt)。

## 第一个查询状态

```vue
<script setup lang="ts">
import { useQueryState } from 'vue-qs';

const name = useQueryState('name'); // Ref<string | null>
</script>

<template>
  <input v-model="name" placeholder="Your name" />
  <p>Hello, {{ name || 'anonymous visitor' }}!</p>
  <button @click="name = null">Clear</button>
</template>
```

在输入框中输入内容会更新 URL 中的 `?name=…`，浏览器前进或后退时，输入框也会同步更新。

## 使用解析器管理类型化状态

不使用解析器时，状态类型为 `string | null`。传入一个[解析器](../../guide/parsers)，
并使用 `.withDefault()` 来移除 `null`：

```ts
import { parseAsBoolean, parseAsInteger, useQueryState } from 'vue-qs';

const page = useQueryState('page', parseAsInteger.withDefault(1)); // Ref<number>
const open = useQueryState('open', parseAsBoolean); // Ref<boolean | null>
```

当键不存在或其值无效时，状态会回退到默认值。将状态设置为默认值（或 `null`）会从 URL 中移除该键。

## 更新状态

可以赋值给 `.value`，也可以调用 `.set()` 来传入[选项](../../guide/options)，
或等待 URL 更新完成：

```ts
page.value = 2;

await page.set((current) => current + 1); // updater function
await page.set(3, { history: 'push' }); // add a history entry
await page.set(null); // remove ?page (page.value === 1)

const search = await page.set(4); // URLSearchParams after the update
```

同一个 tick 中的更新会合并为一次 URL 更新。

## 同时管理多个键

`useQueryStates` 将多个键绑定到一个响应式对象：

```vue
<script setup lang="ts">
import { toRefs } from 'vue';
import { parseAsFloat, useQueryStates } from 'vue-qs';

const coordinates = useQueryStates({
  lat: parseAsFloat.withDefault(45.18),
  lng: parseAsFloat.withDefault(5.72),
});

const { lat, lng } = toRefs(coordinates); // writable refs, for v-model

function reset() {
  coordinates.$set(null); // removes both keys in one update
}
</script>

<template>
  <input v-model.number="lat" type="number" />
  <input v-model.number="lng" type="number" />
</template>
```

## 下一步

- [解析器](../../guide/parsers)：所有内置解析器以及如何编写自定义解析器
- [选项](../../guide/options)：历史记录、滚动、批处理、节流和防抖
- [适配器](../../guide/adapters)：Vue Router、Nuxt、原生 History API 和自定义适配器
- [服务端](../../guide/server)：`createLoader` 和 `createSerializer`
- [测试](../../guide/testing)：测试使用 vue-qs 的组件
- [从 0.1.x 迁移](../../guide/migration)
