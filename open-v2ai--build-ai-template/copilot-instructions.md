## frontend-rules

> - **框架**: Next.js 15.3 + React 19


# 前端开发规范 - React/Next.js/TypeScript

## 技术栈

- **框架**: Next.js 15.3 + React 19
- **语言**: TypeScript 5.x
- **样式**: Tailwind CSS + Shadcn UI
- **状态管理**: React Hooks + Context API
- **国际化**: next-intl
- **包管理**: pnpm

## 文件组织与导入

- **组件**: 所有组件位于 `web/components/`，并按功能分组 (`ui/`, `admin/`, `sidebar/`)。
- **页面**: 页面位于 `web/app/[locale]/`，使用 Next.js App Router。
- **导入规范**: **必须**使用 `@` 绝对路径导入，禁止相对路径。

```typescript
// ✅ 正确
import { Button } from '@/components/ui/button'
import { fetcher } from '@/util/fetcher'

// ❌ 错误
import { Button } from './ui/button'
import { fetcher } from '../util/fetcher'
```

**导入顺序**:

1. React / Next.js 核心库
2. 第三方库
3. 项目内部模块 (组件、工具函数、类型等)
4. 样式文件

## 组件开发规范

- **命名**: 文件名 `kebab-case.tsx`, 组件名 `PascalCase`, Props 接口 `ComponentNameProps`。
- **客户端组件**: 如果需要客户端交互，必须在文件顶部添加 `'use client'`。
- **Props**: Props 接口应清晰定义，并包含 JSDoc 注释。
- **样式**: 使用 `cn()` 工具函数合并 Tailwind CSS 类名，实现条件样式。

```typescript
'use client';

import { cn } from '@/lib/utils';

interface MyComponentProps {
  /** 是否激活 */
  isActive: boolean;
  className?: string;
}

export default function MyComponent({ isActive, className }: MyComponentProps) {
  return (
    <div className={cn("base-styles", isActive && "active-styles", className)}>
      {/* ... */}
    </div>
  );
}
```

## TypeScript 类型规范

- **位置**: 共享类型放于 `web/types/`，页面专用类型放于页面目录下的 `types.ts`。
- **定义**:
  - 使用 `interface` 定义对象结构。
  - 使用 `type` 定义联合类型或工具类型。
  - 使用 `as const` 创建枚举对象，而不是 `enum`。

```typescript
interface User {
  id: number
  username: string
}

type UserRole = 'admin' | 'user'

const MembershipType = {
  FREE: 'free',
  MONTHLY: 'monthly',
} as const
type Membership = (typeof MembershipType)[keyof typeof MembershipType]
```

## API 调用与数据缓存

### API 调用

统一使用 `@/util/fetcher` 工具函数，它封装了认证和错误处理逻辑。

```typescript
import { fetcher } from '@/util/fetcher'

async function fetchUserData() {
  try {
    const data = await fetcher.get('/api/v1/profile', { needAuth: true })
    return data
  } catch (error) {
    toast.error('获取用户信息失败')
    console.error(error)
  }
}
```

### 全局数据缓存 (`useGlobalDataCache`)

为避免重复网络请求和保证数据一致性，项目使用 `useGlobalDataCache` Hook 管理核心数据（如聊天列表和消息）。

**核心原则**:

1. **读操作优先从缓存获取**: 在请求网络前，先检查数据是否存在于缓存中。
2. **写操作更新缓存**: 数据变更后（增、删、改），应直接更新本地缓存，而不是重新拉取整个列表。
3. **使用统一数据源**: 所有组件都应从该 Hook 获取数据，避免维护独立的状态。

```typescript
import { useGlobalDataCache } from '@/hooks/use-global-data-cache'

function ChatSidebar() {
  const { chats, isLoading } = useGlobalDataCache()
  // ... 渲染聊天列表
}

function ChatContent() {
  const { getChatById, addMessageToChat } = useGlobalDataCache()
  // ... 使用 getChatById 获取聊天详情
  // ... 发送消息后，使用 addMessageToChat 更新缓存
}
```

## 国际化 (i18n)

- **翻译文件**: 位于 `web/app/messages/`，按语言 (`en.json`, `zh.json`) 组织。
- **命名规范**: 使用嵌套 JSON 结构，按 `模块.页面.键` 的方式组织。
- **使用**: 在组件中使用 `useTranslations` Hook 获取翻译函数 `t`。

```typescript
import { useTranslations } from 'next-intl';

const t = useTranslations('common'); // 加载 common 命名空间
return <Button>{t('submit')}</Button>;
```

## 性能优化

- **组件**: 对计算成本高的纯组件使用 `React.memo`，对复杂计算和函数分别使用 `useMemo` 和 `useCallback`。
- **图片**: 始终使用 `next/image` 组件进行图片优化。
- **代码分割**: 使用 `next/dynamic` 动态导入大型或不常用的组件。

## 核心开发规范

- **TypeScript 优先**: 严禁使用 `.js` 或 `.jsx` 文件。
- **中文注释**: 所有组件和复杂逻辑必须有详细的中文注释。
- **单一职责**: 保持组件功能单一，复杂组件应拆分为更小的子组件。
- **用户体验**: 必须处理加载 (`loading`) 和错误 (`error`) 状态，提供清晰的 UI 反馈。
- **响应式设计**: 所有页面和组件必须在移动端和桌面端表现良好。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
