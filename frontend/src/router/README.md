# Vue 路由器配置

## Overview

该目录包含 Sub2API 前端应用程序的 Vue Router 配置。该路由器实现了一个全面的导航系统，具有身份验证防护、基于角色的访问控制和延迟加载。

## 文件

- **index.ts**：带有路由定义和导航防护的主路由器配置
- **meta.d.ts**：路由元字段的 TypeScript 类型定义

## 路由结构

### 公共路线（无需身份验证）

|路径|组件|描述 |
| ----------- | ------------ | ---------------------- |
| `/login` |登录查看 |用户登录页面 |
| `/register` |注册查看 |用户注册页面 |

### 用户路由（需要身份验证）

|路径|组件|描述 |
| ------------ | ------------- | ---------------------------- |
| `/` | - |重定向至 `/dashboard` |
| `/dashboard` |仪表板视图 |带有统计信息的用户仪表板 |
| `/keys` |按键视图 | API 密钥管理 |
| `/usage` |使用情况查看 |使用记录及统计 |
| `/redeem` |兑换查看 |兑换码界面 |
| `/profile` |简介查看 |用户个人资料设置 |

### 管理路由（需要管理员角色）

|路径|组件|描述 |
| ------------------ | ------------------ | ------------------------------- |
| `/admin` | - |重定向至 `/admin/dashboard` |
| `/admin/dashboard` |管理仪表板视图 |管理仪表板 |
| `/admin/users` |管理员用户查看|用户管理|
| `/admin/groups` |管理组查看 |集团管理|
| `/admin/accounts` |管理帐户视图 |账户管理 |
| `/admin/proxies` | AdminProxies查看|代理管理|
| `/admin/redeem` |管理员兑换查看 |兑换码管理 |

### 特别路线

|路径|组件|描述 |
| ----------------- | ------------ | -------------- |
| `/:pathMatch(.*)` |未找到查看 | 404错误页面|

## 导航卫士

### 身份验证防护（beforeEach）

路由器实现了全面的导航防护：

1. **设置页面标题**：根据路由元更新文档标题
2. **检查身份验证**：
- 无需登录即可访问公共路线（`requiresAuth: false`）
- 受保护的路由需要身份验证
- 如果未通过身份验证，则重定向到 `/login`
3. **防止双重登录**：
- 将经过身份验证的用户重定向到登录/注册页面
4. **基于角色的访问控制**：
- 管理路由 (`requiresAdmin: true`) 需要管理员角色
- 非管理员用户被重定向到 `/dashboard`
5. **保留预定目的地**：
- 在查询参数中保存原始 URL 以用于登录后重定向

### 流程图

```
User navigates to route
        ↓
Set page title from meta
        ↓
Is route public? ──Yes──→ Already authenticated? ──Yes──→ Redirect to /dashboard
        ↓ No                                        ↓ No
        ↓                                      Allow access
        ↓
Is user authenticated? ──No──→ Redirect to /login with redirect query
        ↓ Yes
        ↓
Requires admin role? ──Yes──→ Is user admin? ──No──→ Redirect to /dashboard
        ↓ No                                  ↓ Yes
        ↓                                     ↓
Allow access ←────────────────────────────────┘
```

## 路由元字段

每个路由可以定义以下元字段：

```typescript
interface RouteMeta {
  requiresAuth?: boolean // Default: true (requires authentication)
  requiresAdmin?: boolean // Default: false (admin access only)
  title?: string // Page title
  breadcrumbs?: Array<{
    // Breadcrumb navigation
    label: string
    to?: string
  }>
  icon?: string // Icon for navigation menu
  hideInMenu?: boolean // Hide from navigation menu
}
```

## 延迟加载

所有路由组件都使用动态导入进行代码分割：

```typescript
component: () => import('@/views/user/DashboardView.vue')
```

好处：

- 减少初始包大小
- 更快的初始页面加载
- 按需加载组件
- Vite自动代码分割

