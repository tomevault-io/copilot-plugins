## prompt-craft

> Prompt Craft v4 — 三层 Prompt 架构 + 3 引擎动画库 + 6 大设计方向。从用户一句需求 → 到设计方向 → 到技术栈 → 到可运行代码，一步到位。


# 🚀 Prompt Craft — 网页生成器 v3.0

> **一句话定位**：把"我想做一个官网"这句话，自动翻译为「设计方向 × 技术档位 × 交互签名 × 可运行代码」的完整产出。
>
> **本文件的阅读者**：人类用户（看前 3 节，1 分钟掌握怎么用）+ AI（读完整份，按第 6 节的流程执行）。

---

## 📌 一句话怎么用（给人类用户的 3 句话）

你只需要按下面的句式说话，剩下的全部交给 Prompt Craft：

| 你想要的东西 | 你可以这么说 | 默认档位 |
|-------------|--------------|---------|
| 一个公司/品牌官网，要有设计感 | `做一个 [你的行业] 公司官网` | Editorial 杂志风 + 🚀 快速原型档 |
| 一个炫酷的沉浸式官网，像 Awwwards 那样 | `做一个 [你的行业] 官网，要很酷，像 Awwwards 那样` | Cyberpunk 沉浸风 + 🚀 快速原型档 |
| 一个 3D 产品展示页，像小米 SU7 那样 | `做一个 [产品] 的 3D 展示页面，可以旋转查看` | 3D Product Showcase + Three.js |
| 一个数据可视化大屏，展示 KPI / 实时数据 | `做一个 [主题] 的数据可视化大屏` | 深色科技风 + 🚀 快速原型档 |
| 一个后台管理系统，有表格 / 表单 / 权限 | `做一个 [主题] 的后台管理系统` | 浅色 + Ant Design / Element Plus |
| 一个 API / 后端服务 | `做一个 [业务] 的 REST / GraphQL API` | Node.js + Express + Prisma |
| 我已有一段代码出问题 | `这段代码报错了：[贴错误信息]` | Debug 流程 |

**如果你什么都没指定，我们会用这个默认组合**（你可以随时要求替换）：

- 设计方向：**A · Editorial 杂志博物馆风**（奶油纸色 + Serif 大标题 + 巨量留白）
- 技术档位：**🚀 快速原型档**（单文件 HTML + ES Modules CDN，双击打开即运行）
- 交互签名：从 20 个签名库里默认挑 5 个最常用的 — **遮罩文字入场 / 自定义光标+磁吸按钮 / hover 反色卡片 / Shader Orb / 滚动下划线生长**

想改动？可以说：`改成 Cyberpunk 风格`、`升级为 Vue 3 生产级档`、`用横向滚动代替垂直滚动`、`强调色换成 #2D7CFF`。

---

## 🏛 系统架构（三层 Prompt）

```
                                 用户一句话需求
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
        T1 · 项目规则           T2 · 专家角色           T3 · 任务模板
 （这是"法律法规"，必遵守）  （这是"设计师/工程师"人设）  （这是"施工蓝图"，按步执行）
                              ┌───────────────────────┐
                              │  组合后的完整 Prompt    │
                              │  = T1 + T2 + T3 + 用户需求 │
                              └───────────┬─────────────┘
                                          ▼
                                   可运行代码输出
```

### 每一层解决的问题

| 层级 | 文件来源 | 解决的问题 | 本页自动注入的默认值 |
|------|---------|-----------|-------------------|
| **T1 项目级规则** | `project-rules/*.md` | 技术栈硬约束 — 用什么框架、什么字体、颜色数量、性能预算、哪些第三方库被禁止。 | `artistic-stack.md`（官网类请求）/ 根据需求类型切换 |
| **T2 专家角色** | `expert-personas/*.md` | AI 的"思维人格"。官网类走 *Creative Technologist*（克制、叙事感、滚动故事），常规前端走 *Senior Frontend Engineer*，后端走 *Senior Backend Engineer*。 | 官网类 → Creative Technologist |
| **T3 任务模板** | `task-templates/*.md` | 每类任务的 **CO-STAR** 框架：Context、Objective、Style、Tone、Audience、Response Format、Constraints。这是"AI 的施工步骤"。 | 官网类 → `build-artistic-website.md` |

