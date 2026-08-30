## knowledgeagent

> 任何代理接手本仓库时，先读完下面三份文件再编码：

# Agent 工作约定

任何代理接手本仓库时，先读完下面三份文件再编码：

1. [开发规划.md](开发规划.md)
2. 本文件 `AGENTS.md`
3. [进度记录.md](进度记录.md)（含当前阶段与未解决问题）

不确定时优先在规划文档中查找；规划未写明再自行决策，并在进度记录中留下依据。

## 阶段纪律

- 按 Phase 0 → 7 顺序执行，**一次只做一个阶段**。
- 开始阶段前，先更新 `进度记录.md` 的「当前阶段 / 目标 / 风险」。
- 结束阶段时记录：实际改动、执行过的命令、测试输出摘要、截图或接口示例、未解决问题、下一阶段不超过三项任务。
- 同步新增 `docs/progress/phase-N.md`（目标、改动、命令、结果、风险、下一步）。
- 进度文件只追加或修订对应阶段，不删除历史结论。
- 若测试失败，先修复再进入下一阶段。

## 完成后必须运行

```bash
cd backend && uv run ruff check . && uv run pytest
cd ../frontend && npm run lint && npm run test:unit && npm run build
```

## 禁止提交

- 真实密钥、`.env`（保留 `.env.example`）
- 上传原文件、解析产物、`storage/` 中的用户数据
- 数据库文件、向量缓存、日志
- `node_modules`、Python 虚拟环境、前端 `dist/` 等构建产物

## 范围边界

v1 不做微服务、Kubernetes、消息队列、复杂 RBAC/SSO、计费、多 Agent、跨企业多租户、企业 IM、复杂长期记忆和自研模型。后续需求只能在 v1 验收后新增。

---
> Source: [lds051332/KnowledgeAgent](https://github.com/lds051332/KnowledgeAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
