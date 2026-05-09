## claude-101

> 交互式学习网站：以 AI 的第一人称视角理解 Claude Code 的工作原理。

# Claude 101

交互式学习网站：以 AI 的第一人称视角理解 Claude Code 的工作原理。

## 技术栈

- **框架**: Astro (岛屿架构)
- **交互组件**: React (按需加载, `client:visible` / `client:idle`)
- **内容**: MDX (章节页面在 `src/pages/chapters/`)
- **样式**: Tailwind CSS v4 + CSS 变量 (暗色主题)
- **动画**: Framer Motion + GSAP
- **语言**: TypeScript (strict)

## 项目结构

```
src/
  config/chapters.ts     # 章节配置数据
  layouts/
    Layout.astro         # 基础布局
    ChapterLayout.astro  # 章节页面布局 (侧边栏 + 导航)
  components/
    Sidebar.astro        # 桌面端侧边栏导航
    MobileNav.astro      # 移动端导航
    ChapterNav.astro     # 上一章/下一章导航
    interactive/         # React 交互组件 (岛屿)
  pages/
    index.astro          # 首页
    chapters/*.mdx       # 各章节内容
  styles/
    global.css           # 全局样式 + Tailwind + 主题变量
    interactive.css      # 交互组件样式
```

## 开发命令

```bash
npm run dev     # 启动开发服务器
npm run build   # 构建生产版本
npm run preview # 预览构建结果
```

## 约定

- 章节页面使用 MDX，layout 指向 `ChapterLayout.astro`
- React 交互组件放在 `src/components/interactive/`，使用 `client:visible` 按需加载
- 组件样式优先使用 CSS 变量 (定义在 global.css 的 `@theme` 中)
- 中文为主要语言，`lang="zh-CN"`
- 暗色主题为默认主题

---
> Source: [airingursb/claude-101](https://github.com/airingursb/claude-101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
