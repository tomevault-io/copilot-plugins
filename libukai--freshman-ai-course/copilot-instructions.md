## freshman-ai-course

> 本文件适用于整个 `freshman-ai-course` 仓库。任何 Agent 在修改课程内容、网站、课件或构建流程前，都应先阅读本文件，再检查当前工作区和相关源文件。

# Freshman AI Course Agent Guide

本文件适用于整个 `freshman-ai-course` 仓库。任何 Agent 在修改课程内容、网站、课件或构建流程前，都应先阅读本文件，再检查当前工作区和相关源文件。

## 1. 项目目标与边界

- 本项目是武汉大学信息管理学院面向大一新生开设的 12 周、36 节基础平台课《大模型基础与创新应用》。
- 课程前 5 周建立数字素养、人工智能、大模型和智能体产品的共同基础；第 6–11 周通过学习办公、多媒体制作和轻度编程三类应用形成成果，第 12 周展示与总结。
- 网站主体使用 VitePress 和 Vue。
- 课堂课件的在线版由 `.slide` 源生成独立 HTML，放入 `public/slides/` 后由 VitePress 作为静态文件托管。
- PPTX 用于线下授课，由同一套 `.slide` 源通过 SlideP 保存或导出；不是由 HTML 转换得到。
- 除非用户明确要求，不执行远端 Git 推送、发布或部署。

## 2. 先确认当前状态

开始工作前至少完成以下检查：

1. 阅读 `README.md`、`package.json` 和本文件。
2. 运行 `git status --short`，保留用户已有修改，不清理或覆盖无关文件。
3. 确认任务属于课程内容、网站界面、课堂课件在线版、PPTX、讲义 PDF 或构建检查中的哪一层。
4. 检查真正被页面引用的文件，不根据文件名猜测当前入口。
5. 仅在用户要求发布时检查远端 Git；本地内容与构建问题优先。

## 3. 当前有效架构

### 3.1 VitePress 网站

- `.vitepress/config.mts`：站点标题、导航、侧栏、搜索、根路径和 Pages 路径配置。
- `AGENTS.md` 与 `tools/` 是项目内部工程资料，必须通过 `srcExclude` 排除在学生站点之外。
- `.vitepress/theme/index.ts`：注册项目使用的 Vue 组件。
- `.vitepress/theme/style.css`：网站全局和自定义组件样式。
- `index.md`：网站首页入口，当前实际加载 `LandingPage.vue`。
- `.vitepress/theme/components/LandingPage.vue`：当前有效首页及其首页课程阶段数据。
- `.vitepress/theme/components/HomePage.vue` 与 `.vitepress/theme/data/course-home.ts`：当前未被首页入口引用。完成合并或删除前，不把它们当成首页事实来源。
- `.vitepress/course-sidebar.mts`：从 `week-*.md` 的标题和二级标题生成课程侧栏，不手工复制周页面锚点。

### 3.2 课程正文与结构化课节

- `course-manifest.json`：课程阶段、周标题、周说明、课件目录和页数的统一事实来源。
- `week-01.md` 至 `week-12.md`：每周课程正文、案例、表格、课堂节奏、小结和阅读入口。
- `.vitepress/theme/data/twelve-week.ts`：12 周、36 节课的结构化目标、问题、材料、时间线、完成证据和自查数据。
- `.vitepress/theme/data/workbuddy-labs.ts`：WorkBuddy 实操路径、提示词、预期结果、人工检查、禁止事项和三轨教学方案。
- `.vitepress/theme/components/LessonSession.vue`：把结构化课节和实训数据渲染到周页面。
- `roadmap.md` 与 `TwelveWeekRoadmap.vue`：从 `twelve-week.ts` 渲染课程讲义入口。

课程正文和结构化数据目前是协同维护关系，不会自动互相生成。修改周目标、课节标题、时间安排或学生产出时，必须检查两边是否都需要同步。

### 3.3 学生、教师与课程配套材料

- `workbook.md`：学生工作册和可填写模板。
- `assessment.md`：任务、展示和个人档案的评价量规。
- `teacher/teacher-guide.md`：教师内部参考，不得进入学生站点、导航或搜索。
- `practice.md`：学生可见的课程实操指南。
- `teacher/setup.md`：教师内部开课准备和边界，不进入学生站点。
- `assignments.md`：三次个人作业、随机评点、AI 首评和教师抽查规则。
- `packs/lamp-plan.md`、`public/packs/lamp-plan/`：可复用的课程模拟资料包及下载材料。
- `artifact-comparisons.md`、`glossary.md`、`reading-list.md`：前后对照、术语和延伸阅读。

