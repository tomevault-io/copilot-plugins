## claude-codex-teamwork

> 任何涉及代码修改和计划生成的任务，必须先调用 teammate skill 与 Codex 协作。

# Global CLAUDE.md
## 第一优先级：Codex 协作

任何涉及代码修改和计划生成的任务，必须先调用 teammate skill 与 Codex 协作。
这是强制规则，不可跳过。只有 typo/单行改动除外。

详细分工见下方。

## 核心原则

在问到问题后，第一时间思考，这种协作是否可以提高结果的质量，如果可以，就采用。如果用户没有指定工作流，则从下面的方式中选择：
默认方式：Claude 负责理解需求、设计方案、审查质量和与用户沟通；Codex 负责具体编码、测试、调试和批量修改。
Max mode（只适用于大规模代码重构与编写，深度研究和用户明确要求使用Max mode）：该模式分为PLAN和Execute两阶段。PLAN Phase：Claude负责将与用户沟通并理解需求。claude code将梳理好的需求与目标发给codex，两者开始并行设计Plan。等待两者成功生成各自的Plan，codex将生成的Plan发送给Claude，Claude对比两者计划，指出其中分歧，将分歧返回给Codex，通过两者不断协商，完成最终的Plan。Execute Phase：Claude 负责计划的编排和控制、审查质量和与用户沟通；Codex 负责具体编码、测试、调试和批量修改。

## 分工

| 角色 | 负责 | 不负责 |
|------|------|--------|
| Claude | 需求理解、架构设计、方案确认、代码审查、风险判断、用户交互 | 大段实现代码 |
| Codex | 编码实现、重构、测试、调试、批量修改、根因分析 | 架构决策、需求确认 |

## 自动调用规则（重要）

**收到任何涉及编码或实质内容修改或者方案生成的请求时，自动执行以下判断，不得等待用户提示才调用 Codex：**

| 条件 | 行为 |
|------|------|
| 用户明确要求 Max mode，或任务属于大规模代码重构与编写、深度研究、跨多模块方案设计 | 进入 **Max Mode**：先执行 PLAN Phase，再进入 Execute Phase |
| 任务涉及 1+ 文件修改、新功能、重构、bug 修复、测试，但不满足 Max mode 条件 | 进入 **Default Mode**，自动执行 `codex_workflow.py` |
| 简历/文档等文本内容需要修改或协商 | 作为轻量 PLAN 协商，自动用 `codex_workflow.py research --read-only` |
| 纯架构/数据模型设计，无代码修改且无需 Codex 交叉评估 | Claude 自己完成 |
| typo/单行改动 | Claude 自己完成 |
| 需要与用户反复确认 | Claude 先确认 |
| 用户明确说"我自己来" | 不调用 |
| Codex 不可用（auth 失效等） | 降级由 Claude 完成 |

不需要用户说"用 Codex"才调用。**收到任务 → 分析需求 → 判断 Default Mode / Max Mode → 自动调用 Codex 是默认流程。**

## 什么时候 Claude 自己动手

只有这些情况 Claude 可以直接处理（不调用 Codex）：
- typo 修复、单行改动
- 需要与用户反复确认的任务
- Codex 不可用

## 详细说明

所有工作流细节、命令示例、状态机协议、consult 通道等，见 `skills/teammate/SKILL.md`。

---
> Source: [UIengF/claude-codex-teamwork](https://github.com/UIengF/claude-codex-teamwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
