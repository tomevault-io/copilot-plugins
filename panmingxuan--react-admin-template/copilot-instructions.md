## react-admin-template

> > 本文档整合代码约束设计思想与行为规则，

# 🧠 前端项目 AI 协作开发规范

> 本文档整合代码约束设计思想与行为规则，  
> 结合现代前端开发最佳实践，用于指导 AI 在前端项目中的代码生成与协作行为。

---

## 📐 第一部分：架构设计原则（Architecture Principles）

### 一、组件单一职责原则

> 每个组件只做一件事，做好一件事。

**实践**

- 展示组件与容器组件分离
- 布局组件与业务组件解耦
- 一个文件一个导出组件（除内部辅助组件）

### 二、类型契约优先原则

> Props 即契约，类型即文档。

**实践**

- 所有组件 Props 必须定义独立类型/接口
- 使用 `export type` 导出类型定义
- 禁止使用 `any`、`unknown` 作为 Props 类型

### 三、依赖隔离原则

> 组件不依赖具体实现，通过 Props 注入依赖。

**实践**

- 路由逻辑通过 Props 回调或 Render Props 注入
- 路由 API（如 TanStack Router）仅在适配器中使用
- 通用组件保持框架无关

### 四、样式一致性原则

> 设计系统即代码规范。

**实践**

- 使用 Tailwind CSS + CSS 变量管理主题
- 遵循 Catalyst Design System 色彩与间距规范
- 暗色模式使用透明度映射而非硬编码颜色

### 五、可访问性优先原则

> 每个交互元素都必须可访问。

**实践**

- 所有 `<button>` 声明 `type` 属性
- 所有 `<img>` 包含有意义的 `alt` 属性
- 所有 `<svg>` 包含 `<title>` 元素
- 使用语义化 HTML 标签

---

## 🧩 第二部分：代码行为规范（Execution Rules）

### 1. TypeScript 规范

```typescript
// ✅ 正确：使用 import type
import type { ReactNode } from 'react';

// ❌ 错误：直接导入类型
import { ReactNode } from 'react';

// ✅ 正确：明确的 Props 类型
interface ButtonProps {
  children: ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

// ❌ 错误：使用 any
interface ButtonProps {
  onClick: any;
}
```

### 2. React/JSX 规范

```tsx
// ✅ 正确：Hook 在组件顶层
function Component() {
  const [state, setState] = useState(false);
  // ...
}

// ❌ 错误：条件内使用 Hook
function Component({ show }) {
  if (show) {
    const [state, setState] = useState(false); // 错误！
  }
}

// ✅ 正确：使用唯一标识作为 key
{items.map((item) => (
  <Item key={item.id} {...item} />
))}

// ❌ 错误：使用数组索引作为 key
{items.map((item, index) => (
  <Item key={index} {...item} />
))}
```

### 3. 样式规范

```tsx
// ✅ 正确：使用语义化的 Tailwind 类名组合
<button className="rounded-lg bg-zinc-950/5 px-3 py-2 text-sm font-medium text-zinc-900 hover:bg-zinc-950/10">

// ❌ 错误：内联样式
<button style={{ backgroundColor: 'rgba(0,0,0,0.05)' }}>

// ✅ 正确：暗色模式使用透明度
<div className="bg-white dark:bg-zinc-900 border-zinc-950/5 dark:border-white/10">

// ❌ 错误：硬编码暗色值
<div className="bg-white dark:bg-[#18181b]">
```

### 4. 组件结构规范

```tsx
// ✅ 正确：Props 接口 + 函数声明
interface SidebarProps {
  items: NavItem[];
  currentPath?: string;
  onNavigate?: (path: string) => void;
}

export function Sidebar({ items, currentPath, onNavigate }: SidebarProps) {
  // 实现
}

// ❌ 错误：匿名导出 + 内联类型
export default ({ items }: { items: any[] }) => {
  // ...
}
```

### 5. 文件组织规范

```
/components
  /layout
    sidebar.tsx          # 主组件
    styles.ts            # 样式常量
    types.ts             # 类型定义
    index.ts             # 统一导出
    /adapters            # 路由适配器
  /ui                    # 基础 UI 组件（shadcn/ui + Catalyst wrapper）
    /shadcn              # shadcn/ui 原语层（CLI 生成，不直接修改）
      button.tsx
      input.tsx
      dialog.tsx
      ...
    /button              # Button wrapper 目录
      button.tsx         # Catalyst 风格封装
      types.ts           # Props 类型定义
      index.ts           # 统一导出
    /input               # Input wrapper 目录（同上结构）
    /dialog              # Dialog wrapper 目录（同上结构）
    button.tsx           # 顶层入口 re-export
    input.tsx            # 顶层入口 re-export
    dialog.tsx           # 顶层入口 re-export
```

