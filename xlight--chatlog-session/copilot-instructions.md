## chatlog-session

> Vue 3 + TypeScript + Vite 前端应用，用于通过 Chatlog API 查看微信聊天记录。使用 Pinia 状态管理、Element Plus UI 组件库、IndexedDB 本地存储。

# AGENTS.md - Development Guide for Chatlog Session

## 使用中文

- 本项目的注释和文档均使用中文编写
- LLM 输出保持中文
- 用户提示和错误信息使用中文

## Project Overview

Vue 3 + TypeScript + Vite 前端应用，用于通过 Chatlog API 查看微信聊天记录。使用 Pinia 状态管理、Element Plus UI 组件库、IndexedDB 本地存储。

## Build Commands

```bash
pnpm dev          # 开发服务器 (端口 5173，代理后端 127.0.0.1:5030)
pnpm build        # 生产构建 (vue-tsc 类型检查 + vite build)
pnpm preview      # 预览生产构建
pnpm type-check   # 仅类型检查 (vue-tsc --noEmit)
pnpm lint         # ESLint 检查并自动修复
pnpm format       # Prettier 格式化 src/
```

## Package Manager

- 项目统一使用 `pnpm`
- 初次进入仓库建议先执行 `corepack enable`
- 如需紧急回滚到 npm，需同时恢复 `package-lock.json`、CI 中的 `npm ci` 配置，以及文档中的 npm 命令说明

本项目无测试框架，无单元测试。

## Code Style (Prettier)

- **No semicolons** (`"semi": false`)
- **Single quotes** (`'string'`)
- **Trailing commas** ES5 style
- **2 spaces** indentation, no tabs
- **100 char** print width
- **Arrow parens**: avoid for single params (`x => x`)
- **LF** line endings
- **Bracket spacing**: `{ key: value }`

## Imports

```typescript
// Vue/Router/Pinia API 自动导入，不要手动 import:
//   ref, reactive, computed, watch, watchEffect, onMounted, onUnmounted,
//   useRoute, useRouter, defineStore, storeToRefs, nextTick, etc.

// 需要手动导入:
import type { Contact } from '@/types/contact' // 类型用 import type
import { useAppStore } from '@/stores/app' // Store
import { request } from '@/utils/request' // 工具函数
import { ElMessage } from 'element-plus' // Element Plus 消息 (非组件)

// 路径别名: @/ → src/
// Element Plus 组件和图标已通过 unplugin 自动导入，无需手动 import
```

## Vue Components

- 模板: `<script setup lang="ts">` exclusively
- Props: `interface Props { ... }` + `withDefaults(defineProps<Props>(), {})`
- 样式: `<style lang="scss" scoped>` (全局样式除外)
- 文件名: PascalCase (`EmptyState.vue`, `SearchBar.vue`)
- SCSS 全局变量通过 `@use "@/assets/styles/variables.scss" as *` 自动注入

## ESLint Rules

- `vue/multi-word-component-names`: off
- `vue/require-default-prop`: off
- `@typescript-eslint/no-explicit-any`: warn (尽量避免，用 `unknown` 替代)
- `@typescript-eslint/no-unused-vars`: error, `_` 前缀变量除外
- `prefer-const`: error
- `no-var`: error
- `no-console`: production 环境 warn
- `no-debugger`: production 环境 error

## Naming Conventions

| 类别        | 规则                     | 示例                                  |
| ----------- | ------------------------ | ------------------------------------- |
| Components  | PascalCase 文件名        | `EmptyState.vue`                      |
| Composables | `useXxx.ts`              | `useKeyboardShortcuts.ts`             |
| Stores      | 描述性名称               | `stores/app.ts`, `stores/contact.ts`  |
| Types       | PascalCase interface     | `Contact`, `SessionInfo`, `AppConfig` |
| Utils       | camelCase 文件，命名导出 | `utils/format.ts`, `utils/date.ts`    |
| API files   | 领域名称                 | `api/contact.ts`, `api/session.ts`    |
| 常量        | UPPER_SNAKE_CASE         | `API_BASE_PATH`, `ContactType`        |

## State Management (Pinia)

两种模式共存，新代码使用 Composition API:

