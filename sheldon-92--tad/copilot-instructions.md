## tad

> > 路由层：什么时候做什么。执行协议在 /alex, /blake, /gate, /tad-maintain。

# TAD 框架使用规则

> 路由层：什么时候做什么。执行协议在 /alex, /blake, /gate, /tad-maintain。

## 1. Handoff 读取规则 ⚠️ CRITICAL

读取 `.tad/active/handoffs/` → 必须调用 /blake → 必须过 Gate 3 + Gate 4。
禁止：读取后直接实现、跳过 Gate、不通过 Blake 改代码。
豁免：`/tad-maintain` CHECK/SYNC 模式。

## 2. 使用场景

| 命令 | 触发条件 |
|------|----------|
| `/alex` | 新功能 (>3 文件), 架构变更, 复杂需求, 多模块重构 |
| `/alex` + `*bug` | Bug 诊断 → express handoff |
| `/alex` + `*discuss` | 产品/策略讨论 (no handoff) |
| `/alex` + `*idea` | 捕获想法 |
| `/alex` + `*learn` | 苏格拉底式学习 |
| `/alex` + `*publish` | GitHub 发布 |
| `/alex` + `*sync` | 同步到注册项目 |
| `/blake` | 有 handoff → 实现；常规发布 |
| `/gate` | Gate 1-4 |

跳过 TAD：单文件修复、配置调整、文档更新、用户说"直接帮我"。
Adaptive Complexity：Alex 评估建议，**人类做最终决策**。
Epic：多阶段任务 → Epic，同时只能 1 个 Active phase。

## 3. Quality Gates

- 规则 0: Handoff 前必须苏格拉底提问 (⚠️ BLOCKING)
- 规则 1: Handoff 必须专家审查 (min 2) + P0 修复 → Gate 2
- 规则 2: 实现后 → Gate 3 | 规则 3: 集成后 → Gate 4
- 规则 4: Gate 不通过 → 阻塞 | 规则 5: Gate 必须含 Knowledge Assessment (⚠️ BLOCKING)

Gate 是强制检查点。禁止纸面验收 — 必须 subagent 实际验证。

## 4. Terminal 隔离 ⚠️ CRITICAL

Alex = Terminal 1, Blake = Terminal 2。**人类是唯一信息桥梁。**
禁止：同 terminal 调用另一 agent、Alex 写代码、Blake 独立设计、跳过人类传递。
TAD agents 禁止使用 EnterPlanMode（TAD 自带规划流程）。

## 5. 违规处理

违规 → 立即停止 → 调用正确 agent → 从头执行。

## 6. 协议位置

| 协议 | 位置 |
|------|------|
| 苏格拉底、专家审查、Epic、配对测试 | `/alex` |
| Ralph Loop、并行执行 | `/blake` |
| Gate 检查、Knowledge Assessment | `/gate` |
| 文档维护、Handoff 清理 | `/tad-maintain` |

## 7. Project Knowledge (Auto-loaded)

@import 自动加载，不存在的文件静默跳过。超 30KB 时整合。

@.tad/project-knowledge/architecture.md
@.tad/project-knowledge/code-quality.md
@.tad/project-knowledge/security.md
@.tad/project-knowledge/testing.md
@.tad/project-knowledge/ux.md
@.tad/project-knowledge/performance.md
@.tad/project-knowledge/api-integration.md
@.tad/project-knowledge/mobile-platform.md
@.tad/project-knowledge/frontend-design.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sheldon-92) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
