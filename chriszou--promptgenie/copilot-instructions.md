## ui-components

> PromptGenie 使用自定义 React 组件构建用户界面。

# UI 组件

PromptGenie 使用自定义 React 组件构建用户界面。

## 布局组件

- [src/components/Layout.tsx](mdc:src/components/Layout.tsx) - 主布局组件
- [src/components/Sidebar.tsx](mdc:src/components/Sidebar.tsx) - 侧边栏组件
- [src/components/Header.tsx](mdc:src/components/Header.tsx) - 头部导航栏

## 提示词相关组件

- [src/components/PromptList.tsx](mdc:src/components/PromptList.tsx) - 提示词列表表格
- [src/components/PromptCard.tsx](mdc:src/components/PromptCard.tsx) - 提示词卡片
- [src/components/PromptForm.tsx](mdc:src/components/PromptForm.tsx) - 提示词表单

## 标签相关组件

- [src/components/TagSelector.tsx](mdc:src/components/TagSelector.tsx) - 标签选择器
- [src/components/TagBadge.tsx](mdc:src/components/TagBadge.tsx) - 标签徽章
  
## UI 工具

应用使用 [Lucide React](mdc:https:/lucide.dev/docs/lucide-react) 作为图标库:
```typescript
import { StarIcon, CopyIcon, PencilIcon, TagIcon, TrashIcon } from 'lucide-react';
```

## 实用工具

- [src/utils/time.ts](mdc:src/utils/time.ts) - 时间格式化函数

---
> Source: [ChrisZou/promptgenie](https://github.com/ChrisZou/promptgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
