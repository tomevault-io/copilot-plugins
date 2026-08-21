## ai-gift

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

AIGift 是一个基于 AI 的**微信小程序礼物推荐助手**：用户描述送礼对象画像（关系/年龄/爱好/性格/禁忌等）与送礼场景，AI 生成可执行的礼物建议清单；用户填写越完整推荐越准，采纳/不采纳反馈会被记录用于优化后续推荐。商业模式为免费增值 + 广告变现（激励视频为主）。

**当前仓库状态**：仅包含需求/PRD 文档与高保真前端原型。PRD 第 11 章描述的云开发后端（云函数 / MySQL / 定时触发器）**尚未实现**——任何后端工作都是从零开始，须以 PRD 为契约。所有内容（文档与 UI）均为中文。

## 仓库结构

```
docs/
  原始需求.md                                  # 用户的一段式原始想法
  superpowers/specs/2026-08-01-AIGift-prd-design.md   # v1.0 PRD（实现的唯一权威契约）
prototype/                                      # 高保真原型的【模块化源码】（编辑入口）
  index.html  css/styles.css  js/data.js  js/app.js
aigift-prototype.html                           # 同一原型的单文件自包含快照（Open Design 产物）
aigift-prototype.html.artifact.json             # 上述快照的产物元数据
.obsidian/                                      # 目录是一个 Obsidian vault（用户用 Obsidian 编文档，勿改）
.od-skills/                                     # Open Design 平台用的 skill（web-prototype / agent-browser），非本项目代码
```

编辑原型逻辑时改 `prototype/`（模块化源码）。根目录的 `aigift-prototype.html` 是把 HTML/CSS/JS 内联打包后的自包含快照，二者会随编辑而漂移；若需同步，须明确询问用户以哪个为准。

## 原型架构（`prototype/`）

一个**单页应用**，在桌面"手机框"舞台里模拟微信小程序体验。无框架、无构建、无依赖，纯原生 HTML/CSS/JS。

- **`js/data.js`** —— 挂到 `window.AIGIFT_DATA` 的"模拟后端"：表单字典 `DICT`、必填 `REQUIRED` / 选填 `OPTIONAL_FIELDS`（驱动完整度计算）、场景模板 `SCENARIO_TEMPLATES`、种子对象 `SEED_RECIPIENTS`、礼物池 `GIFT_POOL`（带 tags/types/预算/画像 fit，供生成器筛选打分）、历史与额度流水。**新增字段或选项时，先改这里。**
- **`js/app.js`** —— 单个 IIFE，所有逻辑集中于此：
  - 中心 `state` 对象（`route` / `quota` / `recipients` / `form` / `lastResults` / `history` / `flow` 等），变更后手动调用对应 render 函数重绘（**非响应式**）。
  - SPA 路由：4 个 tab screen（recommend/recipients/history/profile）+ 1 个覆盖层 screen（result），由 `go(route)` 切换 `.is-active`。
  - 渲染函数族：`renderRecommend` / `renderResult` / `renderRecipients` / `renderHistory` / `renderProfile`，命令式拼接 HTML 字符串后注入对应 `.screen` 容器。
  - 核心业务模拟：`computeCompleteness`（必填 42% + 选填 58% 加权，分档 60%/85%）、`generateResults`（从 `GIFT_POOL` 按预算/画像打分筛选）、`handleGenerate`（额度扣减 + 流水）、`handleFeedback`（采纳/不采纳/收藏写入历史）。
- **`css/styles.css`** —— 设计令牌在 `:root`（暖色 paper/ink 底 + rose/gold/sage/terra 四色，serif 显示字体 `Fraunces`/`Noto Serif SC`，手写体 `LXGW WenKai`）。字体走 Google Fonts + jsDelivr CDN，需联网。改配色/字号应改令牌而非逐处覆盖。

## 运行与开发

无构建步骤、无包管理器、无测试。直接用浏览器打开 `prototype/index.html`（或根目录单文件版）即可。本地起静态服务器体验更接近真机：

```bash
cd prototype && python3 -m http.server 8080   # 访问 http://localhost:8080
```

## 未来后端实现的关键约定（来自 PRD，尚未落地）

一旦开始写后端，严格遵循 PRD §11，并复用用户另一个项目 **fixbill** 的目录与部署模式：

- **单一云函数 `aigift_app` + 内部 `route` 路由**：`index.js`（Timer 分发 + 业务路由 + 鉴权注入）→ `router.js`（route→handler + `PUBLIC_ROUTES` + handlerCache）→ `common/`（db/constants/response/security/utils）→ `middleware/`（auth/errorHandler/validator）→ `modules/<域>/`。
- **资源隔离**：与 fixbill **共用同一 CloudBase 环境** `cloud1-1g2pvzhx7a072799`，靠前缀隔离——云函数/表/触发器一律 `aigift_` 前缀；环境变量 `AIGIFT_MYSQL_*` / `AIGIFT_AI_*`。
- **统一响应** `{ code, message, data }`；错误码集中在 `common/constants.js`；公开路由白名单（如 `user/autoLogin`），其余经 auth 中间件校验 openid 归属。
- **数据库**（MySQL，JSON 列承载标签/画像/AI 结果）：见 PRD §10。要点——把"建议条目"从推荐主记录拆为独立表 `aigift_recommendation_items` 以便 SQL 聚合采纳率；推荐时对 `recipients` 做**快照**存入 `recommendations.recipient_snapshot`，对象日后修改不影响历史复现。
- **AI 层**：`RecommendationProvider` 抽象（默认 DeepSeek，预留 OpenAI/Qwen），System+User Prompt → 结构化 JSON；失败重试 1 次后返回场景模板兜底清单；记录 `tokens_used`/成本。
- **额度**：以 `aigift_quota_logs` 流水为准（事务防超扣），`users.free_quota` 仅缓存。
- **定时触发器**（7 段 cron）：`aigift_daily_reminder`（`0 0 9 * * * *` → `system/scanReminders`）、`aigift_db_keepalive`（每 20 分钟）。部署经验：用 `manageFunctions.updateFunctionCode`（非 `manageCloudRun`）；触发器须**一次性传完整列表**（会整体替换）。
- **安全**：AI API Key 仅存云函数环境变量，绝不前端暴露；所有写操作经云函数鉴权。

## 工作约定

- 需求或设计争议以 `docs/superpowers/specs/` 下最新 PRD 为准；改动产品行为前先读 PRD。
- 文档与 UI 文案均为中文，新增内容保持一致。

---
> Source: [loongwind/ai-gift](https://github.com/loongwind/ai-gift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
