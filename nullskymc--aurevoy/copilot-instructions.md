## aurevoy

> > 参与本仓库的智能体/人类先读本文，再按需下钻 `docs/`。

# AGENTS.md — Aurevoy 协作入口

> 参与本仓库的智能体/人类先读本文，再按需下钻 `docs/`。

## 产品

Aurevoy 是**本地个人 AI Agent 桌面应用**：用户用自然语言给目标，引擎规划、调工具、持续执行直至完成。

- 版本：v0.6.8（已发布 tag `v0.6.8`；上一版 `v0.6.7`）
- 形态：支持 macOS、Windows 和 Linux 桌面端；各平台通过 Tauri 构建对应安装包与更新产物
- 原则：真实链路、可恢复、可审计；禁止 Mock / 假能力

## 架构（30 秒）

```
apps/desktop (Tauri) + packages/web-ui (React)
              │  HTTP + SSE (127.0.0.1:8787)
              ▼
         apps/agent (Node + Fastify + Pi AgentHarness)
              ├── LLM (多协议)  ·  Tools (Effect + MCP)
              └── SQLite (任务 / 轨迹 / 记忆 / KB)
packages/shared  ← 跨进程类型唯一来源
```

主循环固定走 Pi runtime；LLM 等产品配置走设置页 / SQLite，env 仅运维面。详见 [ARCHITECTURE](./docs/ARCHITECTURE.md)。

## 文档地图

| 文档 | 用途 |
|---|---|
| 本文 | 入口、规则、命令 |
| [ARCHITECTURE](./docs/ARCHITECTURE.md) | 模块与数据流 |
| [API](./docs/API.md) | HTTP + SSE 契约说明 |
| [UI_DESIGN](./docs/UI_DESIGN.md) | 交互与界面边界 |
| [CONVENTIONS](./docs/CONVENTIONS.md) | 代码与扩展约定 |
| [TECH_STACK](./docs/TECH_STACK.md) | 选型理由 |
| [ROADMAP](./docs/ROADMAP.md) | 已完成能力 + 未竟项 |
| **文档站** | 用户文档：`docs/guide/`（quickstart / prompting / best-practices…）；开发者：`docs/dev/` + 下文协作页 → [aurevoy.nullskymc.site](https://aurevoy.nullskymc.site/) |

类型真相在 `packages/shared/src/`，API 细节以代码为准。用户可见行为变化时优先改 `docs/guide/`（可操作步骤 + 示例 + 反例）；契约/架构变化再改协作参考页。

文档站：`npm run docs:dev` / `docs:build` / `docs:preview`。推送 `docs/**` 到 `main`/`dev` 时由 `.github/workflows/docs.yml` 部署 Pages。

## 硬性规则

1. 跨进程数据结构只定义在 `packages/shared`；涉及 shared 改动的单个任务完成时必执行 `npm run build:shared`。
2. 新工具放 `apps/agent/src/tool/tools/`，经 `framework/registry` 注册；不写进 Agent loop。
3. 新 Provider 扩展 Pi 映射，不恢复第二套 ReAct 后端。
4. 后端不绑 macOS 路径/命令；前端不假设非 Tauri 环境。
5. 外部能力缺失时明确失败/降级，不假成功。
6. 验证分层：单个任务完成时至少 `npm run typecheck`；行为/安全/存储改动补定向回归或可复现轨迹。
   提交前按改动影响面跑相关回归即可（见下方命令表），不必每次全套 m3–m8；
   完整回归仅在发布/合并前统一执行。
7. 密钥只走环境变量 / 设置存储，禁止提交。
8. 新能力同步考虑：日志、审批、权限、失败恢复、用户可解释性。

## 命令

```bash
npm install              # workspace 依赖（Node >= 22.19.0）
npm run dev              # 引擎 + 桌面
npm run dev:agent        # 仅引擎热重载
npm run typecheck        # 全仓类型检查
npm run build            # shared → agent → desktop
npm run build:shared     # 改 shared 后必做
npm run regression:m3    # 文件工具/工作区（工具改动 → m3+m7）
npm run regression:m4    # 多轮对话/历史（LLM/对话改动 → m4+m7）
npm run regression:m5    # 项目工作区切换
npm run regression:m6    # 记忆/Session Tree（记忆改动 → m6+m8）
npm run regression:m7    # agent 循环/工具 schema/MCP/审批（通用联动）
npm run regression:m8    # 知识库/向量检索（KB 改动 → m6+m8）
npm run regression:*     # 提交前按影响面选跑；全套留给发布/合并前
npm run docs:dev         # 文档站本地预览
```

环境：Node ≥ 22.19.0、Rust stable、macOS Xcode CLT。

## 能力现状（摘要）

**已交付：** Pi harness、多协议 LLM、Effect 工具 + MCP、agent 自动执行、规划/侦查、工具门禁（暂停恢复）、多轮/恢复、  
内联编辑重试/分支/压缩、子代理（并发与工作组 UI）、Skill、网页搜索、多模态、项目工作区、  
双层预算、记忆向量 + KB RAG、设置/数据管理、CI。

**进行中：** macOS Apple 签名与公证、隐式 KB 召回、评测与发布体验。  
**已交付（分发）：** GitHub Releases 自动更新（Tauri updater + `latest.json`）。维护见 [docs/dev/auto-update.md](./docs/dev/auto-update.md)。  
清单与验收见 [ROADMAP](./docs/ROADMAP.md)。

---
> Source: [nullskymc/Aurevoy](https://github.com/nullskymc/Aurevoy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
