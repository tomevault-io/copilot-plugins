## vibeusage

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->

# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:

- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:

- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# 仓库站点地图规则

- 非 trivial 变更前先阅读 `docs/repo-sitemap.md`，用它确定当前模块边界、入口文件与首读路径。
- 当变更影响模块边界、跨模块数据流、公共接口、或首选入口文件时，必须同步更新 `docs/repo-sitemap.md`。
- 渐进式披露：先读站点地图里的顶层目录和目标模块小节，只在依赖仍不清楚时再扩大阅读范围。
- 小改动聚焦单模块；中等改动扩展到相邻模块；跨模块/数据流改动沿着站点地图列出的路径逐步展开。

## 站点地图执行边界（强制 vs 可选）

**强制（必须读 + 更新站点地图）**

- 架构变更或系统边界调整
- 数据流/存储/同步路径变化
- 公共接口或契约改变（API、事件、数据模型）
- 跨模块耦合关系调整
- 模块首选入口文件或作者路径变化

**可选（允许跳过站点地图更新）**

- 局部 bugfix（不影响模块边界/数据流）
- 纯文案/样式/格式类改动
- 孤立脚本修补（不进入核心流程）

**最小摩擦执行规则**

1. 变更前判断是否触发“强制”条件
2. 触发则先读并在结束后更新 `docs/repo-sitemap.md`；不触发可跳过
3. 提交信息或 PR 描述可附一句：`Repo sitemap: updated` 或 `Repo sitemap: not required`

## SQLite 使用习惯（渐进式披露）

- 定位顺序：先用 `docs/repo-sitemap.md` 缩小范围（模块/路径前缀），再用 SQLite 精确查询。
- 查询原则：只输出最小结果集（几十/几百行以内），禁止全量导出。
- 模板优先：使用 `docs/graph/sql-templates.md` 的固定 SQL，避免手写出错。
- 目标定位：SQLite 仅负责“符号级事实定位”，输出结果再交给 AI。

# OpenSpec 使用范围

- 默认使用 skill 工作流，不强制走 OpenSpec。
- 仅在以下“重大模块”场景触发 OpenSpec：外部集成、跨模块核心流程、DB schema 变更、安全/权限边界变化、破坏性变更。

# 文案规则（Copy Registry）

- 本项目页面上所有展示文字必须来自 `dashboard/src/content/copy.csv`。
- 任何文案改动必须汇总到文案表，不允许在组件内新增/修改硬编码文本。
- 文案表与项目官网内容必须双向同步：官网改动需回写文案表，文案表更新需同步到官网。

# 回归用例要求

- 每次提交必须执行回归用例（至少覆盖本次变更相关路径），并记录执行命令与结果。

# PR 预检与风险层门禁

- PR 模板必须填写 `Affected Modules / Contracts`、`Validation` 与 `Risk Flags`；跨模块变更需附 repo sitemap evidence（更新说明或受影响小节）。
- 若 `Risk Flags` 勾选任一项，必须补全 `Risk Addendum`（`Rules / Invariants`、`Boundary Matrix ≥ 3`、`Evidence`）。
- 若勾选 `Public exposure / share links / unauthenticated access`，必须补全 `Public Exposure Addendum`。
- `Reviewer Context` 仅供 reviewer / AI 复审参考，不属于硬门禁。
- CI 会执行 `node scripts/ops/pr-risk-layer-gate.cjs`；在 CI 中优先读取 live PR body，event payload 仅作为 fallback；本地可用 `--body-file` 预检。
- 详细流程见 `docs/ops/pr-review-preflight.md`。

# 工作流规则（Workflow）

- 完成代码后仅执行本地提交（git commit），未经用户明确指示不得推送（git push）。

# 发布收尾规则（Release Closure）

- 发布后必须通过 CI 检测，才算收尾。
- 本地预检命令放在 `package.json` 的 `scripts.ci:local`：
  - `npm run ci:local`
- 若 GitHub Actions 的 `CI` 工作流未通过，不得标记“发布完成”。

# 复盘协议（Retrospective Contract，CLI 无关）

> 目标：让 Codex/Claude/OpenCode/Gemini 等任何 AI CLI 都走同一条复盘流程。

- **单一真源（SSOT）**：复盘流程规则只维护在本文件，不在各 AI 工具配置里复制一份。
- **目录规范**：新复盘必须放在 `docs/retrospective/<repo>/`，禁止新增到平铺根目录。
- **渐进式披露**：
  - 先看 `docs/retrospective/_index.md`（L1 卡片筛选）
  - 再看 `docs/retrospective/<repo>/_index.md`（仓库内筛选）
  - 最后才看完整复盘正文（L2/L3）
- **新复盘最小清单（强制）**：
  1. 文档含 frontmatter：`repo/layer/module/severity/design_mismatch/detection_gap`
  2. 更新全局索引：`docs/retrospective/_index.md`
  3. 更新仓库索引：`docs/retrospective/<repo>/_index.md`
- **自动门禁**：必须通过 `npm run validate:retros`。
- **AI CLI 适配原则**：任何 CLI 只需“执行前读取 AGENTS.md + 通过 validate:retros”，无需额外私有流程。

# 部署规则（Deployment）

- 所有函数都通过 Insforge2 MCP 部署。

# Insforge 聚合与契约（PostgREST）

- 聚合查询统一使用 `sum(column)` 语法，禁止使用 `column.sum()`。
- 新增/修改聚合接口必须有真实 Insforge2 环境 smoke 验证（至少 1 次请求 200 + 合理响应）。
- 若出现 `schema cache` / `relationship` + `'sum'` 相关错误，应直接走聚合 fallback 逻辑并记录根因。
- Smoke 脚本：`scripts/ops/insforge2-smoke-project-usage-summary.cjs`（需要 `VIBEUSAGE_INSFORGE_BASE_URL` 与 `VIBEUSAGE_USER_JWT`）。

---
> Source: [victorGPT/vibeusage](https://github.com/victorGPT/vibeusage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