## 身份验证存储集成

路由器与 Pinia auth store 集成 (`@/stores/auth`)：

```typescript
const authStore = useAuthStore()

// Check authentication status
authStore.isAuthenticated

// Check admin role
authStore.isAdmin
```

## 用法示例

### 程序化导航

```typescript
import { useRouter } from 'vue-router'

const router = useRouter()

// Navigate to a route
router.push('/dashboard')

// Navigate with query parameters
router.push({
  path: '/usage',
  query: { filter: 'today' }
})

// Navigate to admin route (will be blocked if not admin)
router.push('/admin/users')
```

### 路线链接

```vue
<template>
  <!-- Simple link -->
  <router-link to="/dashboard">Dashboard</router-link>

  <!-- Named route -->
  <router-link :to="{ name: 'Keys' }">API Keys</router-link>

  <!-- With query parameters -->
  <router-link :to="{ path: '/usage', query: { page: 1 } }"> Usage </router-link>
</template>
```

### 检查当前路由

```typescript
import { useRoute } from 'vue-router'

const route = useRoute()

// Check if on admin page
const isAdminPage = route.path.startsWith('/admin')

// Get route meta
const requiresAdmin = route.meta.requiresAdmin
```

## 滚动行为

路由器实现自动滚动管理：

- **浏览器导航**：恢复保存的滚动位置
- **新路线**：滚动到页面顶部
- **散列链接**：滚动到锚点（实施时）

## 错误处理

路由器包括导航失败的错误处理：

```typescript
router.onError((error) => {
  console.error('Router error:', error)
})
```

## 测试路线

要测试导航防护和路线访问：

1. **公共路由访问**：无需认证即可访问`/login`
2. **受保护的路由**：尝试在不登录的情况下访问 `/dashboard` （应重定向）
3. **管理员访问**：以普通用户身份登录，尝试 `/admin/users` （应重定向到仪表板）
4. **管理员成功**：以管理员身份登录，访问`/admin/users`（应该成功）
5. **404处理**：访问不存在的路由（应显示404页面）

## 开发技巧

### 添加新路线

1. 在`routes`数组中添加路由定义
2.创建对应的视图组件
3. 设置适当的元字段（`requiresAuth`、`requiresAdmin`）
4. 使用 `() => import()` 进行延迟加载
5. 使用路线文档更新此自述文件

### 调试导航

启用 Vue Router 调试模式：

```typescript
// In browser console
window.__VUE_ROUTER__ = router

// Check current route
router.currentRoute.value
```

### 常见问题

**问题**：页面刷新 404

- **原因**：服务器未配置 SPA
- **解决方案**：配置服务器为所有路由提供 `index.html` 服务

**问题**：导航守卫运行两次

- **原因**：多次 `next()` 调用
- **解决方案**：确保每个代码路径仅调用一次 `next()`

**问题**：用户数据未加载

- **原因**：身份验证存储未初始化
- **解决方案**：在App.vue或main.ts中调用`authStore.checkAuth()`

## 安全考虑

1. **Client-Side Only**：导航守卫是客户端；服务器还必须验证
2. **令牌验证**：API 应在每个请求上验证 JWT 令牌
3. **角色检查**：后端必须验证管理员角色，而不仅仅是前端
4. **XSS防护**：Vue自动转义模板内容
5. **CSRF保护**：使用CSRF令牌进行状态更改操作

## 性能优化

1. **延迟加载**：所有路由都使用动态导入
2. **代码分割**：Vite自动分割路由块
3. **预取**：考虑为公共路径添加路由预取
4. **路由缓存**：Vue Router 缓存组件实例

## 未来的增强

- [ ] 添加面包屑导航系统
- [ ] 实现管理员/用户之外的基于路由的权限
- [ ] 添加路线过渡动画
- [ ] 为预期导航实现路线预取
- [ ] 添加导航分析跟踪
