## mathmodel-workflow

> 这是数模 skill 的源仓库，不是比赛项目。解释、维护、安装与测试本仓库时不启动数模实验流程，不创建 Qx 或思考记录。

# 发布仓库维护说明

这是数模 skill 的源仓库，不是比赛项目。解释、维护、安装与测试本仓库时不启动数模实验流程，不创建 Qx 或思考记录。

- 唯一内容源在 `skills/`；reviewer 规格在主 skill 的 `references/reviewers/`。
- `adapters/` 由 `python3 scripts/manage.py build` 生成，不能手工修三份。
- 内容修改后运行 validate、tests；不把静态检查当成端到端 agent 验证。
- 不修改用户的模型、凭据或权限设置；安装冲突先检查，更新仅管理本套件。
- 发布前核对第三方来源、许可证和测试报告；不自动推送远程。

---
> Source: [smallpinksquare/mathmodel_workflow](https://github.com/smallpinksquare/mathmodel_workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
