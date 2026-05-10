## i18n-styling

> 国际化和样式开发规范和 Tailwind CSS/Shadcn UI 最佳实践


# 国际化和样式开发规范

## 技术栈

- **样式框架**: Tailwind CSS 3.x
- **组件库**: Shadcn UI
- **国际化**: next-intl
- **图标**: Lucide React
- **主题**: next-themes

## 国际化 (i18n) 规范

### 翻译文件结构

翻译文件位于 `web/app/messages/` (`zh.json`, `en.json`)。使用命名空间来组织翻译键。

```json
// zh.json
{
  "common": {
    "submit": "提交",
    "cancel": "取消",
    "loading": "加载中..."
  },
  "auth": {
    "login": "登录",
    "logout": "退出登录"
  },
  "error": {
    "networkError": "网络错误，请稍后重试",
    "unauthorized": "未授权，请先登录"
  }
}
```

- **命名规范**: 使用 `模块.功能.具体项` 的方式，如 `auth.login.title`。避免超过三层嵌套。

### 在组件中使用翻译

```typescript
'use client';
import { useTranslations } from 'next-intl';

export default function LoginComponent() {
  const t = useTranslations('auth');
  const tCommon = useTranslations('common');

  return (
    <form>
      <label>{t('email')}</label>
      <button type="submit">{tCommon('submit')}</button>
    </form>
  );
}
```

对于服务端组件 (RSC)，使用 `getTranslations`。

## Tailwind CSS 规范

### 类名组织

使用 `cn()` 工具函数，并遵循一致的顺序以提高可读性：

1. **布局** (position, display)
2. **盒模型** (width, height, margin, padding, border)
3. **背景**
4. **文字** (font-size, color)
5. **其他**
6. **过渡与动画**
7. **响应式** (md:, lg:)
8. **状态** (hover:, focus:, disabled:)

```typescript
<div className={cn(
  "flex w-full p-4 bg-white dark:bg-gray-900",
  "text-lg text-gray-900 dark:text-white",
  "transition-colors hover:bg-gray-100",
  isActive && "border-blue-500"
)} />
```

### 响应式与暗黑模式

- **移动优先**: 默认样式适用于移动端，使用 `md:`, `lg:` 等前缀扩展到大屏幕。
- **暗黑模式**: 使用 `dark:` 前缀为暗黑模式提供特定样式。

```typescript
<div className="bg-white text-black dark:bg-gray-800 dark:text-white" />
```

## Shadcn UI 组件

- **安装**: 使用 `npx shadcn-ui@latest add <component-name>` 添加新组件。
- **使用**: 从 `@/components/ui/...` 导入组件。通过 `variant`, `size` 等 props 进行定制，并通过 `className` 覆盖样式。

```typescript
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";

<Button variant="destructive" size="sm" className="w-full">
  删除
</Button>

<Card>
  <CardHeader><CardTitle>标题</CardTitle></CardHeader>
  <CardContent><p>内容...</p></CardContent>
</Card>
```

## 主题系统 (`next-themes`)

- **Provider**: `ThemeProvider` 在 `layout.tsx` 中配置，提供全局主题支持。
- **切换**: `ThemeToggleButton` 组件使用 `useTheme` Hook 来切换亮色/暗色/系统主题。

## 图标 (Lucide React)

从 `lucide-react` 导入图标，并通过 `className` 控制大小和颜色。

```typescript
import { User, Loader2 } from 'lucide-react';

<Button>
  <User className="mr-2 h-4 w-4" />
  登录
</Button>

// 加载状态
<Loader2 className="h-4 w-4 animate-spin" />
```

## 最佳实践总结

- **国际化**:
  - 保持翻译键结构清晰，及时更新所有语言文件。
- **样式**:
  - 遵循移动优先和一致的类名顺序。
  - 充分利用 Tailwind CSS，避免自定义 CSS 和内联样式。
- **组件**:
  - 优先使用并扩展 Shadcn UI 组件。
  - 保持组件 API 简洁，支持样式覆盖。
- **可访问性 (A11y)**:
  - 为交互元素添加 `aria-*` 标签。
  - 确保应用支持键盘导航和焦点状态。
  - 保证颜色有足够的对比度。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
