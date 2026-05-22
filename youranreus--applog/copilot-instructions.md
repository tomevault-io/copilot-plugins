## frontend

> - **框架**: Vue 3 + Composition API

# Frontend (Vue3) 专用规则

## 技术栈
- **框架**: Vue 3 + Composition API
- **UI 组件库**: Konsta UI Vue
- **样式**: Tailwind CSS
- **状态管理**: Pinia
- **路由**: Vue Router
- **语言**: TypeScript (严格模式)
- **包管理**: pnpm workspace

## 前端项目结构（必读）

前端项目采用模块化组织方式，严格遵循分层开发理念。项目结构如下：

```
packages/frontend/src/
├── assets/        # @assets - 存放静态资源（CSS、图片、字体等）
├── components/    # @components - 存放公共组件
│   └── ui/        # @components/ui - Shadcn-vue 基础组件
├── constants/     # @constants - 存放常量
├── hooks/         # @hooks - 存放公共 hooks（无业务属性）
├── pages/         # @pages - 存放页面视图
├── router/        # @router - 存放路由信息
├── stores/        # @stores - 存放 store 与业务逻辑
├── types/         # @types - 存放类型定义
└── utils/         # @utils - 存放工具函数
```

### 目录说明

**@assets** - 静态资源目录
- 存放项目静态资源文件
- 包括：CSS 样式文件、图片、字体、图标等
- 文件组织建议：
  - `css/` - 样式文件（如 `base.css`, `variables.css`）
  - `images/` - 图片资源
  - `fonts/` - 字体文件
  - `icons/` - 图标文件
```typescript
// 在组件中引入样式
import '@/assets/css/base.css'

// 在组件中引入图片
import logoImg from '@/assets/images/logo.png'
```

**@components** - 公共组件目录
- 存放可在多个页面复用的公共组件
- 文件命名使用 PascalCase（如 `Button.vue`, `UserCard.vue`）
- 组件应该是通用的、无业务属性的 UI 组件
- 业务相关组件建议放在对应页面目录下
- 组件按功能模块组织（可选）：`Button/`, `Card/`, `Form/` 等

**@components/ui** - 自定义 UI 组件目录
- **重要**：存放自定义的 UI 组件或对 Konsta UI 组件的包装
- Konsta UI 组件通过 npm 包引入，按需导入使用
- 如果需要封装或扩展 Konsta UI 组件，可以放在此目录
- 文件命名使用 PascalCase（如 `Button.vue`, `Card.vue`, `Dialog.vue`）
- 示例目录结构：
  ```
  components/
  ├── ui/              # 自定义 UI 组件
  │   ├── markdown-renderer/
  │   │   └── MarkdownRenderer.vue
  │   └── ...
  ├── Layout/          # 布局组件
  │   ├── Header.vue
  │   └── Footer.vue
  └── ...
  ```

**@constants** - 常量目录
- 存放项目中使用的常量值
- 常量命名使用 `UPPER_SNAKE_CASE`
- 例如：`API_BASE_URL`, `MAX_PAGE_SIZE`, `DEFAULT_TIMEOUT`
```typescript
// constants/api.ts
export const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000'
export const API_TIMEOUT = 30000
export const MAX_PAGE_SIZE = 100
```

**@hooks** - 公共 Hooks 目录
- **重要**：此目录下的 hooks 必须无业务属性，保持通用性和可复用性
- 例如：`useRequest`, `useLocalStorage`, `useDebounce`, `useThrottle`
- 命名使用 `use` 前缀 + PascalCase（如 `useRequest.ts`）
- 这些 hooks 应该可以在任何业务场景中复用

**@pages** - 页面视图目录
- 存放页面级组件（路由对应的视图）
- 文件命名使用 PascalCase（如 `UserList.vue`, `PostDetail.vue`）
- 页面组件应保持简洁，专注于 UI 渲染
- 复杂的业务逻辑应提取到 `@hooks` 或 `@stores` 中

**@router** - 路由目录
- 存放路由配置
- 使用 Vue Router 进行路由管理
- 路由定义应与页面文件结构保持一致

**@stores** - 状态管理目录
- 存放 Pinia stores，包含业务逻辑和状态管理
- 文件命名使用 kebab-case（如 `user-store.ts`, `post-store.ts`）
- Store 应包含：
  - 状态定义（state）
  - 业务逻辑（actions）
  - 计算属性（getters）

```typescript
// stores/user-store.ts
export const useUserStore = defineStore('user', () => {
  const user = ref<IUser | null>(null)
  
  async function fetchUser(id: string) {
    // 业务逻辑
  }
  
  return { user, fetchUser }
})
```

**@types** - 类型定义目录
- 存放 TypeScript 类型和接口定义
- 遵循命名规范：Interface 使用 `I` 前缀 + PascalCase，Type 使用 PascalCase
- 文件命名使用 kebab-case（如 `user.types.ts`, `api.types.ts`）

**@utils** - 工具函数目录
- 存放纯函数工具方法
- 文件命名使用 kebab-case（如 `date-utils.ts`, `format-utils.ts`）
- 工具函数应该无副作用、可测试


## 前端开发重点

### 组件编写
- 使用 `<script setup lang="ts">` 语法
- 使用 Composition API
- 组件应专注于视图渲染，复杂逻辑提取到 hooks 或 stores
- 合理选择组件放置位置：
  - 通用组件放在 `@components`
  - 业务组件放在对应页面目录

### Konsta UI 使用
- 组件通过 `konsta/vue` 包导入使用
- 在 `App.vue` 中使用 `k-app` 包裹应用
- 按需导入组件，减少打包体积
- 如需扩展，可在 `@components/ui` 中创建包装组件

### 接口开发
- 接口开发需要使用 alova 进行开发
- 对于普通接口，在使用时需要使用 useRequest 进行请求状态管理
- 对于需要动态监听的接口，可以使用 useWatcher 进行请求状态管理

### 样式
- 使用 Tailwind CSS 进行样式开发
- 如需在 style 中编写class，需要使用 `@apply` 进行样式组合

### Hooks 分类
- **公共 Hooks（@hooks）**: 必须无业务属性，保持通用性
- **业务 Hooks**: 可在页面目录中创建，包含特定业务逻辑

### Store 使用
- 使用 Pinia 进行状态管理
- Store 文件命名：kebab-case（如 `user-store.ts`）
- API 调用封装在 stores 中

### 类型定义
- Interface 使用 `I` 前缀 + PascalCase（如 `IUser`）
- Type 使用 PascalCase（如 `UserRole`）
- 文件命名：kebab-case（如 `user.types.ts`）

### 代码规范
- 所有函数必须添加 JSDoc 注释
- 单个组件不超过 300 行
- 单个函数不超过 150 行
- 使用 `async/await`，禁止混用 `.then()`

### 分层开发
- **视图层**: 专注于 UI 渲染，不直接调用 API
- **数据层**: Store 管理状态和业务逻辑
- **接口层**: API 请求封装

---
> Source: [youranreus/Applog](https://github.com/youranreus/Applog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