**执行原则（AI 必须遵守）**：永远不要让用户去看某个 md 文件再回来。所有规则的"精华"，你（AI）要在这里当场消化完，直接输出代码。

---

## 🎨 网页生成器 · 设计 DNA 库

> 详细清单请见 `references/artistic-website-dna.md`（约 280 行）。下面是给用户看的"选单" + 给 AI 看的"速查表"。

### 6 个设计方向（选 1 个作为页面主 DNA）

| 方向 | 色彩（一组默认值） | 字体（标题 + 正文） | 布局关键词 | 典型交互 | 适合行业 |
|-----|-------------------|-------------------|----------|---------|---------|
| **A · Editorial 杂志博物馆风** | 奶油纸 `#F5F3EF` + 墨黑 `#1A1A1A` + 赭石 `#E0512B`（3 色原则） | **Display** Newsreader / Playfair（Serif，巨号，行高 ≥ 1.1）<br>**Body** Inter，栏宽 50-65 字符 | 不对称双栏、巨量留白 14-20vh、section 间用巨型数字分隔 | 遮罩文字入场、纸张噪点、hover 反色卡片、滚动下划线生长、长文编辑滚动 | 设计工作室 / 文化机构 / 高端品牌 / 创意咨询 / 金融科技 / 出版社 |
| **B · Cyberpunk 黑暗沉浸式** | 深黑 `#0A0A0F` + 近白文本 + 洋红 `#FF2E97` + 青 `#00F0FF`（双霓虹） | **Display** Condensed / 窄体几何 sans，全大写宽字距<br>**Body** 小号 sans，数字用等宽字体 | 画布优先（WebGL 占满屏），HTML UI 以 HUD 形式浮在 canvas 上 | Shader Orb、全屏 Canvas 2D、粒子场、滚动驱动、HUD 状态面板、实时时钟 ticker | 游戏工作室 / 互动媒体 / 音乐厂牌 / 科幻 IP / 科技产品发布页 / AI 研究实验室 |
| **C · Playful 游戏探索风** | 蓝天绿草橙砖 + 单个签名色（如法拉利红）作交互元素 | **Display** 圆润几何 sans<br>数字用等宽<br>所有标签文字"钉"在 3D 世界标牌上而非 HTML | 没有传统布局，页面是一张 3D 世界地图，区域间用物理导航 | 3D 世界地图、碰撞 / 物理、实体 Tilt Card、横向滚动章节、用户"驾驶"到达内容 | 个人作品集（技术背景）/ 游戏公司 / 儿童教育 / 创意工具发布 |
| **D · 3D Product Showcase** | 深黑 / 纯白背景 + 产品本身材质色 + 1 个 UI 强调色（电蓝 / 霓虹橙） | **Display** 极细 Sans（Inter 300）+ 等宽字体用于规格数字 | 画布优先——WebGL Canvas 占满屏，HTML UI 以 HUD 浮层叠加 | 360° 产品旋转、颜色/材质切换、热点标注、环境光切换、规格表 | 汽车 / 消费电子 / 家具 / 珠宝 / 工业设备 / 高端产品展示 |
| **E · Neo-Swiss 新瑞士粗野主义** | 瑞士米灰 `#E8E6DF` 背景 + 纯墨 `#0A0A0A` + 瑞士红 `#E11D48` + 瑞士蓝 `#1E40AF` | **Display** Space Grotesk（几何无衬线 500-900）<br>**Body** Inter + JetBrains Mono（数字/代码）| 12 列可见网格线、大字号 border 作为结构、news ticker 横向滚动、编号系统 | 反色 manifesto、项目编号列表 (001/002 →)、实时时钟 ticker、hover 时整行位移 | 设计工作室 / 建筑事务所 / 法律金融 / 出版社 / 学术机构 / 严肃科技品牌 |
| **F · Dark Luxury 深色奢华风** | 深墨 `#0A0A0C` + 象牙白 `#E8E6E1` + 暖金 `#C9A962` + 青铜 `#8B7355` | **Display** Inter 200-300 极细（超大型）+ 金色渐变强调<br>**Body** Inter 300<br>**Specs** JetBrains Mono | 产品占 60-80% 屏幕空间，文字信息以 HUD 形式浮动；黄金分割；大量"暗色空间" | 产品 3D 倾斜展示、金色渐变文字、规格表、青铜色次强调、canvas 2D 金色粒子 | 高端手表 / 珠宝 / 香水 / 奢侈品零售 / 私人银行 / 限量版科技产品 / 高端汽车 |

