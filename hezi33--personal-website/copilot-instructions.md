## personal-website

> > **核心理念**: 限制 AI 的自由度 = 保证输出质量。

# CLAUDE.md — AI 开发铁律

> **核心理念**: 限制 AI 的自由度 = 保证输出质量。
> **读我时机**: 修改此项目的任何文件之前，必须完整阅读本文档。
> **铁律性质**: 本文档列出的规则是硬约束，不允许以"我觉得这样更好"为理由违背。

---

## 1. 触发条件

以下任一情况发生时，AI 必须在第一轮对话中读取本文档：

- 用户要求修改 `index.html` 或项目中的任何 HTML/CSS/JS 文件
- 用户要求新增页面或 Section
- 用户要求调整样式、配色、字体、动效
- 用户说"继续开发"、"下一步"、"更新进度"
- 在此项目目录下启动的任何 AI 会话

---

## 2. 强制工作流（7 步）

每次开发任务必须按顺序执行，不得跳过：

```
Step 1: 读规范
   ├── 必读: esther-design-system-main/brand-dna.md
   ├── 按需: esther-design-system-main/references/scene-landing.md
   ├── 按需: esther-design-system-main/references/components.md
   └── 按需: esther-design-system-main/references/checklist.md

Step 2: 读数据
   └── 读取 DATA.yaml，获取当前个人信息

Step 3: 读进度
   └── 读取 docs/PROCESS.md，了解当前开发阶段

Step 4: 改代码
   ├── 基于 demo-landing.html 模板改造，不从零写
   ├── 每个 Section 布局必须不同（参考 layouts.md）
   ├── 所有组件从 components.md 选用，禁止 HTML 默认样式
   └── 更新 DATA.yaml（如涉及数据变更）

Step 5: 跑 checklist
   ├── 对照 esther-design-system-main/references/checklist.md
   ├── P0 必须全过
   └── 任何 P0 不过 → 修改 → 重跑

Step 6: 交付
   └── 确保 index.html 浏览器直接打开可用

Step 7: 更新进度
   └── 更新 docs/PROCESS.md，记录本次开发内容
```

---

## 3. 设计铁律（硬约束）

### 3.1 配色（不商量）

```
蓝 #2B7FD8 : 主色调 ~60%（标题、链接、CTA按钮、Section编号）
黄 #F4D758 : 强调 ~30%（高亮标记、装饰、::selection、引用框边框）
红 #E84A5F : 点缀 ~10%（特别标签、高亮下划线）
────────────────────────────────────────
奶油 #fefcf6 : 主背景（绝不出现纯白 #fff）
深奶油 #faf6eb : 交替 Section 背景
墨色 #1A1A2E : 正文文字（绝不出现纯黑 #000）
深色面板 #151821 : 金句/引用区域
```

### 3.2 字体（不商量）

```
中文标题:  'Noto Serif SC' (font-weight: 900)
中文正文:  'Noto Sans SC' (font-weight: 400/500/700)
英文装饰:  'Fraunces' (italic)
代码/标签: 'Fira Code' (font-weight: 400/500)
手写/注释: 'Caveat' (font-weight: 400/700)
```

### 3.3 布局（不商量）

- **每个 Section 布局必须不同** — 不允许两个相邻 Section 用相同的布局模式
- **推荐节奏**: Hero → 三列卡片 → 深色全宽面板 → 双列亮点 → Pull Quote → CTA
- **内容宽度**: `max-width: 1300px` + `margin: 0 auto` + `padding: 0 2rem`
- **Section 间距**: `padding: clamp(80px, 12vh, 160px) 0`
- **卡片圆角**: `border-radius: 20px`（大卡片）或 `16px`（小卡片）

### 3.4 间距（不商量）

```
全部使用 clamp()，禁止固定 px:
  Section 间距: clamp(80px, 12vh, 160px)
  内容块间距:  clamp(40px, 6vw, 100px)
  卡片内边距:  clamp(28px, 3vw, 40px)
  元素间 gap:  clamp(24px, 3vw, 48px)
```

---

## 4. 组件铁律（不商量）

### 4.1 绝对禁止

| 禁止                            | 原因                              |
| ------------------------------- | --------------------------------- |
| HTML 默认 `<blockquote>`      | 浏览器默认缩进样式千篇一律        |
| `border-left` 引用块          | 过于常见，AI 模板感               |
| 无样式 `<ul>` / `<ol>`      | 必须用 components.md 里的列表组件 |
| 默认 `<table>`                | 必须用组件库里的表格样式          |
| 默认 `<button>`               | 必须使用 `.cta-button` 样式     |
| 纯黑 `#000` 或纯白 `#fff`   | 无温度感                          |
| `box-shadow` 千篇一律的卡片   | 卡片必须有区分度                  |
| glassmorphism（毛玻璃仅限 Nav） | AI 模板感第一来源                 |