---

## 🎨 第三部分：UI 设计准则引用

> 详细的 UI 设计规范请参阅 `doc/UI_DESIGN_SPEC.md`

**核心要点速览：**

- **色系**：Zinc 中性灰 + 透明度（Alpha）
- **圆角**：`rounded-lg` (8px) 作为标准
- **间距**：`px-3 py-2` 紧凑，`px-6 py-8` 宽松
- **图标**：20px（`size-5`），1.5px 线宽
- **分割线**：`border-zinc-950/5` 极淡透明

---

## 🔍 第四部分：UI 问题排查引用

> 详细的 UI 问题排查指南请参阅 `doc/UI_TROUBLESHOOTING.md`

**排查流程速览：**

```
1. 控制台错误 → 2. 元素检查 → 3. 计算样式 → 4. 布局调试 → 5. 代码审查
```

**常见问题分类：**

| 问题类型 | 排查方向 |
|---------|---------|
| 布局错位 | Flexbox/Grid、overflow、position |
| 样式不生效 | 类名拼写、优先级、Tailwind 配置 |
| 暗色模式 | dark: 变体、透明度映射 |
| 交互状态 | hover/focus/active、z-index |
| 响应式 | 断点设置、容器宽度 |

**快速诊断提示词：**

```markdown
请帮我排查以下 UI 问题：
- 问题描述：[现象]
- 期望效果：[目标]
- 相关文件：[路径]
请根据 doc/UI_DESIGN_SPEC.md 和 doc/UI_TROUBLESHOOTING.md 进行分析。
```

---

## ⚙️ 第五部分：AI 协作策略

### 生成前检查

1. 扫描现有组件模式与命名约定
2. 识别项目使用的设计系统
3. 本项目使用 React + Vite + TanStack Router

### 生成中遵守

1. 遵循本文档所有规范
2. 参考 `doc/UI_DESIGN_SPEC.md` 设计准则
3. 遇到 UI 问题参考 `doc/UI_TROUBLESHOOTING.md` 排查指南
4. 保持与现有代码风格一致

### 生成后验证

1. 类型检查通过
2. 无 ESLint/Biome 错误
3. 组件可独立运行测试
4. UI 显示符合设计规范

---

## 🛠️ 第六部分：cn 函数与样式工具

> 基于 `tailwind-merge` + `clsx`，遵循 Tailwind CSS v4 最佳实践

### 核心理念：CSS-First 配置

Tailwind CSS v4 推荐将设计 tokens 定义在 CSS 中而非 JavaScript：

```css
/* src/styles/globals.css */
@import "tailwindcss";

@theme {
  /* 字体系统 */
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-display: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-mono: ui-monospace, "SF Mono", "Menlo", monospace;

  /* 品牌色 - 使用 OKLCH 现代颜色空间 */
  --color-brand: oklch(0.55 0.2 250);
  --color-brand-light: oklch(0.75 0.15 250);
  --color-brand-dark: oklch(0.35 0.2 250);

  /* 语义化颜色 */
  --color-success: oklch(0.65 0.2 145);
  --color-warning: oklch(0.75 0.18 85);
  --color-error: oklch(0.55 0.22 25);
  --color-info: oklch(0.6 0.2 250);
}
```

### Catalyst 设计系统核心参数

> 本项目基于 Catalyst Design System，使用 `Zinc` 色系 + 透明度(Alpha) 设计理念

#### 布局参数

| 组件部分 | Tailwind Class | 设计意图 |
|---------|----------------|----------|
| Sidebar 宽度 | `w-72` (288px) | 比传统 256px 更宽，更现代 |
| 分割线 | `border-zinc-950/5` | 极淡透明黑，非实色灰 |
| 内边距（宽松） | `px-6 py-8` | 留白充足，呼吸感强 |
| 内边距（紧凑） | `px-3 py-2` | 导航项/按钮标准间距 |
| 标准圆角 | `rounded-lg` (8px) | 组件标准圆角 |
| 卡片圆角 | `rounded-2xl` (16px) | 卡片/对话框圆角 |

#### 状态样式映射