### 15 个交互签名（"菜单" — 每次选 3-5 个）

> 编号 IS-01 到 IS-15，完整技术细节（参数、ease、参考代码骨架）在 `references/artistic-website-dna.md`。
> 一页内不要超过 **5 个**签名；否则页面变成"游乐场"，用户无法聚焦。

| 编号 | 签名名称（英文） | 一句话效果 | 适用场景 | 推荐方向 |
|-----|----------------|----------|---------|--------|
| IS-01 | Mask Reveal | 文字被 clip-path 遮罩，从下至上"打开"出现 | Hero 大标题、section 标题 | Editorial / Cyberpunk |
| IS-02 | SplitText Stagger | 标题拆成单字符，逐字入场（stagger 0.02-0.05s） | Hero、manifesto 宣言 | Editorial |
| IS-03 | Scroll-driven Line Grow | section 标题下划线随滚动从 0 "生长"到目标宽度 | 所有 section 的小标题标签 | 所有方向通用 |
| IS-04 | Custom Cursor + Magnetic Button | 隐藏原生光标，用圆环跟随；hover 按钮时圆环变文字框并"吸附" | 全站光标系统 + 所有 CTA | 所有方向通用 |
| IS-05 | Hover Invert | hover 卡片时整卡从白底变黑底，文字变白，箭头旋转 | Features 网格卡片、项目卡片 | Editorial |
| IS-06 | Scroll-driven Shader | hero 的 shader 背景随滚动进度演化，球体变形，色彩偏移 | Hero 背景、章节过渡 | Cyberpunk |
| IS-07 | Horizontal Scroll | 纵向滚动中横向移动的长条（ScrollTrigger pin） | 项目展示、时间轴 | Playful / Editorial |
| IS-08 | Sticky Reveal Narrative | section 粘在视口顶部，滚动时内部内容逐步变化（类似幻灯片） | 故事叙事、产品介绍、历史时间线 | Editorial / Brand story |
| IS-09 | Shader Orb | Hero 右侧 / 中心一个缓慢浮动+变形的 3D 球体，受鼠标轻微推动（fresnel rim light） | Hero 装饰、品牌特征展示 | Cyberpunk / Editorial |
| IS-10 | Canvas 2D Particle Field | 100-300 个粒子在 hero 中漂浮，hover 时粒子向鼠标聚集 | Hero 背景氛围 | Cyberpunk |
| IS-11 | SVG Noise Paper | 全站 2-4% opacity 的 SVG feTurbulence 噪点，模拟纸张/印刷品 | editorial 方向的所有页面 | Editorial（必选） |
| IS-12 | Tilt 3D Card | 鼠标 hover 卡片时，卡片绕 X/Y 轴做 3D perspective 倾斜，内容"浮"起 | Feature cards、产品卡片、案例缩略图 | 所有方向通用 |
| IS-13 | Page Wipe | 路由切换时，一整屏色块从右向左"擦"过页面，在下一页反向退回 | 多页网站、SPA 路由切换 | Cyberpunk / Editorial（高级版） |
| IS-14 | Long-form Editor Scroll | 图文混排长文，滚动时图像从左/右滑入，文字从下方淡入，像翻杂志 | 品牌故事、案例研究、产品介绍 | Editorial |
| IS-15 | Fixed Background Progress | 固定在屏幕上的 SVG/Canvas 图形，其参数（大小、颜色、噪声密度）随滚动进度 0→1 变化 | 整页背景主题元素、品牌 logo 变形 | Cyberpunk |

**每个方向推荐的 5 个签名组合（默认使用，AI 在用户无偏好时照这套）**：

- **Editorial 默认**：IS-01 · IS-04 · IS-05 · IS-11 · IS-03
- **Cyberpunk 默认**：IS-04 · IS-09 · IS-10 · IS-06 · IS-15
- **Playful 默认**：IS-07 · IS-12 · IS-08 · IS-04 · IS-10

---

## 📐 技术栈预设（给 AI 看的硬约束）

