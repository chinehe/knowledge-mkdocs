## 1. 应用入口 main.ts

一个 Vue 3 应用从 `main.ts` 开始。入口文件做三件事：创建应用实例、安装插件、挂载到页面。

### 1.1. 创建实例与安装插件

```typescript
// src/main.ts
import './assets/main.css'

import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'
import { useAuthStore } from '@/stores/auth'

const app = createApp(App)

app.use(createPinia())
app.use(router)

// 启动时若本地有令牌，异步恢复登录态（不阻塞首屏渲染）
useAuthStore().restore()

app.mount('#app')
```

`createApp(App)` 以根组件 `App.vue` 创建一个应用实例。实例上通过 `app.use(plugin)` 安装插件，插件安装后其能力才对整个组件树可用：

| 调用 | 作用 |
| ---- | ---- |
| `app.use(createPinia())` | 注册状态管理，之后任何组件里才能调用 `useXxxStore()` |
| `app.use(router)` | 注册路由，之后模板里才能用 `<RouterView>`、`<RouterLink>`，组件里才能用 `useRouter()` / `useRoute()` |

### 1.2. 装配顺序：先 Pinia 再 router

`createPinia()` 要在 `router` 之前安装。原因是路由守卫（如全局前置守卫）里常常需要读取 store（例如判断是否已登录来决定放行还是跳转登录页）。如果 Pinia 还没安装就去用 store，会取不到实例。按"先数据、后路由"的顺序安装，可以避免这类初始化顺序问题。

`app.use()` 返回实例本身，所以也可以链式写成 `app.use(createPinia()).use(router)`，效果一致。

### 1.3. 启动时异步恢复登录态

页面刷新后，内存里的组件状态全部丢失，但浏览器 `localStorage` 里的令牌还在。入口处调用一次 store 的 `restore()`，用本地令牌向后端换回当前用户信息，把登录态"恢复"到内存：

```typescript
// 不写 await：恢复过程在后台进行，不挡住首屏
useAuthStore().restore()

app.mount('#app')
```

这里**故意不 `await`**。`restore()` 要发一个网络请求，若等它返回再 `mount`，首屏会被网络延迟卡住、白屏时间变长。不等待则先把界面渲染出来，用户信息稍后到达再更新。由于 store 的 `user` 是响应式的，恢复完成后依赖它的组件会自动重渲染，无需手动刷新。

对需要登录才能看的页面，不阻塞带来的副作用是：恢复完成前 `isAuthenticated` 短暂为 `false`。这类"首帧未知登录态"通常由路由守卫或页面上的加载态处理，和入口处是否等待无关。

## 2. 根组件与 `<script setup>`

### 2.1. 单文件组件（SFC）

Vue 的 `.vue` 文件是单文件组件，把一个组件的三部分写在同一个文件里：

| 块 | 内容 |
| ---- | ---- |
| `<script setup lang="ts">` | 组件逻辑（用 TypeScript） |
| `<template>` | 结构（HTML 模板） |
| `<style>` | 样式（可选，加 `scoped` 则只作用于当前组件） |

### 2.2. 根组件挂 RouterView

根组件 `App.vue` 通常很薄，主要职责是放置路由出口和全局常驻的 UI：

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { RouterView } from 'vue-router'

import ThemeToggle from '@/components/ThemeToggle.vue'
</script>

<template>
  <ThemeToggle />
  <RouterView />
</template>
```

`<RouterView>` 是路由出口：当前 URL 匹配到哪个路由，对应的页面组件就渲染在这个位置。放在它外面的组件（如这里的主题切换按钮）不随路由变化，在所有页面里都常驻显示。

### 2.3. `<script setup>` 比 Options API 简洁在哪

`<script setup>` 是组合式 API 的语法糖。对照两种写法引入并使用一个子组件：

```vue
<!-- Options API：要在 components 里登记，逻辑拆在 data/methods/computed 各块 -->
<script lang="ts">
import ThemeToggle from '@/components/ThemeToggle.vue'

export default {
  components: { ThemeToggle },
  data() {
    return { count: 0 }
  },
}
</script>
```

```vue
<!-- script setup：import 进来即可用，顶层变量直接暴露给模板 -->
<script setup lang="ts">
import { ref } from 'vue'
import ThemeToggle from '@/components/ThemeToggle.vue'