| 状态 | 背景 | 文字 | 图标 |
|-----|------|------|------|
| 默认 | `transparent` | `text-zinc-500` | `text-zinc-400` |
| 悬停 | `bg-zinc-950/5` | `text-zinc-900` | `text-zinc-900` |
| 激活 | `bg-zinc-950/5` | `text-zinc-900` | `text-zinc-900` |

#### 暗色模式映射

| 亮色 | 暗色 |
|-----|------|
| `bg-white` | `dark:bg-zinc-900` |
| `text-zinc-950` | `dark:text-white` |
| `border-zinc-950/5` | `dark:border-white/10` |
| `hover:bg-zinc-950/5` | `dark:hover:bg-white/5` |

#### 图标规范

| 参数 | 值 | Tailwind |
|-----|-----|----------|
| 库 | Heroicons / Lucide | Outline 风格 |
| 尺寸 | 20px | `size-5` / `w-5 h-5` |
| 线宽 | 1.5px | `strokeWidth={1.5}` |
| 对齐 | 不收缩 | `shrink-0` |

### ⚠️ 重要注意事项

> **Tailwind CSS v4 的 `@apply` 不支持 `group`、`peer` 等特殊类**

因此以下场景需要直接在组件中使用 Tailwind 类名，而非使用 `@apply` 定义的 CSS 类：

- 导航项（需要 `group` 实现图标悬停变色）
- 下拉菜单项（需要 `group-hover` 效果）
- 任何使用 `peer`/`peer-*` 的表单联动

```tsx
// ✅ 正确：直接使用 Tailwind 类名
<a className="group flex gap-x-3 rounded-lg px-3 py-2">
  <Icon className="text-zinc-400 group-hover:text-zinc-900" />
  导航项
</a>

// ❌ 错误：无法在 @apply 中使用 group
.nav-item {
  @apply group flex gap-x-3; /* 报错！ */
}
```

### 核心工具函数

---

## 🔁 shadcn/ui 集成与迁移规范（新增）

目的：将项目 UI 组件基座由“手工 HTML/Tailwind 组合”逐步迁移为以 shadcn/ui 原语为基础的组件库适配层，同时保留 Catalyst 设计体系的视觉与交互规范。

- **组件基座变更**：新开发或重构的基础 UI 组件应以 `shadcn/ui` 提供的可复用原语为首选基座（如 Button、Input、Dialog 等），在本项目内通过轻量封装（wrapper）适配 Catalyst 设计 tokens 与 className 映射。
- **组件目录规划（示例）**：
  - `src/components/ui/` - 第三方组件封装层（对外导出统一 API）
    - `button/` - `index.tsx`, `button.tsx`, `types.ts`（封装 shadcn Button 并映射 Catalyst 变体）
    - `input/` - 同上
    - `dialog/` - 同上
  - `src/components/` - 业务组合组件（使用 `src/components/ui/*`）

- **严格约束**：AI 或开发者**不得手动复制 shadcn/ui 的源码到本仓库**作为替代方案。必须通过包管理器按需安装官方/社区发布的组件包，或使用 shadcn 官方脚手架/CLI 来生成组件骨架。
  - 交付物应包含：需要安装的组件清单（包名 + 推荐版本）和明确的 `pnpm` 安装命令示例，例如：`pnpm add <shadcn-package-name>`（实际包名由实现前的依赖清单确定）。

- **封装原则**：
  - 在 `src/components/ui/*` 中创建轻量 wrapper，将 shadcn 组件的 props 映射为项目约定的 Props（类型导出必须清晰）。
  - 视觉层使用 Catalyst 的 tokens（CSS 变量 / Tailwind classes），避免直接修改第三方源码。
  - 所有 wrapper 必须导出类型定义并通过 `export` 明确契约。

- **迁移与重构策略（AI 遵循）**：
  1. 先生成“所需 shadcn 组件清单 + 依赖安装命令 + 影响文件清单”，提交给用户审核并等待确认。
  2. 用户确认后，执行依赖安装（由用户或 CI 执行）。
  3. 在 `src/components/ui/` 下实现 wrapper 并编写小范围示例页验证样式。
  4. 逐步将现有业务组件从原始实现替换为新 wrapper（每次替换限一个子模块并提交 PR）。

- **测试与校验**：
  - 每个重构步骤需提供可复现的视觉检查点（截图/Story/小页），并列出回退方案。
  - 提交前运行类型检查与 Biome 格式化（AI 仅生成命令与建议，不要自动执行）。

请注意：以上为规范与流程文档，任何实际的依赖安装或代码更改必须在得到你（项目管理员）的明确批准后才执行。