> 详细代码骨架 / 启动代码 / 文件结构请见 `project-rules/artistic-stack.md`。
> 这里是"AI 在生成代码前必须背下来"的规则清单。

### 1. 两个档位（用户未指定时默认 🚀 快速原型档）

| 档位 | 架构模式 | 文件组织 | 适用阶段 |
|-----|---------|---------|---------|
| 🚀 **快速原型档** | 单文件 HTML + ES Modules **CDN**（`esm.sh` / `unpkg`）+ Google Fonts `<link>`，**零 npm install** | 全部写在一个 `index.html`：`<head>` 内联 `<style>` + `<body>` 末尾 `<script type="module">` | 验证想法、给客户看感觉、学习用途 |
| 🏭 **生产级档** | Vue 3 / Next.js + TypeScript + Vite，模块化 CSS / JS / 组件 | 分离为 `styles/`、`js/animations/`、`js/webgl/`、`components/`、`sections/` | 真实项目、长期维护、需 CMS / SSR / CI |

### 2. 动画引擎（三选一，按方向智能匹配）

| 引擎 | 优势 | CDN Import | 适合方向 |
|-----|------|-----------|----------|
| 🏅 **GSAP 3.x + ScrollTrigger** | 精细时间线、复杂滚动叙事、Awwwards 级 | `import { gsap } from 'https://esm.sh/gsap'` + `import { ScrollTrigger } from 'https://esm.sh/gsap/ScrollTrigger'` | Editorial / Cyberpunk / 3D Product / Swiss |
| 🟢 **Motion.dev (Motion One)** | 最小体积、硬件加速、简洁 API、spring 物理 | `import { animate, timeline, stagger, spring, inView } from 'https://esm.sh/motion'` | Modern SaaS / 产品站 / Editorial |
| 🟣 **anime.js v4** | SVG 路径动画、数据可视化、轻量微交互 | `import anime from 'https://esm.sh/animejs'` | Publishing / SVG-heavy / 数据可视化 |

### 3. 强制项（两个档位都必须满足）

- ✅ **Lenis 平滑滚动**：`new Lenis({ duration: 1.2, easing })` + `lenis.on('scroll', ScrollTrigger.update)`（与 GSAP/Motion/anime.js 均兼容）
- ✅ **2-3 色系统**：CSS Variables `--bg`、`--text`、`--accent`（可加 `--muted` 作为第 4 色）。永远不超过 4 色
- ✅ **clamp 响应式字号**：所有标题使用 `clamp(min, vw, max)`，禁止纯 px 标题
- ✅ **Google Fonts + preconnect**：在 `<head>` 里 `preconnect fonts.googleapis.com` + `preconnect fonts.gstatic.com crossorigin` + `font-display: swap`
- ✅ **字体加载后再启动动画**：`document.fonts.ready.then(() => initHero())`，避免 CLS 导致动画跳帧
- ✅ **语义化 HTML**：`<header> <main> <section> <footer> <nav>`；装饰性 canvas 使用 `aria-hidden`
- ✅ **排版安全**：Display 标题 `line-height ≥ 1.05`，正文 `≥ 1.5`，中文 `≥ 1.6`；禁止 `overflow: hidden` 与 `line-height < 1.1` 组合

### 4. 约束项（性能与可访问性降级）

- ⚠️ **prefers-reduced-motion 降级**：检测到 `(prefers-reduced-motion: reduce)` 时，所有 shaders 关闭、所有 ScrollTrigger scrub 改为瞬时 `opacity: 0→1`、所有 stagger 改为瞬时入场
- ⚠️ **mobile 768px 降级**：在 `@media (max-width: 768px)` 中：关闭自定义光标、关闭 magnetic button、关闭 WebGL shader（降级为静态 SVG）、关闭 heavy scrub、改为简单 fade-in
- ⚠️ **WebGL 预算**：桌面端最多 2 个 active canvas；移动端最多 1 个（且带 fail fallback）
- ⚠️ **颜色对比度**：正文 ≥ 7:1（AAA），按钮文本 ≥ 4.5:1（AA）
- ⚠️ **GSAP 约束**：`scrub` 必须用数字（`0.6`–`0.8`）；永远 `markers: false`
- ⚠️ **Motion.dev 约束**：`easing` 使用 `[0.22, 0.03, 0.26, 1]`（标准 cubic），`spring()` 用于 UI 微交互，不要用 spring 做入场动画
- ⚠️ **禁止第三方 cookies / analytics / ads**：除非用户明确要求，否则不要加任何外部分析脚本