### 4.2 必须使用

| 场景      | 必须使用                                                            |
| --------- | ------------------------------------------------------------------- |
| 引用/金句 | `components.md` 的 Pull Quote 组件                                |
| 卡片      | `components.md` 的卡片组件（杂志裁切/编号主导/侧边Icon）          |
| 列表      | `components.md` 的列表组件                                        |
| CTA 按钮  | `.cta-button`（蓝底白字）或 `.cta-button--yellow`（深色背景上） |

---

## 5. 动效铁律

### 5.1 允许的动效

- ✅ Scroll Reveal（IntersectionObserver + opacity + translateY）
- ✅ Stagger 延迟（`.reveal-d1` ~ `.reveal-d5`，间隔 0.1s）
- ✅ Nav 滚动阴影切换
- ✅ `:hover` 的 `transform: translateY(-2px)` + `box-shadow` 扩展
- ✅ CTA 按钮悬停效果
- ✅ `transition` 使用 `cubic-bezier(0.16, 1, 0.3, 1)` 缓动

### 5.2 禁止的动效

- ❌ `bounce` / `elastic` 缓动
- ❌ `@keyframes` 无限循环（除轨道旋转外）
- ❌ `transition: all`（必须指定属性）
- ❌ `animate width/height`（触发 layout/reflow）

---

## 6. 代码规范

### 6.1 HTML

- 语义化标签：`<nav>` `<section>` `<footer>` `<h1>`-`<h4>`
- 图片必须有 `alt` 属性
- 缩进使用 Tab（与模板一致）
- Section 之间用注释分隔：`<!-- ====== SECTION NAME ====== -->`

### 6.2 CSS

- 所有 CSS 内联于 `<style>` 标签（V1 阶段不拆文件）
- CSS 变量统一在 `:root` 声明
- 样式按页面 Section 顺序组织
- 移动端媒体查询放在最后
- 选择器优先级保持扁平（不超过 2 层嵌套）

### 6.3 JavaScript

- 仅使用原生 ES6+ API
- IntersectionObserver 用于 Scroll Reveal
- scroll 事件监听用于 Nav 效果
- 所有 JS 内联于 `<script>` 标签末尾
- 不引入任何第三方 JS 库

---

## 7. 响应式铁律

```
断点: 900px（多列→单列）、600px（字号微缩）
原则: Adapt, don't amputate — 移动端重新排列，不隐藏内容
必须: @media (prefers-reduced-motion: reduce) 尊重无障碍
```

---

## 8. 犯错修正机制

### 8.1 当 AI 犯错时

```
1. 用户指出错误 → AI 立即修复
2. AI 分析根因 → 为什么犯了这个错误？
3. AI 在本文档新增一条规则 → 防止同类错误再次发生
4. AI 在 docs/PROCESS.md 记录本次错误及修正
```

### 8.2 犯错记录格式

```markdown
### 错误记录 [日期]

- **错误**: [描述]
- **根因**: [为什么犯]
- **修正**: [怎么改的]
- **新增规则**: [CLAUDE.md 新增了什么铁律]
```

---

## 9. 参考文件速查

| 文件     | 路径                                                      | 读我时机             |
| -------- | --------------------------------------------------------- | -------------------- |
| 品牌基因 | `esther-design-system-main/brand-dna.md`                | 每次必读             |
| 场景规范 | `esther-design-system-main/references/scene-landing.md` | 做 Section 布局前    |
| 组件库   | `esther-design-system-main/references/components.md`    | 选组件前             |
| 布局库   | `esther-design-system-main/references/layouts.md`       | 选布局前             |
| 检查清单 | `esther-design-system-main/references/checklist.md`     | 交付前               |
| 模板文件 | `esther-design-system-main/demo-landing.html`           | 写代码前（起点模板） |
| 数据文档 | `DATA.yaml`                                             | 需要个人信息时       |
| 进度文档 | `docs/PROCESS.md`                                       | 每次开始/结束任务时  |
| 产品需求 | `docs/PRD.md`                                           | 了解功能边界时       |
| 技术方案 | `docs/TECHNICAL_DESIGN.md`                              | 做技术决策时         |

---

## 10. 核心底线（刻在脑子里）

> **截图发到 Twitter 上，不能被人评论"又是 AI 做的"。**
> **温暖、有品质、像人做的、不像模板。**

---
> Source: [hezi33/personal-website](https://github.com/hezi33/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
