## coding-workflow-toolkit-project-skill-discovery

> 本仓库提供 AI 辅助交付的通用工作流和 skills。

# 项目工作流

本仓库提供 AI 辅助交付的通用工作流和 skills。

开始任何从初始题目进入的编码或产品拆解任务前，先阅读 [coding-workflow.md](coding-workflow.md)，并优先阅读和使用 `skills/guided-delivery/SKILL.md`。它负责一问一答地收敛共识，并自动同时使用 `skills/delivery-report/SKILL.md` 记录交付；在用户明确“确认实施”后才进入实现。遵循以下顺序：

1. 澄清需求，或写出显式假设。
2. 定义最小范围、验收标准、非目标与时间盒计划。
3. 展示受影响文件和实施方案，等待用户确认。
4. 实现最小可验证切片。
5. 验证主链路、异常路径和相关系统边界。
6. 总结取舍、验证结果、剩余风险和下一步。

`guided-delivery` 的完整流程默认维护交付记录，并在结束时按任务类型生成报告。报告默认写入项目根目录下、以中文需求名命名的目录中的 `交付报告.md`，并在对话中给出摘要和文件链接。

## Skill 路由

- 从初始问题开始，需要澄清、共识、实施和审查的完整闭环：阅读并使用 `skills/guided-delivery/SKILL.md`，它自动联动 `skills/delivery-report/SKILL.md`。
- 需求仍模糊、需要确定 MVP 或取舍：阅读并使用 `skills/mvp-delivery/SKILL.md`。
- 在已有代码库交付一条端到端功能：阅读并使用 `skills/feature-slice/SKILL.md`。
- 完成功能后或提交前审查风险：阅读并使用 `skills/engineering-review/SKILL.md`。
- 运行异常、测试失败或回归问题：阅读并使用 `skills/debug-loop/SKILL.md`。
- 不经过 `guided-delivery` 但需要记录过程或生成最终报告：阅读并使用 `skills/delivery-report/SKILL.md`。

## 透明性与安全边界

- 在写入应用代码、创建依赖或运行变更命令前，先展示计划并等待用户确认。
- 不使用黑盒自动化代替工程判断；AI 输出是候选方案，必须自行审查和验证。
- 在系统边界验证外部输入、身份和授权；不吞掉错误。
- 不执行破坏性命令；本工作流不执行提交、推送、PR 或远端写操作。

---
> Source: [wjyzhixing/coding-workflow-toolkit-project-skill-discovery](https://github.com/wjyzhixing/coding-workflow-toolkit-project-skill-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