### 4. 字体与字号速查表（AI 直接查表使用，不要自己瞎编）

| 元素 | Editorial | Cyberpunk | Playful | 通用规则 |
|-----|-----------|-----------|---------|---------|
| Hero 标题 | Newsreader / Playfair，`clamp(3rem, 10vw, 9rem)`，行高 1.1，字距 -0.02em，weight 300 | Condensed Sans / Space Grotesk Bold，`clamp(4rem, 14vw, 10rem)`，全大写宽字距 0.1em | 圆润几何 sans（如 Instrument Sans），`clamp(3rem, 9vw, 8rem)` | **永远用 clamp + vw** |
| Section 标题 | Newsreader，`clamp(2rem, 5vw, 4.5rem)`，weight 400 | 同上 display sans，`clamp(2.5rem, 6vw, 5rem)` | 同上圆润 sans | 同上 |
| 正文 | Inter / `clamp(0.95rem, 1.2vw, 1.1rem)` / 行高 1.65 | Inter / `0.95rem` / 行高 1.6 | Inter / `1rem` / 行高 1.55 | 栏宽 50-65 字符 |
| Small caps 标签 | Inter / `0.75rem` / uppercase / letter-spacing `0.2em` | Inter / `0.7rem` / uppercase / letter-spacing `0.25em` | Inter / `0.8rem` / uppercase / letter-spacing `0.15em` | 用于 section 编号、日期 |

---

## 🧩 可用的任务模板（完整列表）

> AI 在判断完"用户要做什么"之后，从下表选择一个 T3 模板按 CO-STAR 流程执行。

| 模板文件 | 适用的用户原话示例 | 推荐 T2 角色 | 推荐 T1 规则 |
|---------|-------------------|-------------|-------------|
| `task-templates/build-artistic-website.md` | "做一个设计工作室官网"、"做一个品牌官网，要像 Awwwards 那样"、"帮我做一个很酷的首页" | Creative Technologist | artistic-stack.md |
| `task-templates/build-dashboard.md` | "做一个销售数据大屏"、"做一个实时监控大屏"、"做一个运营数据看板" | Senior Frontend Engineer | vite-vue3-typescript.md 或 nextjs-shadcn.md |
| `task-templates/build-login-form.md` | "做一个登录页"、"做一个注册/忘记密码页面" | Senior Frontend Engineer | nextjs-shadcn.md / vite-vue3-typescript.md |
| `task-templates/build-crud-api.md` | "做一个用户管理的 REST API"、"做一个订单 CRUD 后端" | Senior Backend Engineer | node-express-prisma.md + shared-security.md |
| `task-templates/debug-error.md` | "这段代码报错了 TypeError: xxx"、"为什么页面白屏" | Senior Frontend / Backend Engineer | 按项目上下文选用 |
| `task-templates/code-review.md` | "帮我 review 这段代码"、"这个组件写得好不好" | Senior Frontend / Backend Engineer | 按项目上下文选用 |
| `task-templates/write-tests.md` | "给这个组件写测试"、"为这个 API 写单元测试" | Senior Frontend / Backend Engineer | 按项目上下文选用 |

---

## 🎯 标准工作流（AI 的执行流程 — 每步都具体可执行）

> **这是 AI 执行的核心流程。** 当你（AI）接收到一个网页相关请求时，按以下 Step 1 → Step 7 顺序执行。

### Step 1 · 解析用户需求 → 判断类型

扫描用户输入中的关键词，用下面的**判定规则**决定任务类别（只命中一个主类别；命中多个时向用户确认）：

