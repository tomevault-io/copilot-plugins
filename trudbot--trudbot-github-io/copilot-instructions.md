## trudbot-github-io

> - **框架**: [Next.js 16](https://nextjs.org/) (App Router)

# 项目技能与架构文档

## 1. 技术栈概览
- **框架**: [Next.js 16](https://nextjs.org/) (App Router)
- **构建工具**: Turbopack (开发时使用 `--turbo`)
- **语言**: [TypeScript](https://www.typescriptlang.org/)
- **UI 库**: [React 19](https://react.dev/)
- **样式**: [Tailwind CSS v4](https://tailwindcss.com/)
- **动画**: [Framer Motion](https://www.framer.com/motion/) & `tw-animate-css`
- **组件基础**: [Radix UI](https://www.radix-ui.com/) (通过 Shadcn UI)
- **图标**: [Lucide React](https://lucide.dev/)
- **工具库**: `clcx`、`tailwind-merge`、`date-fns`

## 2. 设计系统

### 色彩体系
但满足设计要求的情况下， 优先取用app/colors/page.tsx中存在的颜色

### 字体排版
- **无衬线**: Geist Sans（简洁、现代的系统字体）
- **等宽**: Geist Mono（用于代码/数据）
- **手写体**: Caveat（用于活泼、个性化的点缀）— 通过 `--font-handwriting` 配置

### 视觉风格
- **核心理念**:
    - **艺术与个性**: 带有强烈的个人特征，带有艺术气息。
    - **几何感**: 注重几何图形与构图。
    - **反套路**: 避免通用设计模式（标准卡片、混浊渐变、莫名阴影）。
    - **童趣感**: 清新、明亮的色彩，像孩子的简笔画一样 — 干净、天真、鲜艳。

## 3. 项目架构

### 目录结构
```
app/
├── (home)/          # 路由分组：首页
├── json/            # 功能：JSON 编辑器与格式化工具
├── share/           # 功能：分享功能
├── layout.tsx       # 根布局，包含 ThemeProvider 和全局样式
└── globals.css      # Tailwind v4 配置与 CSS 变量
components/
├── ui/              # 原子设计组件（Button、Card、Input...）
├── theme-provider.tsx # Next-themes 集成
└── [feature].tsx    # 功能相关组件
lib/
└── utils.ts         # CN 辅助函数 (clsx + tailwind-merge)
```

### 关键架构决策
1.  **原子设计**: UI 组件抽离到 `components/ui` 中，最大化复用性。
2.  **路由分组**: 使用括号文件夹（如 `(home)`）组织路由，不影响 URL 结构。
3.  **客户端/服务端分离**: 交互组件（如 JSON 编辑器）显式使用 `"use client"`，默认保持为服务端组件以提升性能。

## 4. 已采纳的最佳实践

- **严格 TypeScript**: 在整个应用中确保类型安全。
- **Tailwind v4**: 采用最新的 CSS 优先配置方式（大多数情况下不需要 `tailwind.config.js`）。
- **无障碍 (a11y)**: 基于 Radix UI 基础组件，确保键盘导航和屏幕阅读器支持。
- **性能**:
    - 使用 `next/dynamic` 延迟加载重型客户端库（如 `react-json-view`）。
    - 使用 Turbopack 实现即时开发反馈。
- **代码质量**:
    - 预配置 ESLint（未来可能需要 v9 兼容配置）。
    - 关注点分离（Hooks、Components、Utilities）。
- **构建优化**
    - 充分考虑代码对首屏加载速度、页面性能的影响。 代码修改或方案设计会影响性能时需要主动让用户决策。
    - 充分利用next的ssg机制， 尽量使页面不依赖js也能可读

## 5. 功能亮点

### 主题系统
- 基于 `next-themes` 构建。
- CSS 变量定义在 `globals.css` 的 `:root` 和 `.dark` 选择器中，实现即时、无闪烁的主题切换。

### 路由
[ROUTING_GUIDE.md](./ROUTING_GUIDE.md)

### 页面
[PAGES](./PAGES.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trudbot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/trudbot)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
