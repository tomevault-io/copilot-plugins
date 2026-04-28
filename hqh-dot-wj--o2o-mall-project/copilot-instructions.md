## o2o-mall-project

> 开发认知协议 — 默认快速执行，出现信号时深度思考（涉及高风险系统、复杂 Bug、影响范围异常时触发）


# Dev Cognitive Flow（开发认知协议）

> 默认快速执行，出现信号才深度思考。

## 流程

```
Task → Fast Mode（默认） → Signal Detector
  ├─ 无信号 → Execute
  └─ 有信号 → Slow Mode → Decision → Execute → Feedback
```

## Fast Mode（默认，80% 任务）

三个快速检查后直接执行：

1. **理解正确？** 用户要解决什么问题？不是只看表面描述
2. **范围正确？** 是否在正确的层修改？（前端问题可能在后端）
3. **有明显风险？** 涉及支付/资金/权限/认证/删除/数据迁移 → 进入 Slow Mode

三项通过 → 直接执行。

## Signal Detector（6 个信号）

出现以下任一信号 → 进入 Slow Mode：

| # | 信号 | 示例 | 触发 |
|---|------|------|------|
| 1 | 信息不一致 | 给前端页面，但要改后端 | Scope Recheck |
| 2 | 影响范围异常 | 改一个字段，影响多模块 | Superpowers brainstorming |
| 3 | 任务描述不清 | 「帮我优化一下」 | 先提问再执行 |
| 4 | 高风险系统 | 支付/资金/权限/认证/删除/迁移 | 必须深度分析 |
| 5 | 复杂 Bug | 原因不明确 | Superpowers systematic-debugging |
| 6 | 系统行为异常 | 改一处，多个测试失败 | Superpowers systematic-debugging |

## Slow Mode（20% 任务）

按信号类型选择分析工具：

- 信号 1/3 → 暂停执行，重新确认需求
- 信号 2 → Superpowers brainstorming（影响分析 + 方案探索）
- 信号 4 → Superpowers brainstorming + systematic-debugging
- 信号 5/6 → Superpowers systematic-debugging（根因分析）

## Execution 护栏

- 最小修改：只改必要代码，不顺手重构
- 系统一致性：缓存/数据库/接口同步考虑
- 复杂度控制：遵循 `common/core.mdc` 的简洁优先原则

## Feedback

执行后验证：问题是否真正解决？（测试/日志/监控）
结果异常 → 重新进入 Slow Mode。

## 核心规则

1. 默认 Fast Mode
2. 只在信号出现时进入 Slow Mode
3. 理解可能错误 → 暂停执行，重新确认需求
4. 影响范围异常 → 重新评估架构影响
5. 修复不符合预期 → 升级思考深度
6. 任何架构决策必须回答：你的证据是什么？

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hqh-dot-wj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
