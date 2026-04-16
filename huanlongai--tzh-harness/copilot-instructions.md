## tzh-harness

> > **Write-Owner: NODE-M** — MULTI-NODE-COWORK-SPEC v0.3 §3.2 | 跨域写入须遵循 §3.4

# tzh-Harness — Project CLAUDE.md

> **Write-Owner: NODE-M** — MULTI-NODE-COWORK-SPEC v0.3 §3.2 | 跨域写入须遵循 §3.4

## 仓库定位

tzh-Harness 是 Harness Engineering 方法论仓库（_governance 域），定义 Playbook、SOP 标准与工程最佳实践。
位于 `~/Workspace/01_Repos/_governance/tzh-Harness`。

## 核心规则

1. **Playbook 是 SSOT** — 所有工程流程以 playbooks/ 目录为权威
2. **中文为主** — 文档用简体中文，术语保留英文原文
3. **只读治理** — 本仓库定义方法论，不实现代码

## 上游治理

- 母体：tzhOS（宪法层）
- 消费方：所有 PPR 域仓库


---

## Consistency Sentinel

本仓库已部署 Consistency Sentinel CI 门禁（CONSEN-SPEC-001 v1.1）。

- **架构**：Hub-and-Spoke，可复用 workflow 托管于 huanlongAI/sentinel-shared
- **触发**：PR opened/synchronize/reopened + push to main + workflow_dispatch
- **确定性预检**：D-1 CHANGELOG · D-2 术语 · D-3 级联引用 · D-4 目录 · D-5 能力标注 · D-6 Brand Token
- **LLM 审查**：claude-sonnet-4-6，失败时优雅降级为 ESCALATE（非阻塞）
- **配置**：`.sentinel/config.yaml`（仓库级 profile）
- **规格**：tzhOS/CONSEN-SPEC-001.md · tzhOS/REPO-MAP.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huanlongAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
