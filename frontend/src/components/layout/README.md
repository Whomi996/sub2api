# 布局组件

用于 Sub2API 前端的 Vue 3 布局组件，使用 Composition API、TypeScript 和 TailwindCSS 构建。

## Components

### 1.AppLayout.vue

带有侧边栏和标题的主应用程序布局。

**用法：**

```vue
<template>
  <AppLayout>
    <!-- Your page content here -->
    <h1>Dashboard</h1>
    <p>Welcome to your dashboard!</p>
  </AppLayout>
</template>

<script setup lang="ts">
import { AppLayout } from '@/components/layout'
</script>
```

**特征：**

- 响应式侧边栏（可折叠）
- 修复了顶部的标题
- 带插槽的主要内容区域
- 根据侧边栏状态自动调整边距

---

### 2.AppSidebar.vue

带有用户和管理部分的导航侧边栏。

**特征：**

- 顶部徽标/品牌
- 用户导航链接：
- 仪表板
- API 密钥
- 用法
- 赎回
- 轮廓
- 管理员导航链接（仅当用户是管理员时才显示）：
- 管理仪表板
- 用户
- 团体
- 账户
- 代理
- 兑换代码
- 带切换按钮的可折叠侧边栏
- 活动路线突出显示
- 使用 HTML 实体的图标
- 响应式（适合移动设备）

**由 AppLayout 自动使用** - 无需单独导入。

---

### 3.AppHeader.vue

包含用户信息和操作的顶部标题。

**特征：**

- 移动菜单切换按钮
- 页面标题（来自路由元或槽）
- 用户余额显示（仅限桌面）
- 用户下拉菜单：
- 个人资料链接
- 注销按钮
- 带有姓名缩写的用户头像
- 点击外部下拉菜单处理
- 响应式设计

**与自定义标题一起使用：**

```vue
<template>
  <AppLayout>
    <template #title> Custom Page Title </template>

    <!-- Your content -->
  </AppLayout>
</template>
```

**由 AppLayout 自动使用** - 无需单独导入。

---

### 4.AuthLayout.vue

身份验证页面（登录/注册）的简单居中布局。

**用法：**

```vue
<template>
  <AuthLayout>
    <!-- Login/Register form content -->
    <h2 class="mb-6 text-2xl font-bold">Login</h2>

    <form @submit.prevent="handleLogin">
      <!-- Form fields -->
    </form>

    <!-- Optional footer slot -->
    <template #footer>
      <p>
        Don't have an account?
        <router-link to="/register" class="text-indigo-600 hover:underline"> Sign up </router-link>
      </p>
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
import { AuthLayout } from '@/components/layout'

function handleLogin() {
  // Login logic
}
</script>
```

**特征：**

- 居中的卡片容器
- 渐变背景
- 顶部徽标/品牌
- 主要内容槽
- 可选的链接页脚插槽
- 完全响应

---

## 路由配置

要在标题中设置页面标题，请将元添加到您的路线中：

```typescript
// router/index.ts
const routes = [
  {
    path: '/dashboard',
    component: DashboardView,
    meta: { title: 'Dashboard' }
  },
  {
    path: '/api-keys',
    component: ApiKeysView,
    meta: { title: 'API Keys' }
  }
  // ...
]
```

---

## 存储依赖项

这些组件使用以下 Pinia 存储：

- **useAuthStore**：用于用户身份验证状态、角色检查和注销
- **useAppStore**：用于侧边栏状态管理和 toast 通知

确保这些商店在您的应用程序中正确初始化。

---

## 造型

所有组件都使用 TailwindCSS 实用程序类。确保您的 `tailwind.config.js` 包含组件路径：

```js
module.exports = {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}']
  // ...
}
```

---

## 图标

为简单起见，组件使用 HTML 实体图标：

- &#128200;图表（仪表板）
- &#128273;密钥（API 密钥）
- &#128202;条形图（使用情况）
- &#127873;礼物（兑换）
- &#128100;用户（个人资料）
- &#128268;行政
- &#128101;用户
- &#128193;文件夹（组）
- &#127760;环球（账户）
- &#128260;网络（代理）
- &#127991;门票（兑换代码）

如果需要，您可以将它们替换为您喜欢的图标库（例如 Heroicons、Font Awesome）。

---

## 移动响应能力

所有组件均完全响应：

- **AppSidebar**：固定在桌面上的位置，在移动设备上默认隐藏
- **AppHeader**：在小屏幕上显示移动菜单切换，隐藏余额显示
- **AuthLayout**：调整移动设备的填充和卡片大小

侧边栏使用 Tailwind 的响应断点 (md:) 来调整行为。
