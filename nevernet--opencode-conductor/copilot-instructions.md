## opencode-conductor

> Conductor 工作流命令 - 识别 /conductor:* 并执行对应阶段


# Conductor 命令 (Context-Driven Development)

当用户输入以下任一模式时，进入 **Conductor 协议**，并严格按照项目根目录 `SKILL.md` 中的对应阶段执行。

## 命令与模式映射

| 用户输入模式 | 模式 | 执行阶段 |
|-------------|------|----------|
| `/conductor:setup` | SETUP | Phase 1：初始化 .conductor/ 与 product/tech/workflow 等配置 |
| `/conductor:newTrack` 或「开新需求/新 feature」 | NEW_TRACK | Phase 2：创建 tracks/{id}/spec.md、plan.md、metadata.json |
| `/conductor:review-plan` 或「review 计划/评审计划」 | PLAN_REVIEW | Phase 2.5：评审 SPEC/PLAN，输出 PLAN REVIEW REPORT |
| `/conductor:implement` 或「开始实现/implement」 | IMPLEMENT | Phase 3：按 plan 逐任务实现并更新进度 |
| `/conductor:status` | STATUS | Phase 4：输出当前 tracks 状态表 |
| `/conductor:revert` | REVERT | Phase 5：按 track/phase/task 回滚并更新状态 |
| `/conductor:review` | REVIEW | Phase 6：对照 product-guidelines 与 spec 做 Code Review 报告 |
| 仅一般项目问题 | CONTEXT_QUERY | Phase 0：加载并汇报 .conductor 上下文 |

## 执行要求

1. **先做模式检测**：根据上表判断本次请求属于哪种模式。
2. **始终先跑 Phase 0**：除 SETUP 外，先执行 Context Detection（检查 `.conductor/`、加载 product.md / tech-stack.md / workflow.md / tracks.md，汇报上下文摘要）。
3. **严格按 SKILL.md**：各阶段步骤、产出文件、输出格式以 `SKILL.md` 为准，不省略步骤。
4. **禁止反模式**：不跳过 setup、不无 plan 实现、不跳过任务验证、不忽略 product-guidelines、不跳过测试要求、不未 review 就 commit。

## 快捷参考

- 初始化：`/conductor:setup`
- 新需求：`/conductor:newTrack "需求简述"`
- 评审计划：`/conductor:review-plan`
- 开始实现：`/conductor:implement`
- 看进度：`/conductor:status`
- 回滚：`/conductor:revert [scope]`
- 代码评审：`/conductor:review`

完整协议与各阶段细节见项目根目录 **SKILL.md**。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nevernet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