| 关键词命中 | 归类为 | 使用 T3 模板 |
|-----------|--------|-------------|
| "官网"、"品牌站"、"公司主页"、"落地页"、"Awwwards"、"很酷"、"Apple 那种感觉" | **艺术化官网** | `build-artistic-website.md` |
| "大屏"、"数据可视化"、"KPI"、"看板"、"监控" | **数据大屏** | `build-dashboard.md` |
| "后台"、"管理系统"、"CRM"、"表格 + 表单" | **后台管理** | 使用 T1 `nextjs-shadcn.md` 或 `vite-vue3-typescript.md` + `build-login-form.md` / 自建 CRUD 页 |
| "API"、"接口"、"后端"、"CRUD" | **后端 API** | `build-crud-api.md` |
| "报错"、"调试"、"bug"、"白屏" | **调试** | `debug-error.md` |
| "review"、"审查"、"好不好" | **代码审查** | `code-review.md` |
| "测试"、"单元测试"、"e2e" | **测试代码** | `write-tests.md` |

**规则**：无法判定 → 向用户提出 1-2 个澄清问题，**不要自己猜**。

---

### Step 2 · 确认风格档位（官网类的关键一步）

如果命中"艺术化官网"类别，在真正写代码之前，先给用户一条**1 行的设计摘要**，格式固定为：

```
采用 [方向] + [色彩]，使用 [档位]，交互签名：[列出 3-5 个 IS-xx]。
页面结构：[列出 5-6 个 story beat 名称]。
```

- **默认值**（用户不反对即生效）：Editorial 杂志风 + 奶油纸 `#F5F3EF` / 墨黑 `#1A1A1A` / 赭石 `#E0512B` + 🚀 快速原型档 + IS-01 · IS-04 · IS-05 · IS-11 · IS-03。
- **若用户指定了色彩/方向**，以用户为准；否则使用默认值。
- **若用户要求生产级**，升级到 🏭 生产级档（Vue 3 / Next.js + TypeScript + Vite）。

**非官网类（大屏/后台/API/调试）跳过 Step 2，直接按对应模板的默认值。**

---

### Step 3 · 注入三层 Prompt 并生成代码

把下面这三段拼接起来，作为你（AI）自己的完整执行上下文：

```
[ T1 项目级规则 ] ← 从 project-rules/artistic-stack.md 读取（或对应模板的 T1）
  · 核心技术栈清单
  · 颜色系统（3-4 色，CSS Variables）
  · 字体系统（clamp + vw 标题字号）
  · GSAP + Lenis 标准启动代码
  · 性能与可访问性硬性规则
  · 文件结构约定

[ T2 专家角色 ] ← 从 expert-personas/creative-technologist.md（或对应角色）读取
  · role（身份 + 理念）
  · thinking_process（6 步思考流程）
  · design_principles（排版/颜色/运动/留白/材质 5 条原则）
  · code_style（文件结构 + 禁止 @keyframes + 语义化 HTML）
  · negative_constraints（8 条"NEVER" 红线）
  · output_protocol（File: path + 完整代码 + 1-3 条 bullet 说明 + 风险备注）

[ T3 任务模板 ] ← 从 task-templates/build-artistic-website.md（或对应模板）读取
  · Context · Objective · Style · Tone · Audience · Response Format · Constraints · Negative Constraints

[ 用户具体需求 ] ← 用户刚才说的原话
```

**执行要求**：
1. 代码中 **禁止使用 `// TODO` / `// ...` / 省略号**，必须完整可运行
2. 每个 GSAP timeline 必须写明 `duration` + `ease`（不要依赖默认值）
3. 不要输出"好的！"之类的客套语；先给设计摘要，再直接输出文件

---

### Step 4 · 生成后自检清单（代码生成完成但尚未输出前，AI 必须自查）

对照下面 8 项。有任何一项不满足，**回到代码修改后再输出**。

```
☐ 颜色系统 ≤ 4 色（--bg + --text + --accent + 可选 --muted）
☐ 所有标题字号使用 clamp() 含 vw 单位，禁止纯 px 大标题
☐ 实现了 ≥ 3 个交互签名（从 DNA 库挑选且代码中明显可见）
☐ 存在 @media (max-width: 768px) 降级规则（关闭 shader / 光标 / magnetic）
☐ 代码中存在 Lenis 初始化 + GSAP ScrollTrigger 注册
☐ 代码中存在 prefers-reduced-motion 查询并做降级处理
☐ 无第三方 cookies / analytics / ads 脚本（除非用户明确要求）
☐ 代码无省略号、无占位注释，保存为文件可直接运行
```

---

### Step 5 · 输出格式（AI 交付给用户的标准格式）

每一个文件严格按 T2 角色中 `output_protocol` 格式输出：