```typescript
// Composition API (推荐，大多数 store 使用此模式)
export const useXxxStore = defineStore('xxx', () => {
  // ==================== State ====================
  const data = ref<Type[]>([])
  const loading = ref(false)
  const error = ref<Error | null>(null)

  // ==================== Getters ====================
  const filtered = computed(() => data.value.filter(...))

  // ==================== Actions ====================
  async function fetchData() {
    loading.value = true
    error.value = null
    try {
      data.value = await someAPI.getData()
    } catch (err) {
      error.value = err instanceof Error ? err : new Error(String(err))
    } finally {
      loading.value = false
    }
  }

  function $reset() { /* 重置所有 state */ }

  // ==================== Return ====================
  return { data, loading, error, filtered, fetchData, $reset }
})
```

- 所有 store 必须提供 `$reset()` 方法
- 使用 `// ==================== Section ====================` 分隔代码块
- 调试日志使用 emoji 前缀: `📱📦🔍💬✅❌🔄🗑️`, 通过 `appStore.isDebug` 门控

## API Layer Pattern

API 使用 class 单例模式，每个文件包含 `transform` 函数转换后端数据:

```typescript
// 后端数据结构 (定义在 API 文件中)
interface BackendXxx { ... }

// 转换函数
function transformXxx(backend: BackendXxx): FrontendType { ... }

// API 类
class XxxAPI {
  async getItems(params?: Params): Promise<Item[]> {
    const response = await request.get<BackendResponse>('/api/v1/xxx', params)
    return response.items.map(transformXxx)
  }
}

export const xxxAPI = new XxxAPI()
export default xxxAPI
```

- HTTP 客户端: `request` from `@/utils/request` (axios wrapper)
- API 路径前缀: `/api/v1/`
- 所有请求自动添加 `format=json` 参数
- 错误由 axios 拦截器统一处理 (ElMessage + 自动重试)

## Error Handling

- async 操作使用 try-catch，error 存入对应 store 的 `ref<Error | null>`
- 用户提示使用 `ElMessage.error('中文错误描述')`
- HTTP 错误由 `utils/request.ts` 拦截器统一处理 (含自动重试)
- 调试日志门控: `if (appStore.isDebug) { console.log(...) }`

## Comments

- 业务逻辑注释使用中文
- 导出函数使用 JSDoc (中文描述)
- 代码段落使用分隔线: `// ==================== Section ====================`

## Project Structure

```
src/
├── api/           # API 层 - class 单例 + transform 函数
├── assets/styles/ # SCSS 全局变量、mixins、基础样式
├── components/    # 按功能分组: chat/, common/, layout/, search/, PWA/
├── composables/   # 可复用组合函数 (useXxx.ts)
├── router/        # Vue Router - createWebHistory, 懒加载路由
├── stores/        # Pinia stores (Composition API 为主)
├── types/         # TypeScript 类型定义，按领域拆分
├── utils/         # 工具函数: request, db, format, date, storage 等
└── views/         # 页面组件: Chat/, Contact/, Search/, Settings/, Dashboard/
```

## Key Dependencies

- Vue 3.4+, TypeScript ~5.3, Vite 7.x
- Pinia 3.x (状态管理), Element Plus 2.x (UI), Vue Router 4.x
- axios (HTTP), dayjs (日期), pinyin-pro (拼音), dompurify (XSS 防护)
- vue-virtual-scroller (虚拟滚动), @vueuse/core (实用 composables)
- marked (Markdown 渲染), idb/IndexedDB (本地持久化)

## Environment Variables (.env)

| 变量                | 说明          | 默认值                |
| ------------------- | ------------- | --------------------- |
| `VITE_APP_TITLE`    | 应用标题      | Chatlog Session       |
| `VITE_API_BASE_URL` | 后端 API 地址 | http://127.0.0.1:5030 |
| `VITE_PAGE_SIZE`    | 默认分页大小  | 500                   |
| `VITE_ENABLE_DEBUG` | 启用调试模式  | false                 |
| `VITE_BASE_PATH`    | 部署基础路径  | ./                    |

## Caching Architecture

- **SessionStorage**: 消息缓存 (LRU + TTL, `stores/messageCache.ts`)
- **IndexedDB**: 联系人和群聊信息持久化 (`utils/db.ts`)
- **localStorage**: 用户设置、置顶会话、API 配置
- 缓存事件: `window.dispatchEvent(new CustomEvent('chatlog-cache-updated', ...))`

## Auto-generated Files (勿手动编辑)

- `src/auto-imports.d.ts` - unplugin-auto-import 生成
- `src/components.d.ts` - unplugin-vue-components 生成
- `.eslintrc-auto-import.json` - auto-import ESLint globals

---
> Source: [xlight/chatlog-session](https://github.com/xlight/chatlog-session) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