```typescript
// 核心工具函数（精简版）
import {
  cn,                      // 类名合并
  getBadgeClassName,       // Badge 样式
  getAlertClassName,       // Alert 样式
  getCardClassName,        // Card 样式
  getChangeTextClassName,  // 变化指示器文本样式
} from '@/lib/utils';

// 组件使用方式（推荐直接使用 wrapper 组件）
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Dialog, DialogContent, DialogTrigger } from '@/components/ui/dialog';
import { Badge } from '@/components/ui/badge';
import { Alert } from '@/components/ui/alert';
import { Card } from '@/components/ui/card';
import { Avatar, AvatarImage, AvatarFallback } from '@/components/ui/avatar';
import { Separator } from '@/components/ui/separator';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
```

### cn 函数用法

```tsx
import { cn } from '@/lib/utils';

// ✅ 基础用法：后面的类覆盖前面冲突的类
cn('px-2 py-1', 'px-4')
// => 'py-1 px-4' (tailwind-merge 智能合并)

// ✅ 条件类名
cn('base', isActive && 'active', disabled && 'opacity-50')

// ✅ 对象语法
cn('base', { 'text-red-500': hasError, 'text-green-500': isSuccess })

// ✅ 使用 Tailwind 内置类（推荐）
cn('shadow-sm duration-150 ease-out')

// ✅ 使用自定义品牌色
cn('bg-brand text-white', 'hover:bg-brand-dark')

// ❌ 错误：使用模板字符串拼接
<div className={`base ${isActive ? 'active' : ''}`}>
```

### 可用样式函数

> ⚠️ **推荐优先使用 wrapper 组件**（如 `<Button>`、`<Badge>`）而非直接调用样式函数

| 函数 | 参数 | 说明 |
|------|------|------|
| `cn` | (...inputs) | 类名合并（tailwind-merge + clsx） |
| `getBadgeClassName` | (variant) | default/success/warning/error/info |
| `getAlertClassName` | (variant) | info/success/warning/error |
| `getCardClassName` | (padding) | sm/md/lg |
| `getChangeTextClassName` | (changeType) | positive/negative/neutral |

### 可用 UI 组件

| 组件 | 导入路径 | 说明 |
|------|----------|------|
| `Button` | `@/components/ui/button` | 按钮（primary/secondary/outline/ghost/destructive/link） |
| `Input` | `@/components/ui/input` | 输入框（default/error/success + sm/md/lg） |
| `Badge` | `@/components/ui/badge` | 徽章（default/success/warning/error/info） |
| `Alert` | `@/components/ui/alert` | 提示框（info/success/warning/error） |
| `Card` | `@/components/ui/card` | 卡片（sm/md/lg 内边距） |
| `Dialog` | `@/components/ui/dialog` | 对话框（Modal） |
| `DropdownMenu` | `@/components/ui/dropdown-menu` | 下拉菜单 |
| `Avatar` | `@/components/ui/avatar` | 头像（sm/md/lg） |
| `Separator` | `@/components/ui/separator` | 分割线（horizontal/vertical） |

### 组件使用示例

```tsx
// Badge 组件
<Badge variant="success">已完成</Badge>
<Badge variant="warning">处理中</Badge>
<Badge variant="error">失败</Badge>

// Alert 组件
<Alert variant="info">系统维护通知</Alert>
<Alert variant="success">操作成功</Alert>

// Card 组件
<Card padding="md">
  <h3>卡片标题</h3>
  <p>卡片内容</p>
</Card>

// Button 组件
<Button variant="primary">主要按钮</Button>
<Button variant="secondary">次要按钮</Button>
<Button variant="destructive">危险操作</Button>

// Dialog 组件
<Dialog>
  <DialogTrigger asChild>
    <Button>打开弹窗</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>标题</DialogTitle>
      <DialogDescription>描述文字</DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <Button variant="ghost">取消</Button>
      <Button>确认</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### 常用静态样式类

> 基于 Catalyst Design System 预设的常用样式组合

```typescript
// 导航项样式
const navItemStyles = {
  base: 'group flex gap-x-3 rounded-lg px-3 py-2 text-sm font-medium transition-colors duration-150',
  inactive: 'text-zinc-500 hover:bg-zinc-950/5 hover:text-zinc-900 dark:text-zinc-400 dark:hover:bg-white/5 dark:hover:text-white',
  active: 'bg-zinc-950/5 text-zinc-900 dark:bg-white/10 dark:text-white'
};

