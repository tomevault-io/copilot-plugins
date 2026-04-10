## ai-dev-runtime-protocol

> 这个仓库是中文主导、双语术语的 AI 开发运行协议。

# AGENTS

这个仓库是中文主导、双语术语的 AI 开发运行协议。

## 工作规则

- 大改之前先读 [ONBOARDING.md](ONBOARDING.md)
- 优先遵守 [docs/playbooks/stage-router-阶段路由.md](docs/playbooks/stage-router-阶段路由.md)
- 优先生成 `.aidrp/` 工件，而不是直接大范围扫仓库
- 想法不清楚时，先做 `需求访谈（discovery interview）`，再写 `需求简报（requirement-brief）`
- 功能开发先过 `产品评审（product review）`，再用 `仓库地图（repo-map）` 和 `工程评审（engineering review）` 缩小改动边界
- 前端或多端 UI 任务先补 `设计令牌包（design-token-pack）`
- 非 trivial 任务先生成 `任务包（task-packet）`
- bug 先生成 `排障包（debug-pack）` 和 `可观测性关联（observability-correlation）`，再做 `根因调查（investigate）`
- 排障优先按 `trace_id / decision_id / request_id / plan_id / tool_call_id` 聚焦日志，不要先大范围扫代码
- 判断方向变了，就写 `决策轨迹（decision-trace）`
- 影响用户路径、CLI、API、文件输出的改动，必须做 `真实验收（live QA）`
- 真实 bug 修完后，补 `回归用例（eval-case）`
- 非 trivial 改动收尾前，必须补 `文档同步包（doc-sync）`
- 如果命令表面、阶段顺序或系统定位变了，不允许只往 README 追加一段，必须按章节或整篇重写
- 提交信息默认使用中文，并按 [docs/playbooks/git-commit-提交规范.md](docs/playbooks/git-commit-提交规范.md) 编写
- 吸收开源项目思路时，按 [docs/reference/open-source-citation-开源引用规范.md](docs/reference/open-source-citation-开源引用规范.md) 明确写清引用和落地位置

## 验证

- `python -m unittest discover -s tests -v`
- `python -m aidrp repo-map --project-root . --output-dir .aidrp`
- `python -m aidrp --help`

## 编辑边界

- 运行时保持无第三方依赖
- 优先 JSON 工件和确定性输出
- 核心逻辑避免绑定特定模型平台

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BobbyCats)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BobbyCats)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