```
1. `File: path/to/file.ext` — 文件路径
2. ```language
   完整文件内容（无 "//..." 或省略号）
   ```
3. 简短说明：1-3 条 bullets，解释非显而易见的设计决定
   （例如：Hero 标题使用 stagger=0.08 + power3.out，匹配杂志排版节奏）
4. 风险/备注（如有）：性能注意事项、移动端降级、WebGL fail fallback 策略
```

多文件项目按字母序输出（index.html → main.css → main.js → animations/hero.js → webgl/orb.js …）。

---

### Step 6 · 给用户的附带说明（所有项目必加）

代码输出完后，附上这一段，帮助用户理解产出与后续修改：

```
本页使用的交互签名：
  ① IS-xx · 签名名称（关键参数：xxx）
  ② IS-xx · 签名名称（关键参数：xxx）
  ……

如何修改：
  · 换色 → 修改 :root 中的 --bg / --text / --accent（仅 3 行）
  · 换字 → 修改 <head> 中 Google Fonts 链接 + :root 中的 --font-display / --font-body
  · 改文案 → 在对应 <section> 中直接替换
  · 加签名 → 在 references/artistic-website-dna.md 中挑一个，把代码骨架粘到 <script> 末尾

部署：
  · 🚀 快速原型档：双击 index.html 即可运行，或上传到任意静态托管（Vercel / Netlify / GitHub Pages）
  · 🏭 生产级档：npm install + npm run dev 本地开发；npm run build + 上传 dist/
```

---

### Step 7 · 收集反馈与迭代

完成以上步骤后，询问用户一句：`需要调整风格 / 色彩 / 交互签名，还是升级到生产级档？` — 然后等待用户的下一步指令。

---

## 📦 完整文件索引（告诉 AI 每个文件里有什么）

> 这是"内部检索表"。AI 在执行流程中应能根据关键词快速定位到某个 md。

| 文件路径 | 内容摘要 | 何时调用 |
|---------|---------|---------|
| `SKILL.md`（本文件） | v3 总入口、一句话使用方式、三层架构说明、设计 DNA 选单、技术栈硬约束、任务模板列表、7-Step 执行流程、自检清单 | 所有请求的入口 |
| `references/artistic-website-dna.md` | 3 大设计方向（Editorial / Cyberpunk / Playful）完整色彩+字体+布局+内容结构；15 个交互签名的完整技术配方（参数、ease、参考代码骨架）；常见陷阱清单 | **任何官网类请求必读**；选择方向与交互签名的"源头" |
| `references/frontend-awwwards-website.md` | Awwwards 级网站逆向研究，更偏向 shader / WebGL / 滚动叙事实现细节 | Cyberpunk 方向、用户要求"超酷"效果时的进阶参考 |
| `project-rules/artistic-stack.md` | 艺术官网的 T1 规则：核心技术栈清单、颜色系统约定、字体系统约定、GSAP + Lenis 标准启动代码、性能与可访问性硬性规则、单文件 vs 模块化文件结构约定 | 任何艺术化官网请求的 **T1 注入源** |
| `project-rules/nextjs-shadcn.md` | Next.js 14+ · TypeScript · Tailwind · shadcn/ui 项目规则 | 生产级官网 / 后台管理 / 需要 SSR 的项目 |
| `project-rules/vite-vue3-typescript.md` | Vue 3 · Vite · Pinia · Element Plus 项目规则 | 生产级官网 / 数据大屏 / 后台管理（Vue 栈） |
| `project-rules/node-express-prisma.md` | Node.js · Express · Prisma · PostgreSQL 项目规则 | 后端 API / CRUD 项目 |
| `project-rules/shared-security.md` | 所有项目必加的安全规则（CSP、CSRF、XSS、鉴权、密钥管理） | 任何后端/生产级项目的附加 T1 |
| `expert-personas/creative-technologist.md` | 创意技术工程师角色：身份 + 6 步思考流程 + 5 条设计原则（排版/颜色/运动/留白/材质）+ code_style + 8 条"NEVER"红线 + 输出协议 | **艺术化官网的默认 T2** |
| `expert-personas/senior-frontend-engineer.md` | 10 年经验高级前端：完整代码可运行、类型显式、错误处理、无占位注释 | 普通前端 / 大屏 / 后台管理 的 T2 |
| `expert-personas/senior-backend-engineer.md` | 资深后端工程师：REST 规范、数据库设计、错误边界 | 后端 API 的 T2 |
| `expert-personas/senior-fullstack-engineer.md` | 全栈工程师：关注 API 契约优先 | 前后端一体化项目的 T2 |
| `expert-personas/senior-ai-agent-developer.md` | AI / LLM 应用开发专家：RAG、Agent、工具调用 | AI 聊天/Agent 类项目的 T2 |
| `task-templates/build-artistic-website.md` | 艺术化官网的完整 CO-STAR Prompt：Context · Objective · Style · Tone · Audience · Response Format · Constraints · Negative Constraints；含 🚀 快速原型档 与 🏭 生产级档两套骨架 | **官网类请求的 T3 主模板** |
| `task-templates/build-dashboard.md` | 数据可视化大屏的 CO-STAR Prompt | 大屏类请求的 T3 |
| `task-templates/build-login-form.md` | 登录/注册/忘记密码的 CO-STAR Prompt | 登录页类请求的 T3 |
| `task-templates/build-crud-api.md` | REST CRUD API 的 CO-STAR Prompt | 后端 API 类请求的 T3 |
| `task-templates/debug-error.md` | 调试一个错误的 CO-STAR Prompt | 排错类请求的 T3 |
| `task-templates/code-review.md` | 代码审查的 CO-STAR Prompt | Code Review 的 T3 |
| `task-templates/write-tests.md` | 写测试代码的 CO-STAR Prompt | 测试生成类请求的 T3 |
| `SYSTEM_PROMPT_ARCHITECTURE.md` | 三层 Prompt 架构的设计文档（调研来源、设计决策背景） | 仅在"为什么这么设计"层面参考，不参与代码生成 |

