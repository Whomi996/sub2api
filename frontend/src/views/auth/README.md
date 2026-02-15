# 身份验证视图

该目录包含 Sub2API 前端应用程序的 Vue 3 身份验证视图。

## Components

### LoginView.vue

现有用户进行身份验证的登录页面。

**特征：**

- 用户名和密码输入并进行验证
- 持久会话的“记住我”复选框
- 表单验证与实时错误显示
- 身份验证期间的加载状态
- 登录尝试失败时显示错误消息
- 成功登录后重定向至仪表板
- 新用户注册页面链接

**用法：**

```vue
<template>
  <LoginView />
</template>

<script setup lang="ts">
import { LoginView } from '@/views/auth'
</script>
```

**路线：**

- 路径：`/login`
- 姓名：`Login`
- 元：`{ requiresAuth: false }`

**验证规则：**

- 用户名：必填，最少 3 个字符
- 密码：必填，至少 6 个字符

**行为：**

- 使用凭证调用 `authStore.login()`
- 成功登录时显示成功提示
- 显示错误 toast 和失败时的内联错误消息
- 从查询参数重定向到 `/dashboard` 或预期路线
- 将经过身份验证的用户重定向到登录页面

### RegisterView.vue

新用户创建帐户的注册页面。

**特征：**

- 用户名、电子邮件、密码和确认密码输入
- 全面的表单验证
- 密码强度要求（8+字符、字母+数字）
- 使用正则表达式验证电子邮件格式
- 密码匹配验证
- 注册期间的加载状态
- 注册失败时显示错误消息
- 成功时重定向至仪表板
- 链接到现有用户的登录页面

**用法：**

```vue
<template>
  <RegisterView />
</template>

<script setup lang="ts">
import { RegisterView } from '@/views/auth'
</script>
```

**路线：**

- 路径：`/register`
- 姓名：`Register`
- 元：`{ requiresAuth: false }`

**验证规则：**

- 用户名：
- 必需的
- 3-50 个字符
- 仅限字母、数字、下划线和连字符
- 电子邮件：
- 必需的
- 有效的电子邮件格式（RFC 5322 正则表达式）
- 密码：
- 必需的
- 至少 8 个字符
- 必须包含至少一个字母和一个数字
- 确认密码：
- 必需的
- 必须匹配密码

**行为：**

- 使用用户数据调用 `authStore.register()`
- 注册成功时显示成功提示
- 显示错误 toast 和失败时的内联错误消息
- 注册成功后重定向至`/dashboard`
- 将经过身份验证的用户重定向到注册页面

## Architecture

### 组件结构

两种视图都遵循一致的结构：

```
<template>
  <AuthLayout>
    <div class="space-y-6">
      <!-- Title -->
      <!-- Form -->
      <!-- Error Message -->
      <!-- Submit Button -->
    </div>

    <template #footer>
      <!-- Footer Links -->
    </template>
  </AuthLayout>
</template>

<script setup lang="ts">
// Imports
// State
// Validation
// Form Handlers
</script>
```

### 状态管理

两种视图都使用：

- `useAuthStore()` - 用于身份验证操作（登录、注册）
- `useAppStore()` - 用于 Toast 通知和 UI 反馈
- `useRouter()` - 用于导航和重定向

### 验证策略

**客户端验证：**

- 表单提交的实时验证
- 字段级错误消息
- 全面的验证规则
- TypeScript 类型安全

**服务器端验证：**

- 后端API验证所有输入
- 错误响应得到妥善处理
- 显示用户友好的错误消息

### 造型

**设计系统：**

- TailwindCSS 实用程序类
- 一致的配色方案（靛蓝原色）
- 响应式设计
- 可访问的表单控件
- 使用旋转动画加载状态

**视觉反馈：**

- 无效字段上有红色边框
- 输入下方的错误消息
- API 错误的全局错误横幅
- 完成后祝酒成功
- 在提交按钮上加载微调器

