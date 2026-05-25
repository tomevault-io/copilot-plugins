## spark-ai-copilot

> 你是 Data-AI-Copilot 项目的全栈工程师。请严格遵循当前代码库的真实架构与视觉规范，勿按旧版「全局暗黑 zinc」人设改 UI。

你是 Data-AI-Copilot 项目的全栈工程师。请严格遵循当前代码库的真实架构与视觉规范，勿按旧版「全局暗黑 zinc」人设改 UI。

## 1. 技术栈
- Next.js 14 (App Router)、TypeScript、Tailwind CSS、Lucide React
- 对话：DeepSeek API（`app/api/chat/route.ts` 流式）
- Markdown：`react-markdown` + `react-syntax-highlighter`（代码块深色底）
- 持久化：仅 LocalStorage（`hooks/useChatHistory.ts`），禁止引入外部数据库

## 2. 视觉规范（与 `app/page.tsx` 一致 · 浅色医疗/控制台风）

**主界面（默认浅色）**
- 页面底：`bg-[#F8FAFD]`，侧栏 `bg-[#F6F8FB]`，细边框 `border-slate-200/40`～`border-slate-300/50`
- 主文案：`text-slate-800`；次要：`text-slate-400`～`text-slate-600`
- 品牌强调色：`#5B8CFF` / `#4F7CFF`（Logo、选中态左边框）
- 卡片/技能按钮：白底半透明 `bg-white/65`～`bg-white/90`，轻阴影与 `hover:-translate-y` 微动效
- 用户气泡：蓝色渐变 `from-blue-500 to-blue-600`；助手区：浅底流式 Markdown，无厚重气泡框
- 聊天区宽度：`max-w-[1280px]` 外壳，内容 `max-w-[min(880px,70vw)]` 居中

**按 Skill 区分的点缀色（侧栏图标底）**
- Spark：`bg-blue-50` + `text-blue-500`
- 数仓：`bg-amber-50` + `text-amber-500`
- SQL 降本：`bg-indigo-50` + `text-indigo-500`

**深色例外（仅诊断/高危组件）**
- `components/spark/SparkDiagnosisPanel.tsx`：极客确诊单 `bg-zinc-950`、`border-red-900/30`，用于助手侧系统诊断，不要扩散到全站布局

**禁止**
- 勿将全站改回 `bg-zinc-950` 暗黑极简风，除非用户明确要求

## 3. 产品与模块结构

**三大 Skill（`app/page.tsx` 内 `SKILLS`）**
1. ✨ Spark 倾斜诊断（默认）— 规则引擎 + 确诊单 + 知识库
2. 📚 数仓资产管家 — 字典图谱裁剪 + `GraphRefineBadge`
3. 💰 智能 SQL 降本评审 — `lib/sql-sandbox.ts` 静态审查

**Spark 链路（必守分层）**
- `lib/spark/types.ts` / `mock-metrics.ts` / `log-parser.ts` / `scenario-matcher.ts` / `knowledge.ts`
- 诊断单由规则引擎生成，挂在**助手消息** `Message.sparkDiagnosis`（`SparkDiagnosisPanel`），用户气泡只存原始问题
- 知识库目录 `public/knowledge/`：
  - `spark_tuning_sop.md`（system prompt）
  - `spark_case_library.json`、`spark_param_presets.json`、`spark_anti_patterns.md`（经 `knowledge.ts` 按场景裁剪注入）
- 后端：`buildSparkContext()` → `enhancedPrompt`；禁止在 ts/tsx 硬编码大段 SOP/案例

**其他后端模块**
- `lib/mcp-engine.ts`：SOP、数仓字典、applicationId 提取
- `lib/security-shield.ts`：脱敏与表名混淆（流式还原）
- `app/api/chat/route.ts`：按 Skill 分支；全 Skill 遵守篇幅约束（`max_tokens` + 简明 system 规则）

## 4. 代码修改原则
- 最小 diff；未改动处用 `// ... 保持不变` 折叠，禁止整文件无意义重写
- 静态知识一律 `public/knowledge/*` 或 `lib/spark/*` 文件流读取
- 新增 Spark UI 放 `components/spark/`；通用聊天逻辑保留在 `app/page.tsx` 除非已臃肿到需拆分
- 流式、LocalStorage、三 Skill 路由行为非需求勿改

## 5. Token / 交付原则
- 实现任务：直接改文件，少废话
- API 回复已由后端约束为短答（≈800 字内、Top3 动作）；前端勿再堆长文说明组件
- 用户仅改 `.cursorrules` 或知识库时，不要顺带改 UI 配色

---
> Source: [r6z7ysn7yh-droid/spark-ai-copilot](https://github.com/r6z7ysn7yh-droid/spark-ai-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
