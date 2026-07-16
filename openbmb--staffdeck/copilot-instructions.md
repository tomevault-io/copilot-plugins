## frontend-ui-components

> UI component conventions for frontend-enterprise (prefer shadcn/ui)


# 前端组件规范（frontend-enterprise）

新页面或重构时，**优先使用 shadcn/ui 组件**，而非新增 Ant Design 组件。

- shadcn/ui 组件位于 `@/components/ui`，通过 `@/components/ui` 桶文件统一导入。
- 类名合并使用 `cn()`（来自 `@/lib/utils`）。
- 通知使用 `sonner` 的 `toast`，而非 `antd` 的 `message`。
- 现有页面中的 Ant Design 组件保持不变，除非本次任务明确要求重构；不要为了替换而替换。

```tsx
// ✅ GOOD — 新页面使用 shadcn/ui
import { Button, Card, CardContent, CardHeader, CardTitle, Input } from '@/components/ui';
import { toast } from 'sonner';
import { cn } from '@/lib/utils';

// ❌ AVOID — 新页面不再新增 Ant Design 组件
import { Button, Card, Input, message } from 'antd';
```

新增尚未安装的 shadcn 组件：

```bash
cd frontend-enterprise && npx shadcn@latest add <component> -y
```

# 图标规范

页面中使用的各种 icon，**先在 `src/assets/` 下生成 SVG 图标文件，再引入页面中使用**，不要在 JSX 里内联手写 SVG 路径。

```tsx
// ✅ GOOD — 先落到 assets，再作为组件/资源引入
import ArrowIcon from '@/assets/icons/arrow.svg?react';
// 或统一封装后按名引用
import { StaffdeckIcon } from '@/components/StaffdeckIcon';

<ArrowIcon className="size-4" />;

// ❌ AVOID — 在页面里内联手写 SVG
<svg viewBox="0 0 24 24"><path d="M..." /></svg>;
```

# 枚举规范

考虑后续维护，**状态、类型等有限取值的字段应先在 `enums` 中定义枚举/常量，再引用**，避免在业务代码里散落魔法字符串。

```ts
// ✅ GOOD — 先在 enums 中集中定义
// src/enums/agentStatus.ts
export enum AgentStatus {
  Active = 'active',
  Onboarding = 'onboarding',
  Archived = 'archived',
}

if (agent.status === AgentStatus.Active) { /* ... */ }

// ❌ AVOID — 业务代码里散落魔法字符串
if (agent.status === 'active') { /* ... */ }
```

---
> Source: [OpenBMB/StaffDeck](https://github.com/OpenBMB/StaffDeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
