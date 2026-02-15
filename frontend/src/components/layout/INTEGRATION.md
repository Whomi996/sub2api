# 布局组件集成指南

## 快速入门

### 1.导入布局组件

```typescript
// In your view files
import { AppLayout, AuthLayout } from '@/components/layout'
```

### 2. 在路由中使用

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

// Views
import DashboardView from '@/views/DashboardView.vue'
import LoginView from '@/views/auth/LoginView.vue'
import RegisterView from '@/views/auth/RegisterView.vue'

const routes: RouteRecordRaw[] = [
  // Auth routes (no layout needed - views use AuthLayout internally)
  {
    path: '/login',
    name: 'Login',
    component: LoginView,
    meta: { requiresAuth: false }
  },
  {
    path: '/register',
    name: 'Register',
    component: RegisterView,
    meta: { requiresAuth: false }
  },

  // User routes (use AppLayout)
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: DashboardView,
    meta: { requiresAuth: true, title: 'Dashboard' }
  },
  {
    path: '/api-keys',
    name: 'ApiKeys',
    component: () => import('@/views/ApiKeysView.vue'),
    meta: { requiresAuth: true, title: 'API Keys' }
  },
  {
    path: '/usage',
    name: 'Usage',
    component: () => import('@/views/UsageView.vue'),
    meta: { requiresAuth: true, title: 'Usage Statistics' }
  },
  {
    path: '/redeem',
    name: 'Redeem',
    component: () => import('@/views/RedeemView.vue'),
    meta: { requiresAuth: true, title: 'Redeem Code' }
  },
  {
    path: '/profile',
    name: 'Profile',
    component: () => import('@/views/ProfileView.vue'),
    meta: { requiresAuth: true, title: 'Profile Settings' }
  },

  // Admin routes (use AppLayout, admin only)
  {
    path: '/admin/dashboard',
    name: 'AdminDashboard',
    component: () => import('@/views/admin/DashboardView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Admin Dashboard' }
  },
  {
    path: '/admin/users',
    name: 'AdminUsers',
    component: () => import('@/views/admin/UsersView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'User Management' }
  },
  {
    path: '/admin/groups',
    name: 'AdminGroups',
    component: () => import('@/views/admin/GroupsView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Groups' }
  },
  {
    path: '/admin/accounts',
    name: 'AdminAccounts',
    component: () => import('@/views/admin/AccountsView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Accounts' }
  },
  {
    path: '/admin/proxies',
    name: 'AdminProxies',
    component: () => import('@/views/admin/ProxiesView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Proxies' }
  },
  {
    path: '/admin/redeem-codes',
    name: 'AdminRedeemCodes',
    component: () => import('@/views/admin/RedeemCodesView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true, title: 'Redeem Codes' }
  },

  // Default redirect
  {
    path: '/',
    redirect: '/dashboard'
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// Navigation guards
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    // Redirect to login if not authenticated
    next('/login')
  } else if (to.meta.requiresAdmin && !authStore.isAdmin) {
    // Redirect to dashboard if not admin
    next('/dashboard')
  } else {
    next()
  }
})

export default router
```

### 3. 在 main.ts 中初始化 Stores

```typescript
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './style.css'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.use(router)

// Initialize auth state on app startup
import { useAuthStore } from '@/stores'
const authStore = useAuthStore()
authStore.checkAuth()

app.mount('#app')
```

### 4.更新App.vue

```vue
<!-- src/App.vue -->
<template>
  <router-view />
</template>

<script setup lang="ts">
// App.vue just renders the router view
// Layouts are handled by individual views
</script>
```

---

## 查看组件模板

### 经过身份验证的页面模板

```vue
<!-- src/views/DashboardView.vue -->
<template>
  <AppLayout>
    <div class="space-y-6">
      <h1 class="text-3xl font-bold text-gray-900">Dashboard</h1>

      <!-- Your content here -->
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { AppLayout } from '@/components/layout'

// Your component logic here
</script>
```

### 身份验证页面模板

```vue
<!-- src/views/auth/LoginView.vue -->
<template>
  <AuthLayout>
    <h2 class="mb-6 text-2xl font-bold text-gray-900">Login</h2>

    <!-- Your login form here -->

    <template #footer>
      <p class="text-gray-600">
        Don't have an account?
        <router-link to="/register" class="text-indigo-600 hover:underline"> Sign up </router-link>
      </p>
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
import { AuthLayout } from '@/components/layout'

// Your login logic here
</script>
```

---

## 定制

### 改变颜色

默认情况下，这些组件使用 Tailwind 的靛蓝配色方案。更改：

```vue
<!-- Change all instances of indigo-* to your preferred color -->
<div class="bg-blue-600">   <!-- Instead of bg-indigo-600 -->
<div class="text-blue-600">  <!-- Instead of text-indigo-600 -->
```

### 添加自定义图标

将 HTML 实体图标替换为您喜欢的图标库：

```vue
<!-- Before (HTML entities) -->
<span class="text-lg">&#128200;</span>

