# 前端开发最佳实践

> AI全流程研发知识库 - Vue.js/TypeScript开发最佳实践
>
> **版本**: v1.0
> **更新日期**: 2026-02-13
> **适用范围**: Vue 3 + TypeScript前端开发

## 目录

- [1. 项目结构](#1-项目结构)
- [2. 组件设计](#2-组件设计)
- [3. 状态管理](#3-状态管理)
- [4. 路由管理](#4-路由管理)
- [5. API调用](#5-api调用)
- [6. 错误处理](#6-错误处理)
- [7. 性能优化](#7-性能优化)
- [8. TypeScript实践](#8-typescript实践)
- [9. 代码规范](#9-代码规范)
- [10. 测试策略](#10-测试策略)

---

## 1. 项目结构

### 1.1 标准目录结构

```
frontend/
├── public/                       # 静态资源
│   ├── favicon.ico
│   └── index.html
├── src/
│   ├── api/                      # API调用
│   │   ├── request.ts            # Axios配置
│   │   ├── user.ts               # 用户相关API
│   │   └── order.ts              # 订单相关API
│   ├── assets/                   # 资源文件
│   │   ├── images/               # 图片
│   │   ├── icons/                # 图标
│   │   └── styles/               # 全局样式
│   │       ├── variables.css     # CSS变量
│   │       ├── reset.css         # 样式重置
│   │       └── common.css        # 公共样式
│   ├── components/               # 通用组件
│   │   ├── common/               # 基础组件
│   │   │   ├── Button.vue
│   │   │   ├── Input.vue
│   │   │   └── Modal.vue
│   │   └── business/             # 业务组件
│   │       ├── UserCard.vue
│   │       └── OrderList.vue
│   ├── composables/              # 组合式函数
│   │   ├── useUser.ts
│   │   ├── useAuth.ts
│   │   └── usePagination.ts
│   ├── directives/               # 自定义指令
│   │   ├── permission.ts
│   │   └── loading.ts
│   ├── layouts/                  # 布局组件
│   │   ├── DefaultLayout.vue
│   │   └── BlankLayout.vue
│   ├── router/                   # 路由
│   │   ├── index.ts
│   │   └── routes.ts
│   ├── store/                    # 状态管理
│   │   ├── modules/
│   │   │   ├── user.ts
│   │   │   └── app.ts
│   │   └── index.ts
│   ├── types/                    # 类型定义
│   │   ├── user.ts
│   │   ├── order.ts
│   │   └── common.ts
│   ├── utils/                    # 工具函数
│   │   ├── format.ts
│   │   ├── validate.ts
│   │   └── storage.ts
│   ├── views/                    # 页面组件
│   │   ├── home/
│   │   │   └── Index.vue
│   │   ├── user/
│   │   │   ├── Login.vue
│   │   │   └── Profile.vue
│   │   └── order/
│   │       ├── List.vue
│   │       └── Detail.vue
│   ├── App.vue                   # 根组件
│   └── main.ts                   # 入口文件
├── tests/                        # 测试文件
│   ├── unit/
│   └── e2e/
├── .env.development              # 开发环境变量
├── .env.production               # 生产环境变量
├── .eslintrc.js                  # ESLint配置
├── .prettierrc                   # Prettier配置
├── tsconfig.json                 # TypeScript配置
├── vite.config.ts                # Vite配置
└── package.json
```

---

## 2. 组件设计

### 2.1 组件拆分原则

```
单一职责原则：
- 一个组件只做一件事
- 组件代码不超过300行
- 复杂组件拆分为多个子组件

✅ 好的拆分：
components/
├── UserCard.vue           # 用户卡片
├── UserAvatar.vue         # 用户头像
├── UserInfo.vue           # 用户信息
└── UserActions.vue        # 用户操作按钮

❌ 不好的拆分：
components/
└── UserComponent.vue      # 所有用户相关功能都在一个组件里
```

### 2.2 组件通信

#### Props传递（父 → 子）
```vue
<!-- ✅ 好的实践：使用TypeScript定义Props -->
<script setup lang="ts">
interface Props {
  userId: number
  username: string
  email?: string
  isActive?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  email: '',
  isActive: true
})
</script>

<template>
  <div class="user-card">
    <h3>{{ props.username }}</h3>
    <p>{{ props.email }}</p>
  </div>
</template>
```

#### Emits触发（子 → 父）
```vue
<!-- 子组件 -->
<script setup lang="ts">
interface Emits {
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
}

const emit = defineEmits<Emits>()

const handleUpdate = (value: string) => {
  emit('update', value)
}

const handleDelete = (id: number) => {
  emit('delete', id)
}
</script>

<!-- 父组件 -->
<template>
  <ChildComponent
    @update="handleUpdate"
    @delete="handleDelete"
  />
</template>
```

#### Provide/Inject（跨层级）
```vue
<!-- 祖先组件 -->
<script setup lang="ts">
import { provide, ref } from 'vue'

const theme = ref('light')

provide('theme', theme)
</script>

<!-- 后代组件 -->
<script setup lang="ts">
import { inject, type Ref } from 'vue'

const theme = inject<Ref<string>>('theme')
</script>
```

### 2.3 组件最佳实践

```
✅ 好的实践：
- 使用Composition API
- Props和Emits使用TypeScript定义
- 组件名使用PascalCase
- 文件名使用PascalCase或kebab-case
- 使用<script setup>语法糖
- 样式使用scoped

❌ 不好的实践：
- 使用Options API（除非必要）
- Props没有类型定义
- 组件名使用camelCase
- 样式没有scoped，污染全局
- 直接修改Props
```

---

## 3. 状态管理

### 3.1 Pinia基本使用

```typescript
// store/modules/user.ts
import { defineStore } from 'pinia'
import type { User } from '@/types/user'
import { userApi } from '@/api/user'

interface UserState {
  currentUser: User | null
  token: string | null
  loading: boolean
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    currentUser: null,
    token: localStorage.getItem('token'),
    loading: false
  }),

  getters: {
    isLoggedIn: (state) => !!state.token,

    displayName: (state) => {
      return state.currentUser?.name || '游客'
    }
  },

  actions: {
    /**
     * 登录
     */
    async login(username: string, password: string) {
      this.loading = true
      try {
        const res = await userApi.login({ username, password })
        this.token = res.token
        this.currentUser = res.user
        localStorage.setItem('token', res.token)
      } catch (error) {
        throw error
      } finally {
        this.loading = false
      }
    },

    /**
     * 登出
     */
    logout() {
      this.token = null
      this.currentUser = null
      localStorage.removeItem('token')
    },

    /**
     * 获取当前用户信息
     */
    async fetchCurrentUser() {
      if (!this.token) return

      try {
        this.currentUser = await userApi.getCurrentUser()
      } catch (error) {
        this.logout()
      }
    }
  }
})
```

#### 组件中使用Store
```vue
<script setup lang="ts">
import { useUserStore } from '@/store/modules/user'

const userStore = useUserStore()

const handleLogin = async () => {
  try {
    await userStore.login(username.value, password.value)
    router.push('/dashboard')
  } catch (error) {
    console.error('登录失败', error)
  }
}
</script>

<template>
  <div>
    <p>当前用户: {{ userStore.displayName }}</p>
    <button v-if="!userStore.isLoggedIn" @click="handleLogin">
      登录
    </button>
  </div>
</template>
```

### 3.2 状态管理最佳实践

```
✅ 好的实践：
- 全局状态用Pinia管理
- 本地状态用ref/reactive
- Store按功能模块划分
- Actions处理异步逻辑
- Getters派生计算属性

❌ 不好的实践：
- 所有状态都放在Store
- 直接修改State（应该用Actions）
- 一个Store包含所有功能
- 在组件中处理复杂逻辑
```

---

## 4. 路由管理

### 4.1 路由配置

```typescript
// router/routes.ts
import type { RouteRecordRaw } from 'vue-router'

export const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/layouts/DefaultLayout.vue'),
    children: [
      {
        path: '',
        name: 'Home',
        component: () => import('@/views/home/Index.vue'),
        meta: {
          title: '首页',
          requiresAuth: false
        }
      },
      {
        path: 'dashboard',
        name: 'Dashboard',
        component: () => import('@/views/dashboard/Index.vue'),
        meta: {
          title: '工作台',
          requiresAuth: true
        }
      },
      {
        path: 'users',
        name: 'UserList',
        component: () => import('@/views/user/List.vue'),
        meta: {
          title: '用户管理',
          requiresAuth: true,
          permission: 'user:view'
        }
      }
    ]
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/auth/Login.vue'),
    meta: {
      title: '登录',
      requiresAuth: false
    }
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('@/views/error/404.vue')
  }
]
```

### 4.2 路由守卫

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import { useUserStore } from '@/store/modules/user'
import { routes } from './routes'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes
})

/**
 * 全局前置守卫
 */
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore()

  // 设置页面标题
  document.title = to.meta.title as string || '应用名称'

  // 检查是否需要认证
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    next({
      name: 'Login',
      query: { redirect: to.fullPath }
    })
    return
  }

  // 检查权限
  if (to.meta.permission) {
    const hasPermission = await checkPermission(to.meta.permission as string)
    if (!hasPermission) {
      next({ name: 'Forbidden' })
      return
    }
  }

  next()
})

/**
 * 全局后置钩子
 */
router.afterEach((to, from) => {
  // 页面切换后滚动到顶部
  window.scrollTo(0, 0)
})

export default router
```

### 4.3 路由最佳实践

```
✅ 好的实践：
- 使用路由懒加载
- 路由meta定义页面信息
- 使用路由守卫处理权限
- 统一处理404
- 路由命名使用PascalCase

❌ 不好的实践：
- 所有路由组件直接import
- 权限判断分散在各个组件
- 没有404页面
- 路由命名不规范
```

---

## 5. API调用

### 5.1 Axios配置

```typescript
// api/request.ts
import axios, { type AxiosInstance, type AxiosError } from 'axios'
import { useUserStore } from '@/store/modules/user'
import router from '@/router'

// 创建Axios实例
const request: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000
})

/**
 * 请求拦截器
 */
request.interceptors.request.use(
  (config) => {
    const userStore = useUserStore()

    // 添加Token
    if (userStore.token) {
      config.headers.Authorization = `Bearer ${userStore.token}`
    }

    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

/**
 * 响应拦截器
 */
request.interceptors.response.use(
  (response) => {
    const { code, data, message } = response.data

    if (code === 0) {
      return data
    }

    // 业务错误
    ElMessage.error(message || '请求失败')
    return Promise.reject(new Error(message || '请求失败'))
  },
  (error: AxiosError<{ message: string }>) => {
    // HTTP错误
    if (error.response) {
      switch (error.response.status) {
        case 401:
          // 未授权，跳转登录
          useUserStore().logout()
          router.push('/login')
          break
        case 403:
          ElMessage.error('没有权限')
          break
        case 404:
          ElMessage.error('请求的资源不存在')
          break
        case 500:
          ElMessage.error('服务器错误')
          break
        default:
          ElMessage.error(error.response.data?.message || '请求失败')
      }
    } else if (error.request) {
      ElMessage.error('网络错误，请检查网络连接')
    }

    return Promise.reject(error)
  }
)

export default request
```

### 5.2 API封装

```typescript
// api/user.ts
import request from './request'
import type { User, LoginRequest, LoginResponse } from '@/types/user'

export const userApi = {
  /**
   * 登录
   */
  login(data: LoginRequest): Promise<LoginResponse> {
    return request.post('/api/auth/login', data)
  },

  /**
   * 获取当前用户
   */
  getCurrentUser(): Promise<User> {
    return request.get('/api/users/current')
  },

  /**
   * 获取用户列表
   */
  getUsers(params: {
    page: number
    size: number
    keyword?: string
  }): Promise<{
    list: User[]
    total: number
  }> {
    return request.get('/api/users', { params })
  },

  /**
   * 创建用户
   */
  createUser(data: Partial<User>): Promise<User> {
    return request.post('/api/users', data)
  },

  /**
   * 更新用户
   */
  updateUser(id: number, data: Partial<User>): Promise<User> {
    return request.put(`/api/users/${id}`, data)
  },

  /**
   * 删除用户
   */
  deleteUser(id: number): Promise<void> {
    return request.delete(`/api/users/${id}`)
  }
}
```

### 5.3 API调用最佳实践

```
✅ 好的实践：
- 统一配置Axios
- API按模块封装
- 使用TypeScript类型
- 统一处理错误
- 使用环境变量配置URL

❌ 不好的实践：
- 组件中直接使用axios
- API调用分散在各处
- 没有类型定义
- 错误处理分散
- 硬编码API地址
```

---

## 6. 错误处理

### 6.1 全局错误处理

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// 全局错误处理
app.config.errorHandler = (err, instance, info) => {
  console.error('Global error:', err)
  console.error('Component:', instance)
  console.error('Error info:', info)

  // 上报错误到监控系统
  reportError({
    error: err,
    component: instance?.$options.name,
    info
  })
}

app.mount('#app')
```

### 6.2 组件错误边界

```vue
<!-- ErrorBoundary.vue -->
<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

const error = ref<Error | null>(null)

onErrorCaptured((err, instance, info) => {
  error.value = err
  console.error('Error captured:', err, info)

  // 返回false阻止错误继续传播
  return false
})

const retry = () => {
  error.value = null
}
</script>

<template>
  <div v-if="error" class="error-boundary">
    <h2>出错了</h2>
    <p>{{ error.message }}</p>
    <button @click="retry">重试</button>
  </div>
  <slot v-else />
</template>
```

### 6.3 异步错误处理

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { userApi } from '@/api/user'

const loading = ref(false)
const error = ref<string | null>(null)
const users = ref<User[]>([])

const fetchUsers = async () => {
  loading.value = true
  error.value = null

  try {
    const data = await userApi.getUsers({ page: 1, size: 10 })
    users.value = data.list
  } catch (err) {
    error.value = err instanceof Error ? err.message : '加载失败'
  } finally {
    loading.value = false
  }
}

const retry = () => {
  fetchUsers()
}
</script>

<template>
  <div>
    <div v-if="loading">加载中...</div>
    <div v-else-if="error" class="error-state">
      <p>{{ error }}</p>
      <button @click="retry">重试</button>
    </div>
    <div v-else-if="users.length === 0" class="empty-state">
      暂无数据
    </div>
    <div v-else>
      <UserCard v-for="user in users" :key="user.id" :user="user" />
    </div>
  </div>
</template>
```

---

## 7. 性能优化

### 7.1 组件懒加载

```typescript
// ✅ 好的实践：路由懒加载
const routes = [
  {
    path: '/users',
    component: () => import('@/views/user/List.vue')
  }
]

// ✅ 好的实践：动态组件懒加载
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

const HeavyComponent = defineAsyncComponent(() =>
  import('@/components/HeavyComponent.vue')
)
</script>
```

### 7.2 列表优化

```vue
<!-- ✅ 好的实践：虚拟滚动 -->
<script setup lang="ts">
import { ref, computed } from 'vue'

const items = ref<Item[]>([...])  // 10000条数据
const scrollTop = ref(0)
const itemHeight = 50
const visibleCount = 20

const visibleItems = computed(() => {
  const start = Math.floor(scrollTop.value / itemHeight)
  const end = start + visibleCount
  return items.value.slice(start, end)
})

const totalHeight = computed(() => items.value.length * itemHeight)
const offsetY = computed(() =>
  Math.floor(scrollTop.value / itemHeight) * itemHeight
)

const handleScroll = (e: Event) => {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}
</script>

<template>
  <div class="virtual-list" @scroll="handleScroll">
    <div :style="{ height: `${totalHeight}px` }">
      <div :style="{ transform: `translateY(${offsetY}px)` }">
        <div
          v-for="item in visibleItems"
          :key="item.id"
          :style="{ height: `${itemHeight}px` }"
        >
          {{ item.name }}
        </div>
      </div>
    </div>
  </div>
</template>
```

### 7.3 计算属性缓存

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const items = ref<Item[]>([...])

// ✅ 好的实践：使用computed缓存结果
const expensiveResult = computed(() => {
  return items.value
    .filter(item => item.active)
    .map(item => ({
      ...item,
      displayName: formatName(item.name)
    }))
})

// ❌ 不好的实践：每次渲染都重新计算
const getExpensiveResult = () => {
  return items.value
    .filter(item => item.active)
    .map(item => ({
      ...item,
      displayName: formatName(item.name)
    }))
}
</script>

<template>
  <!-- ✅ 使用computed -->
  <div v-for="item in expensiveResult" :key="item.id">
    {{ item.displayName }}
  </div>

  <!-- ❌ 每次渲染都调用函数 -->
  <div v-for="item in getExpensiveResult()" :key="item.id">
    {{ item.displayName }}
  </div>
</template>
```

### 7.4 防抖和节流

```typescript
// composables/useDebounce.ts
import { ref, customRef } from 'vue'

export function useDebouncedRef<T>(value: T, delay = 300) {
  let timeout: ReturnType<typeof setTimeout>

  return customRef<T>((track, trigger) => ({
    get() {
      track()
      return value
    },
    set(newValue) {
      clearTimeout(timeout)
      timeout = setTimeout(() => {
        value = newValue
        trigger()
      }, delay)
    }
  }))
}
```

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import { useDebouncedRef } from '@/composables/useDebounce'
import { userApi } from '@/api/user'

const searchKeyword = ref('')
const debouncedKeyword = useDebouncedRef(searchKeyword.value, 500)

watch(debouncedKeyword, async (keyword) => {
  if (keyword) {
    // 防抖后的搜索
    const results = await userApi.search(keyword)
  }
})
</script>

<template>
  <input v-model="searchKeyword" placeholder="搜索用户" />
</template>
```

---

## 8. TypeScript实践

### 8.1 类型定义

```typescript
// types/user.ts

/**
 * 用户状态枚举
 */
export enum UserStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  BANNED = 'banned'
}

/**
 * 用户接口
 */
export interface User {
  id: number
  username: string
  email: string
  status: UserStatus
  createdAt: string
  updatedAt: string
}

/**
 * 登录请求
 */
export interface LoginRequest {
  username: string
  password: string
}

/**
 * 登录响应
 */
export interface LoginResponse {
  token: string
  user: User
  expiresIn: number
}

/**
 * 分页请求
 */
export interface PageRequest {
  page: number
  size: number
}

/**
 * 分页响应
 */
export interface PageResponse<T> {
  list: T[]
  total: number
  page: number
  size: number
}
```

### 8.2 泛型使用

```typescript
// composables/usePagination.ts
import { ref, computed } from 'vue'
import type { PageRequest, PageResponse } from '@/types/common'

export function usePagination<T>(
  fetchFn: (params: PageRequest) => Promise<PageResponse<T>>
) {
  const loading = ref(false)
  const list = ref<T[]>([])
  const total = ref(0)
  const page = ref(1)
  const size = ref(10)

  const totalPages = computed(() => Math.ceil(total.value / size.value))

  const fetch = async () => {
    loading.value = true
    try {
      const res = await fetchFn({ page: page.value, size: size.value })
      list.value = res.list
      total.value = res.total
    } finally {
      loading.value = false
    }
  }

  const nextPage = () => {
    if (page.value < totalPages.value) {
      page.value++
      fetch()
    }
  }

  const prevPage = () => {
    if (page.value > 1) {
      page.value--
      fetch()
    }
  }

  return {
    loading,
    list,
    total,
    page,
    size,
    totalPages,
    fetch,
    nextPage,
    prevPage
  }
}
```

---

## 9. 代码规范

### 9.1 命名规范

```typescript
// ✅ 好的命名
// 组件名：PascalCase
const UserCard = defineComponent({...})

// 变量名：camelCase
const userName = ref('')
const isLoading = ref(false)

// 常量：UPPER_SNAKE_CASE
const MAX_PAGE_SIZE = 100
const API_BASE_URL = '/api'

// 类型/接口：PascalCase
interface User {}
type UserStatus = 'active' | 'inactive'

// 函数名：camelCase，动词开头
const getUserList = async () => {}
const handleSubmit = () => {}
const validateForm = () => {}

// 私有方法：下划线前缀
const _internalMethod = () => {}

// ❌ 不好的命名
const user_name = ref('')  // 应该用camelCase
const GetUserList = () => {}  // 应该用camelCase
const a = ref('')  // 没有意义的命名
const data = ref({})  // 命名太泛化
```

### 9.2 注释规范

```typescript
/**
 * 获取用户列表
 * @param params 查询参数
 * @returns 用户列表和总数
 */
export function getUserList(params: UserSearchParams): Promise<UserListResponse> {
  return request.get('/api/users', { params })
}

// ✅ 好的注释：解释为什么这样做
// 使用防抖避免频繁请求接口
const debouncedSearch = useDebouncedRef(searchKeyword.value, 500)

// ❌ 不好的注释：重复代码的意思
// 设置loading为true
loading.value = true
```

### 9.3 代码格式化

```javascript
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "none",
  "arrowParens": "always",
  "printWidth": 100,
  "tabWidth": 2
}
```

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'plugin:vue/vue3-recommended',
    '@vue/typescript/recommended',
    'prettier'
  ],
  rules: {
    'vue/multi-word-component-names': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
  }
}
```

---

## 10. 测试策略

### 10.1 单元测试

```typescript
// tests/unit/utils/format.spec.ts
import { describe, it, expect } from 'vitest'
import { formatDate, formatCurrency } from '@/utils/format'

describe('format utils', () => {
  describe('formatDate', () => {
    it('should format date correctly', () => {
      const date = new Date('2026-02-13T10:30:00')
      expect(formatDate(date)).toBe('2026-02-13')
    })

    it('should handle invalid date', () => {
      expect(formatDate(null)).toBe('-')
    })
  })

  describe('formatCurrency', () => {
    it('should format currency with commas', () => {
      expect(formatCurrency(1000)).toBe('¥1,000.00')
      expect(formatCurrency(1234567.89)).toBe('¥1,234,567.89')
    })

    it('should handle zero', () => {
      expect(formatCurrency(0)).toBe('¥0.00')
    })
  })
})
```

### 10.2 组件测试

```typescript
// tests/unit/components/UserCard.spec.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from '@/components/UserCard.vue'

describe('UserCard', () => {
  it('renders user info correctly', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: {
          id: 1,
          name: '张三',
          email: 'zhangsan@example.com'
        }
      }
    })

    expect(wrapper.text()).toContain('张三')
    expect(wrapper.text()).toContain('zhangsan@example.com')
  })

  it('emits delete event when delete button clicked', async () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: '张三' }
      }
    })

    await wrapper.find('.delete-btn').trigger('click')

    expect(wrapper.emitted('delete')).toBeTruthy()
    expect(wrapper.emitted('delete')?.[0]).toEqual([1])
  })
})
```

---

## 总结

### 开发检查清单

```yaml
项目结构:
  - [ ] 目录结构清晰合理
  - [ ] 组件按功能分类
  - [ ] 类型定义完整

组件设计:
  - [ ] 单一职责原则
  - [ ] Props和Emits类型定义
  - [ ] 样式使用scoped
  - [ ] 组件代码不超过300行

状态管理:
  - [ ] 全局状态用Pinia
  - [ ] 本地状态用ref/reactive
  - [ ] Store按模块划分

路由管理:
  - [ ] 路由懒加载
  - [ ] 使用路由守卫
  - [ ] 定义404页面

API调用:
  - [ ] 统一配置Axios
  - [ ] API按模块封装
  - [ ] 统一错误处理

错误处理:
  - [ ] 全局错误处理
  - [ ] 组件错误边界
  - [ ] 异步错误处理

性能优化:
  - [ ] 组件懒加载
  - [ ] 列表虚拟滚动
  - [ ] 计算属性缓存
  - [ ] 防抖节流

TypeScript:
  - [ ] 类型定义完整
  - [ ] 使用泛型
  - [ ] 避免any类型

代码规范:
  - [ ] 命名规范统一
  - [ ] 注释清晰有用
  - [ ] 代码格式化

测试:
  - [ ] 单元测试覆盖工具函数
  - [ ] 组件测试覆盖核心组件
```

---

**记住**：好的前端代码是类型安全、性能优良、易于维护的！
