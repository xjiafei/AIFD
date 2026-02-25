# 前端编码规范 (Vue.js + TypeScript)

## 文件命名

- **组件**: PascalCase (例: UserProfile.vue, LoginForm.vue)
- **页面**: PascalCase (例: HomePage.vue, UserManagement.vue)
- **工具函数**: camelCase (例: formatDate.ts, validateEmail.ts)
- **Store模块**: camelCase (例: userStore.ts, authStore.ts)

## 组件结构

### 单文件组件 (SFC) 顺序
```vue
<template>
  <!-- 模板 -->
</template>

<script setup lang="ts">
// 导入
import { ref, computed, onMounted } from 'vue'

// Props定义
interface Props {
  userId: number
  name?: string
}
const props = defineProps<Props>()

// Emits定义
const emit = defineEmits<{
  update: [id: number]
  delete: [id: number]
}>()

// 响应式数据
const count = ref(0)

// 计算属性
const doubleCount = computed(() => count.value * 2)

// 方法
const handleClick = () => {
  emit('update', props.userId)
}

// 生命周期
onMounted(() => {
  // 初始化逻辑
})
</script>

<style scoped>
/* 样式 */
</style>
```

## TypeScript类型定义

```typescript
// API响应类型
interface UserResponse {
  id: number
  name: string
  email: string
  createdAt: string
}

// 表单类型
interface LoginForm {
  email: string
  password: string
}

// Store状态类型
interface UserState {
  currentUser: UserResponse | null
  isAuthenticated: boolean
}
```

## 最佳实践

1. 优先使用组合式API (Composition API)
2. 使用TypeScript严格模式
3. Props和Emits必须定义类型
4. 使用<script setup>语法糖
5. 组件名使用多个单词（避免与HTML元素冲突）
6. 使用v-for时必须绑定key
7. 避免在模板中使用复杂表达式，使用计算属性
8. API调用放在独立的service文件中
9. 使用CSS scoped避免样式污染
10. 合理使用Pinia进行状态管理
