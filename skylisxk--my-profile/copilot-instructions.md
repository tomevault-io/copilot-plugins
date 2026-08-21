## my-profile

> > 本文件为项目最高准则。所有代码变更必须遵守此宪法，不得偏离。

# 个人作品集网站 — 项目宪法

> 本文件为项目最高准则。所有代码变更必须遵守此宪法，不得偏离。

---

## 一、项目定位

- **类型**：个人作品集/Portfolio 单页网站
- **目标**：展示个人项目、技能、经历，建立专业在线形象
- **受众**：潜在雇主、合作者、技术社区

---

## 二、技术栈（不可变约束）

| 层级 | 技术 | 备注 |
|------|------|------|
| 结构 | HTML5 | 语义化标签，零 `<div>` 滥用 |
| 样式 | CSS3（原生） | 零预处理器（无 Sass/Less），使用 CSS 自定义属性 |
| 逻辑 | Vanilla JS（ES6+） | 零框架（无 React/Vue/jQuery），模块化组织 |
| 字体 | 系统字体栈 + 可选 Web Font | 优先系统字体以保证性能 |
| 图标 | SVG 内联 / Unicode | 不引入图标库 |
| 构建 | 无 | 零构建步骤，编辑器即开发环境 |
| 部署 | 任意静态托管 | Netlify / Vercel / GitHub Pages |

**铁律**：本项目永远不引入 npm 依赖、不安装构建工具、不添加框架。违反此条需重写项目。

---

## 三、目录结构

```
website/
├── index.html              # 唯一页面，所有板块在此
├── assets/
│   ├── css/
│   │   ├── reset.css       # CSS Reset（最小化）
│   │   ├── variables.css   # 设计 Token：颜色、字体、间距、阴影
│   │   ├── base.css        # 基础元素样式（body, h1-h6, p, a, img）
│   │   ├── layout.css      # 布局：导航、板块、页脚
│   │   ├── components.css  # 可复用组件（按钮、卡片、标签）
│   │   ├── animations.css  # 关键帧动画 & 过渡定义
│   │   └── responsive.css  # 响应式断点 & 覆盖
│   ├── js/
│   │   ├── main.js         # 入口：DOM 加载后初始化所有模块
│   │   ├── navigation.js   # 导航栏：滚动监听、菜单切换
│   │   ├── animations.js   # 滚动触发动画（Intersection Observer）
│   │   └── utils.js        # 纯函数工具（节流、防抖、选择器）
│   ├── images/             # 压缩后的图片（WebP 优先，保留 JPG 回退）
│   └── fonts/              # 自托管字体文件（如使用）
├── CLAUDE.md               # 本文件
└── .claude/
    ├── settings.local.json # 项目级 Claude 配置
    └── skills/             # 项目专属 Skills
```

---

## 四、编码规范

### HTML
- 必须使用语义化标签：`<header>` `<nav>` `<main>` `<section>` `<article>` `<footer>`
- `<div>` 仅用于纯布局容器，且有 `class` 说明用途
- 图片必须有 `alt` 属性，功能性图片用空 `alt=""`
- `<a>` 的外部链接加 `rel="noopener noreferrer"`，根据需要加 `target="_blank"`
- 表单元素必须关联 `<label>`

### CSS
- 命名采用 **BEM**：`.block__element--modifier`
- 所有颜色、字体、间距通过 CSS 变量引用，**禁止硬编码**
- 选择器优先级从低到高：BEM class → `[data-state]` → 不写 ID 选择器
- 动画仅作用于 `transform` 和 `opacity`（GPU 加速），避免 `width/height` 动画
- 响应式：移动优先，`min-width` 断点：`576px` `768px` `1024px` `1200px`

### JavaScript
- `data-*` 属性做 JS 行为钩子，**绝不**用 CSS class 选择器绑定行为
- 每个 JS 文件一个职责，通过 `main.js` 统一初始化
- 使用 `const`/`let`，禁用 `var`
- 异步操作用 `async/await`
- 事件监听统一在初始化函数中绑定，避免散落
- 工具函数必须**纯函数**，无副作用