---

## 🔧 调试清单（代码生成后自动核对）

> AI 在 Step 4 自检时，逐项打勾；有任何一项不满足必须返工。
> 人类用户也可以对照这份清单自查你的产出是否达标。

```
☐ 颜色系统 ≤ 4 色（--bg + --text + --accent + 可选 --muted）
☐ 标题字号使用 clamp() 含 vw 单位（例如 clamp(3rem, 10vw, 9rem)）
☐ **排版安全：所有 line-height ≥ 1.0（display 标题 ≥ 1.05，body 正文 ≥ 1.5）**
☐ **排版安全：无 overflow: hidden 与 line-height < 1.1 的组合（会导致文字裁剪）**
☐ **排版安全：中文内容行高 ≥ 1.6，西文正文行高 ≥ 1.5**
☐ 实现了 ≥ 3 个交互签名（代码中能明确对应到 IS-xx）
☐ 响应式：存在 @media (max-width: 768px) 降级规则
☐ Lenis 初始化存在：new Lenis() + lenis.on('scroll', ScrollTrigger.update)
☐ GSAP ScrollTrigger 存在：gsap.registerPlugin(ScrollTrigger) + 至少 1 个 scrub 动画
☐ prefers-reduced-motion 查询存在，并关闭了 shader / scrub / stagger
☐ 无第三方 cookies / analytics / ads（除非用户明确要求）
☐ 代码无省略号、无 "// TODO"、无 "// 这里放 xxx" 占位注释
☐ 文件保存为 .html/.css/.js 后可直接运行（单文件原型档：双击打开）
☐ Google Fonts preconnect + font-display: swap 存在
☐ document.fonts.ready.then() 触发动画存在（避免 CLS 跳帧）
☐ WebGL shader orb（如果用了）有 SVG/图片 fallback 方案
☐ 代码没有使用 CSS @keyframes 动画（所有动画走 GSAP）
☐ 没有 animate width / top / left / font-size；用 transform + opacity
```

---

## ✨ 一句话总结（本文件存在的意义）

> 让用户"说出需求 30 秒后"看到符合**设计 DNA**、遵守**技术栈硬约束**、带**3-5 个交互签名**、可**直接运行**的网页代码。

如果以上规则与某个子 md 文件冲突，**以子 md 为准**，在回复中注明"此处依据 xxx.md 的更具体规则执行"。

---
> Source: [xcsweb/prompt-craft](https://github.com/xcsweb/prompt-craft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