### 3.4 课件

- `pptx/<deck>/slides/*.slide`：课件内容与布局的唯一源文件。
- `pptx/<deck>/<deck>.html`：由 `.slide` 生成的同目录 HTML 成品。
- `public/slides/<deck>.html`：网站使用的同步副本，不手工修改。
- `pptx/<deck>/<deck>.pptx`：由 SlideP 从同一套 `.slide` 源保存或导出的 PPTX。
- `public/slides/index.html`：课堂课件入口，由 `course-manifest.json` 和 `tools/slide2html.mjs` 自动生成，不手工修改。

课件正确关系是：

```text
.slide
├── tools/slide2html.mjs ──> deck HTML ──> public/slides ──> VitePress 静态托管
└── SlideP ──> PPTX ──> 线下授课与分发
```

不要采用 `.slide -> HTML -> PPTX` 的二次转换链路。

### 3.5 学生讲义 PDF

- `scripts/build-lecture-manuscript.mjs` 从课程大纲、12 周正文、工作册、量规、术语和阅读清单生成合订 Markdown。
- `scripts/build-lecture-pdf.sh` 将 SVG 图示转为 PNG，并通过 Pandoc 和 Tectonic 生成 PDF。
- `output/pdf/` 是本地工作成品。
- `public/downloads/lecture-notes.pdf` 是学生站点使用的交付副本，不手工修改。
- `LessonSession` 的交互式内容会从打印版中移除；修改结构化课节数据时，要判断 PDF 正文或工作册是否需要补充对应信息。

## 4. 内容修改顺序

处理课程内容修改时，按以下顺序进行：

1. 明确教学目标、受众、课时和本次要解决的问题。
2. 先修改课程主线、周目标、课节关系和学生产出，不先做视觉装饰。
3. 先更新 `course-manifest.json` 中受影响的阶段、周标题、说明或课件信息，再同步检查 `week-*.md` 与 `twelve-week.ts`。
4. 涉及实操时同步检查 `workbuddy-labs.ts`、`practice.md` 和资料包。
5. 涉及任务验收时同步检查 `workbook.md`、`assessment.md` 和教师参考。
6. 再修改对应的 `.slide` 课件源。
7. 先生成并检查 HTML 课件。
8. HTML 内容和布局确认后，再从同一批 `.slide` 导出 PPTX。
9. 渲染受影响的 PPTX 页面，检查字体、SVG、图片、溢出和跨软件兼容性。
10. 需要学生讲义时重建 PDF，最后运行相应的全项目检查。

不要只修改周正文而遗漏课程讲义入口、实训卡、工作册或课件；也不要只修改课件而让网站正文和评价标准继续保留旧逻辑。

### 大一新生能力边界

- 每周只回答一个核心问题，每节只引入一至三个新概念。
- 第 1–5 周以教师讲解、案例与截图演示为主，每周只安排轻量微练；第 6、8、10 周讲解演示，第 7、9、11 周随机评点并全员修改。
- 每位学生独立提交两次实践小作业和一次综合实践大作业；随机展示不改变成绩。
- 全员目标以“理解、应用、核验和解释”为主；从零创建 Skill、复杂项目空间、完整自动化和多 Agent 系统只进入提高轨。
- 复杂模板可以保留在资源库，但评价不得要求全员完成全部模板。
- WorkBuddy 是默认实操平台，不是学习目标。产品按钮、安装数量、Skill 数量和提示词长度不直接评分。
- WorkBuddy 卡分为核心实操、教师演示和评点复盘；手机可以完成课程要求，电脑只是可选效率设备。
- 真实邮箱、日历、通讯录、班级群和个人敏感信息不进入课堂演练。

## 5. 常见修改的同步范围

### 修改周标题或课节标题

至少检查：

- 对应 `week-*.md` 的 frontmatter、H1 和 H2；
- `course-manifest.json`；
- `.vitepress/theme/data/twelve-week.ts`；
- 对应 `.slide` 源、课件封面和目录页；
- `README.md` 中的课程结构说明。

侧栏会从周 Markdown 自动生成，不要手工维护同一份锚点列表。