const count = ref(0)
</script>
```

差别在于：

- 导入的组件**不用再在 `components` 里登记**，import 进来就能在模板里用。
- `<script setup>` 块里的顶层变量、函数自动暴露给模板，不必通过 `return` 或 `this` 转发。
- 相关逻辑可以写在一起，而不是被 `data`、`methods`、`computed` 切成几段；配合 `<script setup>` 顶层的类型推断，TypeScript 体验也更好。

## 3. Vue Router：路由配置

### 3.1. 创建 router 实例

路由集中在一个模块里创建并导出，再在入口 `app.use(router)` 安装：

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '@/views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView,
    },
    {
      path: '/login',
      name: 'login',
      // 懒加载：访问 /login 时才下载这块代码
      component: () => import('@/views/LoginView.vue'),
    },
  ],
})

export default router
```

### 3.2. history 模式与 BASE_URL

`createWebHistory()` 启用 HTML5 History 模式，URL 形如 `/login`，干净、没有 `#`。与之相对的是 `createWebHashHistory()`，URL 形如 `/#/login`。

| 模式 | 创建函数 | URL 样子 | 代价 |
| ---- | -------- | -------- | ---- |
| History | `createWebHistory()` | `/login` | 需要服务器把未匹配的路径都回退到 `index.html`，否则刷新子路由会 404 |
| Hash | `createWebHashHistory()` | `/#/login` | 无需服务器配置，但 URL 带 `#` |

传入的 `import.meta.env.BASE_URL` 是构建工具注入的"部署基路径"。应用部署在域名根目录时它是 `/`；部署在子路径（如 `/app/`）时它是 `/app/`。把它交给 `createWebHistory`，路由在拼接地址时就会自动带上这个前缀，换部署位置无需改路由代码。

### 3.3. 路由配置项

`routes` 数组里每条路由常用三个字段：

| 字段 | 作用 |
| ---- | ---- |
| `path` | URL 路径，如 `/`、`/login` |
| `name` | 路由名，跳转时可用 `router.push({ name: 'login' })` 代替手写路径，改路径时不必全局替换 |
| `component` | 该路径渲染的页面组件 |

### 3.4. 路由懒加载

`component` 有两种写法，决定了页面组件的代码何时下载：

```typescript
import HomeView from '@/views/HomeView.vue'   // 顶部静态 import：同步引入

{ path: '/', component: HomeView },                       // 直接用，打包进主包
{ path: '/login', component: () => import('@/views/LoginView.vue') }, // 懒加载
```

- **同步引入**：组件在应用启动时就随主包一起加载。首页这类进入应用几乎必然要看的页面适合同步引入，省去一次额外请求。
- **懒加载** `() => import('...')`：写成返回动态 `import()` 的函数。构建工具会把这个组件单独切成一个代码块（chunk），只有路由真正匹配到该路径时才下载。

收益是减小首屏需要下载的体积：登录页、设置页这类次要或不一定访问的页面做成懒加载，用户不去就永远不下载。划分原则是**首屏必经的页面同步引入，其余懒加载**。

## 4. Pinia setup store

Pinia 是 Vue 官方的状态管理库，用于在组件之间共享状态。它提供两种定义风格，这里用贴近组合式 API 的 **setup store**。

### 4.1. 定义风格对比

`defineStore` 的第二个参数可以是一个配置对象（options store），也可以是一个函数（setup store）：

```typescript
// options 写法：像 Options API，用 state/getters/actions 三个固定字段
export const useCounter = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: { double: (s) => s.count * 2 },
  actions: {
    inc() {
      this.count++
    },
  },
})
```

```typescript
// setup 写法：像 script setup，用 ref / computed / 普通函数自由组织
export const useCounter = defineStore('counter', () => {
  const count = ref(0)
  const double = computed(() => count.value * 2)
  function inc() {
    count.value++
  }
  return { count, double, inc }
})
```

setup store 把一个函数交给 `defineStore`，函数体内用组合式 API 的原语组织状态：

| 概念 | setup store 里的写法 | 对应 options store |
| ---- | -------------------- | ------------------ |
| state（状态） | `ref()` 定义的变量 | `state` |
| getter（派生状态） | `computed()` | `getters` |
| action（动作/方法） | 普通函数（可以是 `async`） | `actions` |

最后 `return` 一个对象，**只有被 return 出来的成员**才是 store 对外暴露的内容。它和 `<script setup>` 的心智模型一致：会写组件就会写 store，不必在两套 API 之间切换。

### 4.2. 使用 store

在组件里调用 `useXxxStore()` 拿到 store 实例，读属性、调方法即可。注意用 `ref` 定义的 state 在组件里直接用 `store.xxx` 读取，不需要 `.value`（`.value` 只在 store 定义内部使用）：

```typescript
const auth = useAuthStore()

auth.isAuthenticated   // 读派生状态
await auth.login(name, pwd)  // 调 action
```

## 5. 实例：认证状态 store

把前面几点串起来，看一个维护"当前登录用户"的认证 store：

