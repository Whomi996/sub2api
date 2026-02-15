# Pinia 商店文档

该目录包含 Sub2API 前端应用程序的所有 Pinia 存储。

## 商店概览

### 1.授权商店 (`auth.ts`)

管理用户身份验证状态、登录/注销和令牌持久性。

**状态：**

- `user: User | null` - 当前经过身份验证的用户
- `token: string | null` - JWT 身份验证令牌

**计算：**

- `isAuthenticated: boolean` - 用户当前是否已通过身份验证

**行动：**

- `login(credentials)` - 使用用户名/密码验证用户
- `register(userData)` - 注册新用户帐户
- `logout()` - 清除身份验证和注销
- `checkAuth()` - 从本地存储恢复会话
- `refreshUser()` - 从服务器获取最新的用户数据

### 2. 应用商店 (`app.ts`)

管理全局 UI 状态，包括侧边栏、加载指示器和 toast 通知。

**状态：**

- `sidebarCollapsed: boolean` - 侧边栏折叠状态
- `loading: boolean` - 全局加载状态
- `toasts: Toast[]` - 活动 toast 通知

**计算：**

- `hasActiveToasts: boolean` - 是否有任何 toast 处于活动状态

**行动：**

- `toggleSidebar()` - 切换侧边栏状态
- `setSidebarCollapsed(collapsed)` - 显式设置侧边栏状态
- `setLoading(isLoading)` - 设置加载状态
- `showToast(type, message, duration?)` - 显示 toast 通知
- `showSuccess(message, duration?)` - 显示成功祝酒词
- `showError(message, duration?)` - 显示错误 toast
- `showInfo(message, duration?)` - 显示信息 toast
- `showWarning(message, duration?)` - 显示警告 toast
- `hideToast(id)` - 隐藏特定的 toast
- `clearAllToasts()` - 清除所有 toast
- `withLoading(operation)` - 在加载状态下执行异步操作
- `withLoadingAndError(operation, errorMessage?)` - 执行加载和错误处理
- `reset()` - 将存储重置为默认值

## 用法示例

### 授权商店

```typescript
import { useAuthStore } from '@/stores'

// In component setup
const authStore = useAuthStore()

// Initialize on app startup
authStore.checkAuth()

// Login
try {
  await authStore.login({ username: 'user', password: 'pass' })
  console.log('Logged in:', authStore.user)
} catch (error) {
  console.error('Login failed:', error)
}

// Check authentication
if (authStore.isAuthenticated) {
  console.log('User is logged in:', authStore.user?.username)
}

// Logout
authStore.logout()
```

### 应用商店

```typescript
import { useAppStore } from '@/stores'

// In component setup
const appStore = useAppStore()

// Sidebar control
appStore.toggleSidebar()
appStore.setSidebarCollapsed(true)

// Loading state
appStore.setLoading(true)
// ... do work
appStore.setLoading(false)

// Or use helper
await appStore.withLoading(async () => {
  const data = await fetchData()
  return data
})

// Toast notifications
appStore.showSuccess('Operation completed!')
appStore.showError('Something went wrong!', 5000)
appStore.showInfo('FYI: This is informational')
appStore.showWarning('Be careful!')

// Custom toast
const toastId = appStore.showToast('info', 'Custom message', undefined) // No auto-dismiss
// Later...
appStore.hideToast(toastId)
```

### Vue 组件中的组合使用

```vue
<script setup lang="ts">
import { useAuthStore, useAppStore } from '@/stores'
import { onMounted } from 'vue'

const authStore = useAuthStore()
const appStore = useAppStore()

onMounted(() => {
  // Check for existing session
  authStore.checkAuth()
})

async function handleLogin(username: string, password: string) {
  try {
    await appStore.withLoading(async () => {
      await authStore.login({ username, password })
    })
    appStore.showSuccess('Welcome back!')
  } catch (error) {
    appStore.showError('Login failed. Please check your credentials.')
  }
}

async function handleLogout() {
  authStore.logout()
  appStore.showInfo('You have been logged out.')
}
</script>

<template>
  <div>
    <button @click="appStore.toggleSidebar">Toggle Sidebar</button>

    <div v-if="appStore.loading">Loading...</div>

    <div v-if="authStore.isAuthenticated">
      Welcome, {{ authStore.user?.username }}!
      <button @click="handleLogout">Logout</button>
    </div>
    <div v-else>
      <button @click="handleLogin('user', 'pass')">Login</button>
    </div>
  </div>
</template>
```

## 坚持

- **Auth Store**：令牌和用户数据自动保存到 `localStorage`
- 键：`auth_token`、`auth_user`
- 在 `checkAuth()` 调用时恢复
- **App Store**：无持久性（UI 状态在页面重新加载时重置）

## TypeScript 支持

所有商店都完全使用 TypeScript 进行打字。从 `@/types` 导入类型：

```typescript
import type { User, Toast, ToastType } from '@/types'
```

## 测试

商店可以重置为初始状态：

```typescript
// Auth store
authStore.logout() // Clears all auth state

// App store
appStore.reset() // Resets to defaults
```
