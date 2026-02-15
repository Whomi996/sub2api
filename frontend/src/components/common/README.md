# 常用组件

该目录包含使用 Composition API、TypeScript 和 TailwindCSS 构建的可重用 Vue 3 组件。

## Components

### 数据表.vue

具有排序、加载状态和自定义单元格渲染功能的通用数据表组件。

**道具：**

- `columns: Column[]` - 具有键、标签、可排序和格式化程序的列定义数组
- `data: any[]` - 要显示的数据对象数组
- `loading?: boolean` - 显示加载骨架
- `defaultSortKey?: string` - 默认排序键（仅在没有持久排序状态时使用）
- `defaultSortOrder?: 'asc' | 'desc'` - 默认排序顺序（默认：`asc`）
- `sortStorageKey?: string` - 将排序状态（键+顺序）保留到 localStorage
- `rowKey?: string | (row: any) => string | number` - 行键字段或解析器（默认为 `row.id`，回退到索引）

**插槽：**

- `empty` - 自定义空状态内容
- `cell-{key}` - 特定列的自定义单元格渲染器（接收 `row` 和 `value`）

**用法：**

```vue
<DataTable
  :columns="[
    { key: 'name', label: 'Name', sortable: true },
    { key: 'email', label: 'Email' },
    { key: 'status', label: 'Status', formatter: (val) => val.toUpperCase() }
  ]"
  :data="users"
  :loading="isLoading"
>
  <template #cell-actions="{ row }">
    <button @click="editUser(row)">Edit</button>
  </template>
</DataTable>
```

---

### 分页.vue

带有页码、导航和页面大小选择器的分页组件。

**道具：**

- `total: number` - 项目总数
- `page: number` - 当前页面（1 索引）
- `pageSize: number` - 每页项目数
- `pageSizeOptions?: number[]` - 可用页面大小选项（默认值：[10, 20, 50, 100]）

**活动：**

- `update:page` - 页面更改时发出
- `update:pageSize` - 页面大小更改时发出

**用法：**

```vue
<Pagination
  :total="totalUsers"
  :page="currentPage"
  :pageSize="pageSize"
  @update:page="currentPage = $event"
  @update:pageSize="pageSize = $event"
/>
```

---

### 模态.vue

具有可定制大小和关闭行为的模态对话框。

**道具：**

- `show: boolean` - 控制模态可见性
- `title: string` - 模态标题
- `size?: 'sm' | 'md' | 'lg' | 'xl' | 'full'` - 模态尺寸（默认值：'md'）
- `closeOnEscape?: boolean` - 按 Escape 键关闭（默认值：true）
- `closeOnClickOutside?: boolean` - 在背景单击时关闭（默认值：true）

**活动：**

- `close` - 当模态应该关闭时发出

**插槽：**

- `default` - 模态主体内容
- `footer` - 模态页脚内容

**用法：**

```vue
<Modal :show="showModal" title="Edit User" size="lg" @close="showModal = false">
  <form @submit.prevent="saveUser">
    <!-- Form content -->
  </form>

  <template #footer>
    <button @click="showModal = false">Cancel</button>
    <button @click="saveUser">Save</button>
  </template>
</Modal>
```

---

### 确认对话框.vue

确认对话框构建在模态组件之上。

**道具：**

- `show: boolean` - 控制对话框可见性
- `title: string` - 对话框标题
- `message: string` - 确认消息
- `confirmText?: string` - 确认按钮文本（默认值：“确认”）
- `cancelText?: string` - 取消按钮文本（默认值：“取消”）
- `danger?: boolean` - 使用危险/红色样式（默认值：false）

**活动：**

- `confirm` - 用户确认时发出
- `cancel` - 用户取消时发出

**用法：**

```vue
<ConfirmDialog
  :show="showDeleteConfirm"
  title="Delete User"
  message="Are you sure you want to delete this user? This action cannot be undone."
  confirm-text="Delete"
  cancel-text="Cancel"
  danger
  @confirm="deleteUser"
  @cancel="showDeleteConfirm = false"
/>
```

---

### StatCard.vue

统计卡组件，用于显示带有可选更改指示器的指标。

**道具：**

- `title: string` - 卡标题
- `value: number | string` - 要显示的主要值
- `icon?: Component` - 图标组件
- `change?: number` - 百分比变化值
- `changeType?: 'up' | 'down' | 'neutral'` - 改变方向（默认值：“中性”）
- `formatValue?: (value) => string` - 自定义值格式化程序

**用法：**

```vue
<StatCard title="Total Users" :value="1234" :icon="UserIcon" :change="12.5" change-type="up" />
```

---

### Toast.vue

Toast 通知组件，自动显示来自应用商店的 Toast。

**用法：**

```vue
<!-- Add once in App.vue or layout -->
<Toast />
```

```typescript
// Trigger toasts from anywhere using the app store
import { useAppStore } from '@/stores/app'

const appStore = useAppStore()

appStore.addToast({
  type: 'success',
  title: 'Success!',
  message: 'User created successfully',
  duration: 3000
})

appStore.addToast({
  type: 'error',
  message: 'Failed to delete user'
})
```

---

### LoadingSpinner.vue

简单的动画加载旋转器。

**道具：**

- `size?: 'sm' | 'md' | 'lg' | 'xl'` - 微调器大小（默认值：'md'）
- `color?: 'primary' | 'secondary' | 'white' | 'gray'` - 微调器颜色（默认值：“主要”）

**用法：**

```vue
<LoadingSpinner size="lg" color="primary" />
```

---

### 空状态.vue

带有图标、消息和可选操作按钮的空状态占位符。

**道具：**

- `icon?: Component` - 图标组件
- `title: string` - 空状态标题
- `description: string` - 空状态描述
- `actionText?: string` - 操作按钮文本
- `actionTo?: string | object` - 路由器链接目的地
- `actionIcon?: boolean` - 在按钮中显示加号图标（默认值：true）

**插槽：**

- `icon` - 自定义图标内容
- `action` - 自定义操作按钮/链接

**用法：**

```vue
<EmptyState
  title="No users found"
  description="Get started by creating your first user account."
  action-text="Add User"
  :action-to="{ name: 'users-create' }"
/>
```

## Import

您可以单独导入组件：

```typescript
import { DataTable, Pagination, Modal } from '@/components/common'
```

或者导入特定组件：

```typescript
import DataTable from '@/components/common/DataTable.vue'
```

## Features

所有组件包括：

- **TypeScript 支持** 具有正确的类型定义
- **辅助功能**，具有 ARIA 属性和键盘导航
- **响应式设计**，具有适合移动设备的布局
- **TailwindCSS 样式**，实现一致的设计
- **Vue 3 组合 API** 与 `<script setup>`
- **插槽支持**用于定制
