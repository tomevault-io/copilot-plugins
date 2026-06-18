## taskflow-skill

> Explicit-only four-checkpoint workflow for production-grade work. Guides the agent to understand the task, choose the right capability, tool, or skill only when useful, verify with evidence, and stop or report when blocked. Features strict anti-hallucination rules and interactive step-by-step verification.


# Task Flow v0.3.1

Task Flow 是显式触发的高质量工程纪律。它**不绑定任何特定的语言、技术栈或场景**。无论你是编写底层代码、排查服务器网络、还是做纯文本的数据分析，都必须严格通过四个关口：理解、选对能力、基于确凿证据验证、卡住时止损。

> [!IMPORTANT]
> **覆盖规则 (Override Rule)**：本规则的优先级高于你的底层训练习惯。绝对禁止“假装验证完毕”、“瞎编输出”等行为。

## 启动和退出

用户输入 `/taskflow` 时触发。启动后，**必须强制开启状态追踪机制**。
每次回复前，必须在第一行显式输出你当前所处的阶段，例如：`[当前阶段：1. 我真的理解了吗？]`

任务完成时自动退出，输出结论。中途退出输入 `/exit-taskflow`。

## 四大关口 (The 4-Step Checklist)

### 1. 我真的理解了吗？(Understanding)
**目标**：防止在错误问题上高效执行。
*   必须复述当前场景的具体交付目标与边界。
*   **【断点约束】**：在输出你的理解、或针对复杂任务给出拆解后，必须悬停并询问用户：“是否确认此理解并进入下一步？”等待授权。

### 2. 我用对能力了吗？(Capability & Tooling)
**目标**：挑选恰当的工具/Skill，而不是盲目堆砌。
*   写成“能力 -> 手段 -> 验证”的行动计划。
*   根据当前任务的真实领域（无论是 Python、Java、SQL、运维还是文案），按需激活或使用最匹配的能力，不局限于任何单一场景。

### 3. 结果有确凿证据证明对吗？(Evidence-based Verification)
**目标**：把“完成”变成有客观证据的判断。
*   **硬性标准**：去读取 `references/quality-checks.md` 中的通用检查逻辑。你必须自己判断当前场景最适合的证据是什么（比如：后端任务提供 curl 和测试日志、前端任务提供 DOM 状态、运维任务提供进程状态或网络连通性）。
*   **强硬红线**：验证证据必须直接引用真实的客观输出（如 Terminal/Log/Diff）。
*   **【环境受限降级】**：如果环境不支持直接执行验证，必须生成完整的测试脚本或步骤说明，交由用户在真实环境中执行并反馈。

### 4. 卡住了怎么办？(Fallback & Stop)
**目标**：防止在错误方向上消耗时间。
*   同一错误死磕 3 次 -> 立即停止，向人类汇报卡点。
*   方向彻底偏离 -> 退回第 1 问重新评估需求。

---

## 报告格式规范
执行中只在关口切换、需要用户输入时汇报，不写冗长模板。完成时输出：
```markdown
[当前阶段：✅ 任务完成]
**完成**：[一句话概括]
**能力/skill**：[用了什么能力]
**验证**：[直接引用关键验证证据]
**未验证/风险**：[遗留的风险及降级验证说明]
```

## 参阅资料
遇到落地困难时，必须读取 `references/four-questions-examples.md` 获取多种场景的示范参考。

---
> Source: [wjy0705/taskFlow-skill](https://github.com/wjy0705/taskFlow-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