<!-- After (Heroicons example) -->
<ChartBarIcon class="h-5 w-5" />
```

### 侧边栏定制

修改`AppSidebar.vue`中的导航项：

```typescript
// Add/remove/modify navigation items
const userNavItems = [
  { path: '/dashboard', label: 'Dashboard', icon: '&#128200;' },
  { path: '/new-page', label: 'New Page', icon: '&#128196;' } // Add new item
  // ...
]
```

### 标题定制

修改 `AppHeader.vue` 中的用户下拉列表：

```vue
<!-- Add new dropdown items -->
<router-link
  to="/settings"
  @click="closeDropdown"
  class="flex items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
>
  <span class="mr-2">&#9881;</span>
  Settings
</router-link>
```

---

## 移动响应行为

### 侧边栏

- **桌面 (md+)**：始终可见，可以折叠为仅图标视图
- **移动**：默认隐藏，通过标题中的菜单切换显示

### 标题

- **桌面**：显示完整的用户信息和余额
- **移动**：显示带有汉堡菜单的紧凑视图

为了改善移动体验，您可以添加叠加和过渡：

```vue
<!-- AppSidebar.vue enhancement for mobile -->
<aside
  class="fixed left-0 top-0 z-40 h-screen transition-transform duration-300"
  :class="[
    sidebarCollapsed ? 'w-16' : 'w-64',
    // Hide on mobile when collapsed
    'md:translate-x-0',
    sidebarCollapsed ? '-translate-x-full md:translate-x-0' : 'translate-x-0'
  ]"
>
  <!-- ... -->
</aside>

<!-- Add overlay for mobile -->
<div
  v-if="!sidebarCollapsed"
  @click="toggleSidebar"
  class="fixed inset-0 z-30 bg-black bg-opacity-50 md:hidden"
></div>
```

---

## 状态管理集成

### 授权商店使用

```typescript
import { useAuthStore } from '@/stores'

const authStore = useAuthStore()

// Check if user is authenticated
if (authStore.isAuthenticated) {
  // User is logged in
}

// Check if user is admin
if (authStore.isAdmin) {
  // User has admin role
}

// Get current user
const user = authStore.user
```

### 应用商店使用情况

```typescript
import { useAppStore } from '@/stores'

const appStore = useAppStore()

// Toggle sidebar
appStore.toggleSidebar()

// Show notifications
appStore.showSuccess('Operation completed!')
appStore.showError('Something went wrong')
appStore.showInfo('Did you know...')
appStore.showWarning('Be careful!')

// Loading state
appStore.setLoading(true)
// ... perform operation
appStore.setLoading(false)

// Or use helper
await appStore.withLoading(async () => {
  // Your async operation
})
```

---

## 辅助功能

所有布局组件包括：

- **语义 HTML**：正确使用 `<nav>`、`<header>`、`<main>`、`<aside>`
- **ARIA 标签**：按钮具有描述性标签
- **键盘导航**：所有交互元素均可通过键盘访问
- **焦点管理**：使用 Tailwind 的 `focus:` 实用程序实现正确的焦点状态
- **颜色对比度**：符合 WCAG AA 标准的颜色组合

进一步加强：

```vue
<!-- Add skip to main content link -->
<a
  href="#main-content"
  class="sr-only rounded bg-white px-4 py-2 focus:not-sr-only focus:absolute focus:left-4 focus:top-4"
>
  Skip to main content
</a>

<main id="main-content">
  <!-- Content -->
</main>
```

---

## 测试

### 单元测试布局组件

```typescript
// AppHeader.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import AppHeader from '@/components/layout/AppHeader.vue'

describe('AppHeader', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('renders user info when authenticated', () => {
    const wrapper = mount(AppHeader)
    // Add assertions
  })

  it('shows dropdown when clicked', async () => {
    const wrapper = mount(AppHeader)
    await wrapper.find('button').trigger('click')
    expect(wrapper.find('.dropdown').exists()).toBe(true)
  })
})
```

---

## 性能优化

### 延迟加载

在上面的路由器示例中，使用布局的视图已经被延迟加载。

### 代码分割

布局组件在导入时会自动进行代码分割：

```typescript
// This creates a separate chunk for layout components
import { AppLayout } from '@/components/layout'
```

### 减少重新渲染

布局组件使用 `computed` 引用来防止不必要的重新渲染：

```typescript
const sidebarCollapsed = computed(() => appStore.sidebarCollapsed)
// This only re-renders when sidebarCollapsed changes
```

---

## 故障排除

### 侧边栏未显示

- 检查`useAppStore`是否正确初始化
- 验证 Tailwind 类是否正在处理
- 检查 z-index 与其他组件的冲突

### 路线未在侧边栏中突出显示

- 确保路线路径完全匹配
- 检查`isActive()`函数逻辑
- 验证 `useRoute()` 是否正常工作

### 用户信息不显示

- 确保使用 `checkAuth()` 初始化身份验证存储
- 验证用户是否已登录
- 检查 localStorage 的身份验证数据

### 移动菜单不起作用

- 验证 `toggleSidebar()` 是否被正确调用
- 检查响应断点 (md:)
- 在实际移动设备或浏览器开发工具上进行测试