// 图标样式
const iconStyles = {
  base: 'size-5 shrink-0 transition-colors',
  inactive: 'text-zinc-400 group-hover:text-zinc-900 dark:text-zinc-500 dark:group-hover:text-white',
  active: 'text-zinc-900 dark:text-white'
};

// 卡片样式
const cardStyles = {
  base: 'rounded-2xl border bg-white dark:bg-zinc-900',
  default: 'border-zinc-950/5 shadow-sm dark:border-white/10',
  elevated: 'border-zinc-200/80 shadow-xl dark:border-white/15'
};

// 用户头像样式
const avatarStyles = 'size-8 rounded-full bg-zinc-50 object-cover ring-1 ring-zinc-950/10 dark:bg-zinc-800 dark:ring-white/10';

// 分割线样式
const dividerStyles = 'border-t border-zinc-950/5 dark:border-white/10';

// 下拉菜单样式
const dropdownStyles = 'rounded-2xl border border-zinc-200/80 bg-white shadow-xl dark:border-white/15 dark:bg-zinc-900';
```

---

## 📝 第七部分：Git Commit Message 规范

> 统一的提交信息格式，方便代码审查和版本追溯。

### 提交格式

```
<type>(<scope>): <subject>

[body]

[footer]
```

### Type 类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(sidebar): add collapsible navigation` |
| `fix` | 修复 Bug | `fix(layout): resolve scroll overflow issue` |
| `style` | 样式调整（不影响功能） | `style(button): update hover state colors` |
| `refactor` | 代码重构（无功能变化） | `refactor(store): simplify state logic` |
| `docs` | 文档更新 | `docs(readme): add installation guide` |
| `chore` | 构建/工具配置 | `chore(deps): upgrade tailwindcss to v4` |
| `test` | 测试相关 | `test(utils): add unit tests for cn function` |
| `perf` | 性能优化 | `perf(list): implement virtual scrolling` |

### Scope 范围（可选）

- `layout` - 布局组件
- `sidebar` - 侧边栏
- `router` - 路由相关
- `store` - 状态管理
- `ui` - UI 组件
- `styles` - 样式文件
- `config` - 配置文件
- `deps` - 依赖更新

### Subject 规则

- 使用英文，首字母小写
- 动词开头（add, fix, update, remove, refactor）
- 不超过 50 个字符
- 不以句号结尾

### 示例

```bash
# 新功能
feat(sidebar): add user profile dropdown menu

# 修复
fix(layout): resolve content area scroll issue on mobile

# 样式调整
style(card): update border radius to match design spec

# 重构
refactor(types): extract common interfaces to shared module

# 文档
docs(agents): add git commit message guidelines

# 依赖更新
chore(deps): upgrade @tanstack/react-router to v1.120.3

# 带 body 的详细提交
feat(dashboard): add statistics cards section

- Add StatCard component with positive/negative indicators
- Implement responsive grid layout
- Support dark mode styling
```

### AI 生成 Commit Message 提示词

```
请根据以下变更生成 Git commit message：

变更文件：[git diff 或文件列表]
变更内容：[描述你做了什么]

要求：
- 遵循 Conventional Commits 规范
- 使用英文
- type 从 feat/fix/style/refactor/docs/chore 中选择
- 包含适当的 scope
```

---

## ✅ 第八部分：代码生成检查清单

### TypeScript & React

- [ ] Props 类型已定义且导出
- [ ] 所有 `<button>` 有 `type` 属性
- [ ] 使用 `import type` 导入类型
- [ ] 无 `any`、`unknown`、`enum`
- [ ] Key 使用唯一标识符

### 样式规范

- [ ] 使用 `cn` 函数合并条件类名
- [ ] 优先使用 wrapper 组件（Button、Badge 等）
- [ ] 暗色模式使用透明度映射
- [ ] 遵循 Catalyst 色彩系统
- [ ] Tailwind 类名符合 Biome 排序规则

---

## 📚 规范文档索引

| 文档 | 用途 | 使用场景 |
|-----|------|----------|
| [AGENTS.md](./AGENTS.md) | AI 协作开发规范 | 代码生成、组件创建 |
| [doc/UI_DESIGN_SPEC.md](doc/UI_DESIGN_SPEC.md) | UI 设计规范 | 样式编写、设计系统 |
| [doc/UI_TROUBLESHOOTING.md](doc/UI_TROUBLESHOOTING.md) | UI 问题排查指南 | 界面问题诊断修复 |

---

---
> Source: [panmingxuan/react-admin-template](https://github.com/panmingxuan/react-admin-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