## 依赖关系

### Components

- `AuthLayout` - `@/components/layout` 中的身份验证页面的布局包装器

### 商店

- `authStore` - 来自 `@/stores/auth` 的身份验证状态管理
- `appStore` - 应用程序状态和来自 `@/stores/app` 的 toast

### 图书馆

-Vue 3 组合 API
- 用于导航的 Vue 路由器
- Pinia 用于状态管理
- 用于类型安全的 TypeScript

## 用法示例

### 基本登录流程

```typescript
// User enters credentials
formData.username = 'john_doe'
formData.password = 'SecurePass123'

// Submit form
await handleLogin()

// On success:
// - authStore.login() called
// - Token and user stored
// - Success toast shown
// - Redirected to /dashboard

// On error:
// - Error message displayed
// - Error toast shown
// - Form remains editable
```

### 基本注册流程

```typescript
// User enters registration data
formData.username = 'jane_smith'
formData.email = 'jane@example.com'
formData.password = 'SecurePass123'
formData.confirmPassword = 'SecurePass123'

// Submit form
await handleRegister()

// On success:
// - authStore.register() called
// - Token and user stored
// - Success toast shown
// - Redirected to /dashboard

// On error:
// - Error message displayed
// - Error toast shown
// - Form remains editable
```

## 错误处理

### 客户端错误

```typescript
// Validation errors
errors.username = 'Username must be at least 3 characters'
errors.email = 'Please enter a valid email address'
errors.password = 'Password must be at least 8 characters with letters and numbers'
errors.confirmPassword = 'Passwords do not match'
```

### 服务器端错误

```typescript
// API error responses
{
  response: {
    data: {
      detail: 'Username already exists'
    }
  }
}

// Displayed as:
errorMessage.value = 'Username already exists'
appStore.showError('Username already exists')
```

## 辅助功能

- 语义 HTML 元素（`<label>`、`<input>`、`<button>`）
- 标签上正确的 `for` 属性
- 加载状态的 ARIA 属性
- 键盘导航支持
- 焦点管理
- 错误公告
- 足够的色彩对比度

## 测试注意事项

### 单元测试

- 表单验证逻辑
- 错误处理
- 状态管理
- 路由器导航

### 集成测试

- 完整的登录流程
- 完整的注册流程
- 错误场景
- 重定向行为

### E2E 测试

- 完整的用户旅程
- 表单交互
- API集成
- 成功/错误状态

## 未来的增强

潜在的改进：

- OAuth/SSO 集成（Google、GitHub）
- 双因素身份验证 (2FA)
- 密码强度计
- 电子邮件验证流程
- 忘记密码功能
- 社交登录按钮
- 验证码集成
- 会话超时警告
- 密码可见性切换
- 自动填充支持增强

## 安全考虑

- 密码永远不会被记录或显示
- 生产中需要 HTTPS
- JWT 令牌安全存储在 localStorage 中
- API 上的 CORS 保护
- Vue 自动转义的 XSS 保护
- 基于令牌的身份验证的 CSRF 保护
- 后端API速率限制
- 输入净化
- 安全密码要求

## Performance

- 延迟加载的路线
- 最小捆绑尺寸
- 快速初始渲染
- 使用反应性参考优化重新渲染
- 没有不必要的观察者
- 高效的表单验证

## 浏览器支持

- 现代浏览器（Chrome、Firefox、Safari、Edge）
- 需要 ES2015+
- Flexbox 和 CSS 网格
- Tailwind CSS 实用程序
-Vue 3 运行时

## 相关文档

- [Auth Store Documentation](/src/stores/README.md#auth-store)
- [AuthLayout Component](/src/components/layout/README.md#authlayout)
- [Router Configuration](/src/router/index.ts)
- [API Documentation](/src/api/README.md#authentication)
- [Type Definitions](/src/types/index.ts)
