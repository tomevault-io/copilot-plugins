## logtool

> This repository uses role-based agents to constrain implementation quality, change scope, and delivery risk.

# AGENTS

This repository uses role-based agents to constrain implementation quality, change scope, and delivery risk.

## 1) 项目概况与技术栈

### 项目概况

- `logtool` 是一个面向医疗设备/运维场景的综合平台，不是单一日志工具。
- 核心能力包含：
  - 日志管理与分析：日志上传、解析/重解析、批量检索、统计可视化、CSV 导出、任务队列。
  - 故障码管理与查询：故障码检索、多语言维护、XML/CSV 导出、技术排查方案附件管理。
  - 智能检索与知识库：`smart-search` 对话检索、知识库文档索引/重建/检索、会话管理。
  - 手术数据分析：手术统计、按设备/时间分析、导出与可视化。
  - 故障案例与反馈闭环：案例库（含 i18n）、用户反馈流转、运维监控与审计日志。
  - 权限与治理：用户/角色/权限、操作日志、监控面板。
- 当前架构方向：不推进完整 Agent 化；保留现有单 `CoreTaskAgent` 作为统一接入与执行壳，能力建设聚焦 `smart-search`。工具路径与 `docs/Agent.md` 一致：意图 → 规划 → 工具执行 → **将 ToolResult 等写入消息后**再构建 `ContextEnvelope`，重新走意图与规划，直至 `reply_direct` 再持久化助手消息。
- 当前能力目标：
  - 会话式智能搜索：基于单位时间 token + 会话轮次进行限制。
  - 意图识别拓展：上传文件日志分析、手术数据分析、多模态查询预留、故障案例收集。
  - 钉钉渠道增强：接入钉钉机器人 MAGI，经 `dingtalk_stream` 适配器统一进入 Agent 网关。
- 执行模式：默认同步，超时自动异步化（保持现状）。

### 技术栈

- 前端：
  - `Vue 3`、`Vue Router`、`Vuex`、`Element Plus`、`Vant`、`axios`、`vue-i18n`、`ECharts`。
- 后端：
  - `Node.js`、`Express`、`Sequelize`、`Mongoose`、`Redis`、`Bull`、`ws`。
  - 钉钉接入：`dingtalk-stream` 长连接（`dingtalk_stream` 适配器），经 `sessionWebhook` 回推回复；无 HTTP Webhook 入口。
- 数据与存储：
  - MySQL 业务数据、ClickHouse 日志数据冷热分离、MongoDB 故障案例+智能查询历史记录、Redis，并包含 PostgreSQL（手术统计相关）与 Elasticsearch（故障码检索相关）能力接入。

## 2) 前端实现约束

### Token 优先策略

- 样式实现必须优先使用设计 Token，避免在页面或组件中硬编码颜色、间距、圆角、阴影。
- Token 权威文件：
  - `frontend/src/assets/styles/design-tokens.css`
  - `frontend/src/assets/styles/mobile-design-tokens.css`
  - `frontend/src/assets/styles/element-plus-theme.css`
- 设计系统参考：
  - `frontend/src/assets/styles/DESIGN-SYSTEM.md`

### 样式边界约束

- `frontend/src/components/base/**` 下的 Base 组件应保持轻样式，优先通过主题映射生效。
- 新页面样式应复用现有语义化 Token 组合，不新增页面私有 Token 命名。
- 移动端视觉覆盖必须遵循 `mobile-design-tokens.css` 既有平台作用域规则。

### 前端验证基线

- 必须执行：
  - `npm --prefix frontend run lint`

## 3) 后端实现约束

- 分层规则：默认依赖方向 `route -> controller -> service/tool -> model`，禁止跨层耦合和在 controller 堆叠复杂业务。
- 鉴权规则：所有 `/api/**` 默认必须 `auth`，并配置权限校验；例外必须在代码中注明原因。
- 队列规则：队列接口分权为 `queue:read`（状态查询）和 `queue:manage`（清理/暂停/恢复）。
- 认证规则：默认使用 `Authorization: Bearer`；`query token` 仅允许白名单资源型 GET 接口。
- 安全规则：禁止在日志输出敏感凭证（Authorization、JWT、secret、token）；外部回调必须做来源校验。
- 数据规则：MySQL/ClickHouse/MongoDB/Redis/PostgreSQL/Elasticsearch 职责分离，索引与缓存不作为唯一业务真相。
- 可靠性规则：异步任务必须有重试策略与幂等方案，失败状态可追踪。
- 可观测性规则：关键链路必须透传 `traceId/requestId`，日志可用于审计和故障定位。

## 4) 当前需要做的任务



## 5) 工作原则
- 任何任务的开发都必须按照`docs/trytoHarness/task.md`要求执行；
- 实现前复核方案是否足够简单、边界是否清晰（控制面 vs 文档计算面）。
- 尽量避免「吞掉异常」的兜底；错误应可观测、可定位，再用明确分支处理。
- 能用系统自带或项目已有能力解决，就不要额外引入新工具，开发环境是windows系统，可以根据实际情况优先 PowerShell 或 Python。

---
> Source: [syz007110/logtool](https://github.com/syz007110/logtool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
