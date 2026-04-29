## pitchmaster-ai

> 本文档旨在为 PitchBetter.ai 项目建立一套标准的开发规范，以提高代码质量、可维护性和团队协作效率。所有项目成员都应遵循本文档中的约定。

# PitchBetter.ai 项目开发规范

本文档旨在为 PitchBetter.ai 项目建立一套标准的开发规范，以提高代码质量、可维护性和团队协作效率。所有项目成员都应遵循本文档中的约定。

## 1. 技术栈

- **框架**: @Vue 3
- **构建工具**: @Vite
- **语言**: @TypeScript
- **路由**: @Vue Router
- **状态管理**: @Pinia
- **HTTP 请求**: @Axios
- **UI 组件/图标**: @Lucide Icons
- **代码风格**: @ESLint + @Prettier

## 2. 文件夹结构

为了保持项目清晰、可扩展，我们推荐以下文件夹结构。这在您现有结构的基础上增加了 `api`, `composables`, `utils` 和 `constants` 目录。

```
src
├── api/             # API 请求模块 (按业务模块划分)
│   ├── auth.ts
│   └── project.ts
├── assets/          # 静态资源 (图片, 全局样式等)
│   └── styles/
│       └── main.css
├── components/      # 全局通用组件
│   ├── common/      # 基础原子组件 (Button, Input, Modal)
│   └── layout/      # 布局组件 (Navbar, Sidebar, Footer)
├── composables/     # 可复用的组合式函数 (Hooks)
│   ├── useAuth.ts
│   └── useApi.ts
├── constants/       # 全局常量
│   ├── index.ts
│   └── validation.ts
├── router/          # 路由配置
│   └── index.ts
├── stores/          # Pinia 状态管理
│   ├── user.ts
│   └── project.ts
├── types/           # TypeScript 类型定义
│   ├── api.ts
│   └── index.ts
├── utils/           # 工具函数
│   ├── formatDate.ts
│   └── index.ts
├── views/           # 页面级组件
│   ├── HomeView.vue
│   └── LoginView.vue
├── App.vue          # 根组件
└── main.ts          # 应用入口
```

## 3. 命名约定

- **文件夹**: `kebab-case` (短横线连接), 复数形式 (如 `components`, `views`)。
- **组件文件**: `PascalCase` (大驼峰命名法), e.g., `MyComponent.vue`。
- **TS/JS 文件**: `camelCase` (小驼峰命名法), e.g., `useAuth.ts`, `apiClient.ts`。
- **变量/函数**: `camelCase`, e.g., `const userName = 'test'`。
- **常量**: `UPPER_CASE` (大写下划线), e.g., `const API_TIMEOUT = 5000`。
- **Pinia Stores**: `camelCase` 并以 `Store` 结尾, e.g., `useUserStore`。

## 4. 开发规范

### 4.1. 组件开发

- **SFC (单文件组件)**: 始终使用 `<script setup lang="ts">` 语法糖来编写组件，代码更简洁。
- **Props 定义**: 使用 `defineProps` 宏，并为每个 prop 提供明确的类型定义。
- **事件定义**: 使用 `defineEmits` 宏来声明所有触发的事件。
- **组件拆分**: 保持组件的单一职责原则。如果一个组件过于复杂，应将其拆分为更小的、可复用的子组件。
- **样式**: 默认使用 `<style scoped>` 来避免样式污染。全局样式应放在 `src/assets/styles` 目录下。

```vue
<!-- good -->
<script setup lang="ts">
interface Props {
  title: string;
  items: string[];
}
const props = defineProps<Props>();

const emit = defineEmits(['select']);

function onSelectItem(item: string) {
  emit('select', item);
}
</script>
```

### 4.2. 状态管理 (Pinia)

- **模块化**: 每个 store 对应一个业务模块 (e.g., `user.ts`, `project.ts`)。
- **命名**: 在 `stores` 目录中，文件名使用 `camelCase` (e.g. `userStore.ts`)。导出的 store setup 函数使用 `use...Store` 格式，如 `export const useUserStore = defineStore(...)`。
- **State**: 使用函数返回 state 对象，以确保服务端渲染 (SSR) 和测试的兼容性。
- **Getters**: 用于派生状态，类似于计算属性。
- **Actions**: 用于处理业务逻辑和异步操作。所有对 state 的修改都应在 actions 中进行。

### 4.3. API 请求

- **分层**: 在 `src/api` 目录下创建 API 服务层。按业务模块创建文件 (e.g., `auth.ts`, `project.ts`)。
- **封装 Axios**: 可以在 `utils` 或 `api` 目录下创建一个 `apiClient.ts` 文件，封装 Axios 实例，统一处理请求/响应拦截器 (如添加 token, 错误处理)。
- **类型定义**: 为所有 API 的请求参数和响应数据创建 TypeScript 类型或接口，存放在 `src/types/api.ts`。

### 4.4. 路由 (Vue Router)

- **中心化管理**: 所有路由定义在 `src/router/index.ts` 中。
- **懒加载**: 页面级组件 (`views`) 应使用动态导入进行懒加载，以优化首屏加载速度。
  ```typescript
  // good
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/LoginView.vue')
  }
  ```
- **路由守卫**: 在 `router/index.ts` 中使用 `beforeEach` 来处理全局路由守卫逻辑（如登录验证）。

### 4.5. TypeScript

- **严谨类型**: 尽可能为所有变量、函数参数和返回值添加明确的类型。避免使用 `any`。
- **接口与类型**: 在 `src/types` 目录下定义可复用的接口 (`interface`) 和类型别名 (`type`)。按模块组织文件。

### 4.6. 注释规范 (Commenting Guidelines)

- **TypeScript 代码**: 为关键函数、复杂逻辑、公共模块或业务相关的计算添加必要的注释。注释应包含中文和英文，以便团队所有成员理解。
  ```typescript
  /**
   * @description Fetches user profile by ID. // 根据ID获取用户资料
   * @param {string} userId - The ID of the user. // 用户ID
   * @returns {Promise<User>} The user's profile. // 用户的个人资料
   */
  async function fetchUserProfile(userId: string): Promise<User> {
    // ...
  }
  ```

- **CSS 文件**: 对非显而易见的样式规则、`z-index` 值或 hack/workaround 写法添加注释。注释同样需要中英双语。
  ```css
  /* To ensure the modal appears above all other content */
  /* 确保模态框显示在所有其他内容之上 */
  .modal-overlay {
    z-index: 1000;
  }
  ```

## 5. Git 工作流

推荐使用一个简化的 Git-flow 模型：

- **`main`**: 主分支，用于部署生产环境。只接受来自 `develop` 分支的合并。
- **`develop`**: 开发分支，作为所有功能开发的基准分支。
- **`feat/feature-name`**: 功能分支，从 `develop` 创建。开发新功能时使用，完成后合并回 `develop`。
- **`fix/bug-name`**: 修复分支，从 `develop` 创建，用于修复 bug。

---

这份文档将作为您项目的核心开发指南。随着项目的发展，您可以随时更新和完善它。 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ifishcool) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