```typescript
// src/stores/auth.ts
import { computed, ref } from 'vue'
import { defineStore } from 'pinia'

import * as authApi from '@/api/auth'
import { clearToken, getAccessToken, getRefreshToken, setToken } from '@/api/token'
import type { User } from '@/api/types'

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)

  const isAuthenticated = computed(() => user.value !== null)

  async function login(username: string, password: string): Promise<void> {
    const tokens = await authApi.login(username, password)
    setToken(tokens.access_token, tokens.refresh_token)
    user.value = await authApi.fetchMe()
  }

  async function logout(): Promise<void> {
    const refreshToken = getRefreshToken()
    try {
      if (refreshToken) {
        await authApi.logout(refreshToken)
      }
    } finally {
      // 不管后端成不成，本地都清掉
      clearToken()
      user.value = null
    }
  }

  // 应用启动时若本地有令牌，尝试用它恢复登录态
  async function restore(): Promise<void> {
    if (!getAccessToken()) {
      return
    }
    try {
      user.value = await authApi.fetchMe()
    } catch {
      clearToken()
      user.value = null
    }
  }

  return { user, isAuthenticated, login, logout, restore }
})
```

### 5.1. 状态与派生状态

唯一的"真相来源"是 `user`：登录成功后是用户对象，未登录是 `null`。`isAuthenticated` 用 `computed` 从 `user` 派生，不单独存一个布尔值。好处是不会出现"`user` 有值但 `isAuthenticated` 为 `false`"这种两份状态对不上的情况——派生状态永远跟着来源自动更新。

### 5.2. 三个 action 如何维护 user

| action | 做什么 | 结束后 `user` |
| ------ | ------ | ------------- |
| `login` | 用账号密码换令牌并保存，再拉取用户信息 | 当前用户 |
| `logout` | 通知后端令牌失效，清掉本地令牌 | `null` |
| `restore` | 启动时用本地已有令牌拉回用户信息 | 当前用户或 `null` |

两个细节：

- `logout` 把清理放在 `finally` 里。只要用户点了退出，无论通知后端的请求成功还是失败，本地令牌和内存状态都要清干净，不能因为网络出错就让用户"退不出去"。
- `restore` 在拉取失败时（如令牌已过期）走 `catch`，清掉无效令牌并置空 `user`，相当于回到未登录状态，而不是抛错让启动流程中断。

### 5.3. 分层：store 管状态，令牌读写在 API 层

注意这个 store 不直接碰 `localStorage`，而是调用 `@/api/token` 暴露的 `setToken` / `getAccessToken` / `clearToken`。令牌具体存在哪、用什么键名、怎么读写，被封装在独立的令牌模块里；store 只关心"登录态"这件事。

```typescript
// src/api/token.ts —— 令牌读写集中在这里，用中性前缀给键名做命名空间
const ACCESS_KEY = 'myapp.access_token'
const REFRESH_KEY = 'myapp.refresh_token'

export function getAccessToken(): string | null {
  return localStorage.getItem(ACCESS_KEY)
}
// setToken / getRefreshToken / clearToken 同理
```

这样分层的好处是职责清晰：store 是状态层，令牌存取和请求携带令牌属于 API 层。将来要把令牌从 `localStorage` 换成别的存储，只改令牌模块一处，store 不受影响。令牌模块与请求拦截的细节属于 API 层的话题，这里只点到为止。

## 6. 小结

| 主题 | 要点 |
| ---- | ---- |
| 应用入口 | `createApp(App)` 建实例，`app.use()` 装插件，`app.mount('#app')` 挂载 |
| 装配顺序 | 先 `createPinia()` 后 `router`，便于路由守卫里读 store |
| 恢复登录态 | 入口调 `restore()` 不 `await`，先渲染首屏，用户信息稍后到达靠响应式更新 |
| 根组件 | `App.vue` 很薄，放 `<RouterView>` 和全局常驻 UI |
| `<script setup>` | 组件免登记、顶层变量直接给模板用，比 Options API 简洁 |
| Router | `createWebHistory(import.meta.env.BASE_URL)` 启用 History 模式并带部署基路径；路由配 `path`/`name`/`component` |
| 懒加载 | 首屏必经页面同步引入，次要页面用 `() => import()` 按需下载 |
| setup store | `defineStore(name, setup)`：`ref` 作 state、`computed` 作派生、函数作 action，`return` 暴露 |
| 认证 store | 以 `user` 为唯一真相，`isAuthenticated` 派生；`login`/`logout`/`restore` 维护 `user` |
| 分层 | store 管状态，令牌读写封装在 API 层，互不耦合 |
