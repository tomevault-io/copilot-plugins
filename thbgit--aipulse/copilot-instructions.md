## aipulse

> AIPulse 是一个 AI 工具新闻资讯网站，提供大模型、图像生成、编程助手、效率工具和行业动态等分类的新闻资讯浏览。采用编辑式排版风格，深靛蓝主色 + 亮青强调色的配色方案。

# AGENTS.md

## 项目概览

AIPulse 是一个 AI 工具新闻资讯网站，提供大模型、图像生成、编程助手、效率工具和行业动态等分类的新闻资讯浏览。采用编辑式排版风格，深靛蓝主色 + 亮青强调色的配色方案。

## 版本技术栈

- **Framework**: Next.js 16 (App Router)
- **Core**: React 19
- **Language**: TypeScript 5
- **UI 组件**: shadcn/ui (基于 Radix UI)
- **Styling**: Tailwind CSS 4

## 目录结构

```
├── public/                    # 静态资源
├── src/
│   ├── app/
│   │   ├── globals.css        # 全局样式与主题变量（含 brand 自定义色）
│   │   ├── layout.tsx         # 根布局（ThemeProvider + Header + Footer）
│   │   ├── page.tsx           # 首页（客户端组件，含分类筛选）
│   │   ├── not-found.tsx      # 404 页面
│   │   └── article/[id]/
│   │       └── page.tsx       # 文章详情页（服务端组件）
│   ├── components/
│   │   ├── ui/                # shadcn/ui 组件库
│   │   ├── header.tsx         # 顶栏导航（搜索/暗色切换/移动端菜单）
│   │   ├── footer.tsx         # 页脚
│   │   ├── hero-section.tsx   # 首页 Hero 精选区（1 主 + 2 副头条）
│   │   ├── news-card.tsx      # 新闻卡片（default/compact 两种变体）
│   │   ├── category-tabs.tsx  # 分类标签切换栏
│   │   ├── sidebar.tsx        # 侧栏（热门/标签/订阅）
│   │   ├── theme-provider.tsx # 暗色模式 Context Provider
│   │   └── theme-toggle.tsx   # 暗色切换按钮
│   └── lib/
│       ├── data.ts            # 模拟数据层（articles + 查询函数）
│       └── utils.ts           # 通用工具函数 (cn)
├── DESIGN.md                  # 设计规范
└── package.json
```

## 构建与测试命令

- 开发启动: `pnpm run dev`
- 类型检查: `pnpm run ts-check`
- Lint: `pnpm run lint`
- 构建: `pnpm run build`
- 生产启动: `pnpm run start`

## 代码风格指南

- 严格 TypeScript，禁止隐式 `any` 和 `as any`
- 使用 shadcn/ui 组件和风格规范
- 自定义颜色通过 CSS 变量 `--brand` / `--brand-foreground` 定义，Tailwind 中使用 `text-brand` / `bg-brand` 等类名
- 客户端交互组件需添加 `'use client'` 指令
- 页面级组件优先使用服务端组件，交互逻辑下沉到子组件

## 数据层说明

- `src/lib/data.ts` 包含 12 篇模拟文章数据和查询函数
- 分类枚举: `Category = 'all' | 'llm' | 'image-gen' | 'coding' | 'productivity' | 'industry'`
- 主要函数: `getArticleById`, `getArticlesByCategory`, `getFeaturedArticles`, `getTrendingArticles`, `getRelatedArticles`

## 主题系统

- 通过 `ThemeProvider` + `useTheme` Hook 管理暗色模式
- 主题存储在 `localStorage('theme')`
- CSS 变量定义在 `globals.css` 的 `:root` 和 `.dark` 选择器中
- 品牌/强调色使用独立 `--brand` / `--brand-foreground` 变量

## 常见问题修复

- Hydration 错误: 避免在服务端渲染中使用 `typeof window`、`Date.now()` 等，需用 `useEffect` + `useState` 确保仅客户端执行
- ThemeProvider context 丢失: 确保 `useTheme` 仅在 ThemeProvider 子树内使用，且 Provider 始终包裹 children（不要在未挂载时跳过 Provider）

---
> Source: [thbgit/aipulse](https://github.com/thbgit/aipulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