### 修改课堂任务或 WorkBuddy 操作

至少检查：

- 对应 `week-*.md`；
- `twelve-week.ts` 的目标、时间线、证据和自查；
- `workbuddy-labs.ts` 的点击路径、提示词、预期结果、人工检查和禁止事项；
- `workbook.md`、`assessment.md`、`teacher/teacher-guide.md`；
- 对应 `.slide` 课件。

### 修改成果或评价标准

至少检查：

- 首页和 `roadmap.md` 的成果表述；
- 对应周正文与结构化课节；
- `workbook.md`；
- `assessment.md`；
- `teacher/teacher-guide.md`；
- `assignments.md`；
- 课程总览及相关周课件。

## 6. 构建命令与顺序

### 本地网站与课堂课件

```bash
npm run dev
```

`npm run dev` 会先运行 `npm run build:slides`，从 `.slide` 重建全部课件 HTML 并同步到 `public/slides/`，再启动 VitePress。

只重建课堂课件在线版：

```bash
npm run build:slides
```

只调试单套课件时可以直接运行 `tools/slide2html.mjs`，但正式验收必须回到全量命令。

### 学生讲义

```bash
npm run build:lecture-pdf
```

### 检查

```bash
npm run check
npm run check:pages
npm run check:course
npm run check:slides
npm run check:all
```

- 根路径构建与 GitHub Pages 路径构建必须串行执行，避免临时输出互相污染。
- `npm run check:slides` 主要检查 HTML 自动缩放，不等于 PPTX 视觉验收。
- `npm run check:all` 通过后仍需对受影响的首页、周页面、HTML 课件和 PPTX 页面做视觉检查。

## 7. PPTX 验收规则

PPTX 不能只凭“文件已生成”或“页数一致”判定完成。至少确认：

1. PPTX 不早于对应 `.slide` 源文件。
2. 页数与 `.slide`、HTML 一致。
3. 受影响页面已渲染为图片并逐页查看。
4. 标题、正文、页脚和图示没有溢出、遮挡、意外换行或不可读的小字号。
5. 中文字体在 PowerPoint、WPS 或 LibreOffice 等目标环境中没有缺字。
6. SVG 在目标查看器中能正常显示；不要只依赖 SVG-only 图片对象而没有兼容回退。
7. 出现白色矩形时，优先检查该位置是否是未被查看器解析的 SVG，而不是把它当作设计留白。
8. 简单流程图优先使用 PPT 原生形状；必须使用 SVG 时，考虑提供 PNG 回退并做跨软件验证。

## 8. 生成物与编辑边界

- 不手工修改 `public/slides/<deck>.html`；修改 `.slide` 后重新生成。
- 不手工修改 `public/slides/index.html`；修改 `course-manifest.json` 后运行 `npm run build:slides`。
- 不手工修改 `public/downloads/lecture-notes.pdf`；修改源文件后重新生成。
- 不把 `.slidep/`、`.workbuddy/`、`tmp/`、缓存、审计图片或临时输出当成课程源文件。
- 不因为生成物存在就声称已经交付；本地预览、导出文件、远端发布是不同状态。
- 不删除、重置或覆盖用户已有的未提交修改。

## 9. 完成标准

一次内容修改只有在以下条件满足后才可以称为完成：

- 课程逻辑和目标已在所有受影响的源文件中对齐；
- 对应 HTML、PPTX 或 PDF 生成物已更新；
- 根路径和 Pages 路径检查按需要串行通过；
- 受影响页面已做真实视觉检查；
- 学生站点没有包含教师答案；
- 没有把旧入口、未引用组件或临时产物误当成当前事实；
- 最终汇报明确区分：修改了什么源文件、生成了什么本地产物、做了什么验证、是否发布到远端。

## 10. 推荐的后续治理

在不改变课程内容的前提下，优先逐步完成：

1. 在确认旧首页不再需要后，移除未被引用的 `HomePage.vue`；其数据已改为读取统一课程清单。
2. 为 PPTX 建立明确的批量导出命令或受控的 SlideP 保存流程。
3. 把 PPTX 全页渲染、SVG 兼容检查和视觉验收纳入课程课件交付流程。
4. 再评估是否将这套通用流程抽成跨项目 Skill；在此之前，项目级 `AGENTS.md` 是事实来源。

---
> Source: [libukai/freshman-ai-course](https://github.com/libukai/freshman-ai-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