### 注释
- 使用中文注释
- 每个 CSS 文件顶部注明用途
- 每个 JS 函数加 JSDoc 简要说明
- 复杂逻辑必须注释意图，而非复述代码

---

## 五、设计 Token

### 色彩系统（CSS 变量）

```css
:root {
  /* 主色调 */
  --color-primary: #6C63FF;        /* 主色 - 按钮、链接、强调 */
  --color-primary-hover: #5A52D5;
  --color-primary-muted: rgba(108, 99, 255, 0.1);

  /* 背景 */
  --color-bg-primary: #0F0F1A;     /* 主背景 */
  --color-bg-secondary: #1A1A2E;   /* 卡片、区块背景 */
  --color-bg-elevated: #252540;    /* 悬浮层 */

  /* 文字 */
  --color-text-primary: #EAEAEA;
  --color-text-secondary: #A0A0B0;
  --color-text-muted: #6B6B80;

  /* 边框 & 分割线 */
  --color-border: rgba(255, 255, 255, 0.08);
  --color-border-hover: rgba(255, 255, 255, 0.16);

  /* 状态色 */
  --color-success: #4ECDC4;
  --color-warning: #FFD93D;
  --color-error: #FF6B6B;
}
```

### 字体系统

```css
:root {
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', 'Consolas', monospace;
  --font-display: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;

  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 2rem;      /* 32px */
  --text-4xl: 2.5rem;    /* 40px */
  --text-5xl: 3.5rem;    /* 56px - Hero 标题 */
}
```

### 间距系统（4px 基准）

```css
:root {
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-24: 6rem;     /* 96px */
}
```

### 圆角 & 阴影

```css
:root {
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;
  --radius-full: 9999px;

  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.4);
  --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.5);
  --shadow-glow: 0 0 20px rgba(108, 99, 255, 0.3);
}
```

---

## 六、动效原则

| 类型 | 时长 | 缓动函数 | 用途 |
|------|------|----------|------|
| 微交互 | 150-250ms | `ease-out` | hover、点击反馈 |
| 过渡 | 300-400ms | `cubic-bezier(0.4, 0, 0.2, 1)` | 导航切换、展开收起 |
| 入场动画 | 500-800ms | `cubic-bezier(0.22, 0.61, 0.36, 1)` | 滚动进入视口 |
| 粒子/背景 | 持续 | `linear` | 装饰性背景动画 |

**规则**：
- 所有动画必须考虑 `prefers-reduced-motion` 媒体查询
- 入场动画由 Intersection Observer 触发，只播一次
- 不堆砌动画 —— 一个板块最多一个入场效果

---

## 七、性能指标

- 首屏加载 < 1.5s（3G 网络）
- 总页面大小 < 500KB（含图片）
- Lighthouse Performance ≥ 95
- 图片全部压缩，照片用 WebP（`<picture>` 提供 JPG 回退）
- CSS/JS 文件不合并不压缩（保持可读性），文件数量控制在 10 个以内

---

## 八、板块规划

首页按以下顺序排列（单页滚动）：

1. **Hero** — 名字、一句话介绍、CTA 按钮
2. **About** — 个人简介
3. **Skills** — 技能树/技术栈
4. **Projects** — 精选项目卡片
5. **Experience** — 经历时间线
6. **Contact** — 联系方式 & 表单

---

## 九、开发约定

- 修改网站内容时，优先改 HTML 结构，不要动 CSS/JS 文件
- 修改视觉风格时，优先改 `variables.css`，再改具体样式
- 新增板块时，遵循 `HTML → CSS → JS` 的顺序
- 每次改动后，用 `/preview` 在浏览器中确认效果
- 此宪法为活文档 —— 发现更好的实践时，更新本文件并注明原因

---
> Source: [skylisxk/My_Profile](https://github.com/skylisxk/My_Profile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
