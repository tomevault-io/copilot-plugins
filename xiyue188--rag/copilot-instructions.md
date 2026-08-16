## rag

> 本文件是本仓库所有 AI 会话的入口。**每次新会话必须先完整读完本文件。**

# CLAUDE.md — Agent 工作契约

本文件是本仓库所有 AI 会话的入口。**每次新会话必须先完整读完本文件。**

## 事实来源优先级

1. **代码与 `git log`** — 唯一权威
2. `STATUS.md` / `docs/handoffs/LATEST.md` — 导航用，可能过期
3. 聊天记录 — **不可依赖**，跨会话不共享

文档与代码冲突时，**相信代码**，并顺手修正文档。禁止基于"上次会话提到过"这类记忆行事。

## 每次会话的启动清单

按顺序读：

1. `CLAUDE.md`（本文件）
2. `docs/ROADMAP.md` —— 全局蓝图、阶段依赖、已定决策
3. `STATUS.md` —— 当前位置与阻塞项
4. `docs/handoffs/LATEST.md` —— 上次会话交接
5. `STATUS.md` 指向的当前阶段 `docs/plans/M{n}-*.md`
6. `git log --oneline -10` + `git status` + 未提交 diff

`README.md` 是给外部读者的，只在改动它本身时读。

## 工作循环：Plan → Implement → Validate → Review → Record

**Plan** — 动手前先声明：当前状态理解、本次边界（做什么/不做什么）、输入输出、失败条件、验收标准。边界外的问题记进 `STATUS.md` 的"待办观察"，不顺手改。

**Implement** — 小步提交。每个提交是一个自洽的可回滚单元。

**Validate** — 声明的验收标准逐条实测，贴真实输出。测不了的明说"未验证"，不写"应该可以"。

**Review** — 每次产出代码后，强制回答（不适用的写"不适用"并说明）：

1. 输入、输出、副作用是什么？
2. 异常在哪里被捕获？未捕获的会冒泡到哪？
3. 并发下是否安全？共享可变状态在哪？
4. 写入失败会不会留下半成功状态？
5. 为什么选这个组件/方案，而不是替代方案？
6. 怎么测试？
7. 线上怎么观察它出错？

**Record** — 结束前更新 `STATUS.md`、覆写 `docs/handoffs/LATEST.md`、涉及技术选型的补 `docs/decisions/ADR-*.md`、跑质量检查、给出建议提交信息。

## 架构约束

- `rag/` 是核心逻辑，**不得 import `backend/`**。依赖方向单向：`backend/` → `rag/`。
- `config.py` 是配置唯一入口，不在业务代码里读 `os.getenv`。
- 相关性分数只从 `rag/scoring.py` 取，禁止在别处重新发明归一化。
- 向量库 collection 名必须带 embedding 模型指纹，不同模型物理隔离。
- Redis 不作为事实来源，只放缓存/限流/取消信号/事件流。
- 向量留在 ChromaDB；PostgreSQL 存关系数据与可追溯轨迹。不引入 pgvector。

## 禁止事项

- 禁止 `eval` / `exec` 处理用户输入（计算器工具用 `ast` 白名单）。
- 禁止在日志、SSE、异常信息里输出 API Key 或完整系统提示词。
- 禁止把 `.env` 加入版本控制。
- 禁止 `--force` 推送、`reset --hard`、`clean -f`，除用户明确要求。
- 禁止用 `or` 链在多个数值分数间做回退（`0.0` 会被误判为缺失）。
- 禁止无声降级：检索不到就说不到，不放宽阈值凑结果。

## 提交规范

中文，小步，一个提交一件事。

```
feat: 新功能
fix: 修 bug
refactor: 重构，行为不变
perf: 性能
test: 测试
docs: 文档
chore: 构建/依赖/杂项
```

正文说清「为什么」，不复述 diff。

## 质量检查

```bash
# 后端
venv/Scripts/python.exe -m pytest tests/ -q
# 前端
cd frontend-vue && npx vue-tsc --noEmit
```

---
> Source: [xiyue188/RAG](https://github.com/xiyue188/RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
