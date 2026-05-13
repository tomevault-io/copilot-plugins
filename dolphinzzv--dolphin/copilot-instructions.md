## dolphin

> > **所有变更必须严格遵循 `workflow/change-flow.md` 定义的 9 步流程。任何跳过步骤的行为均被禁止。**

# AGENTS.md — dolphin Development Guidelines

> **所有变更必须严格遵循 `workflow/change-flow.md` 定义的 9 步流程。任何跳过步骤的行为均被禁止。**

## Change Flow (概要)

```
1. 需求/Bug → todo/ 或 feature/ 编号归档
2. Agent 两轮自审需求（不清晰则追问）
3. 输出设计文档到 design/ 或写清方案
4. Agent 两轮自审设计（有问题则回退修改）
5. 创建 feature/bugfix 分支，严格按设计写代码
6. 单元测试 go test -race ./internal/... -count=1 100% 通过
7. Agent 两轮自审代码（逐条核对需求 + 边界情况 + 并发安全）
8. 提交代码 → 自评影响范围、回滚方案、兼容性
9. 询问用户：合并或改进
```

## Project Scope

dolphin 是一个跨终端/邮件/聊天/SSH 的 AI Agent，运行 shell 命令、控制浏览器、委派子 Agent、遵循定时任务。

## 核心约束

- 所有配置通过 `config.yaml` 或环境变量，不硬编码
- 使用 zap 结构化日志，关键路径 INFO 级别
- 代码必须稳定、可恢复、可观测、可测试
- 测试覆盖：整体 60%+，关键路径 80%+
- 测试带 race 检测：`go test -race ./...`

## 设计文档

- 架构设计见 `design/` 目录
- 新功能/跨组件变更必须先更新设计文档，再进入编码
- 设计文档路径：`design/modules/<模块名>.md`

## 安全

- 不硬编码凭据、密钥，使用环境变量或 `config.yaml` 的 secret 引用
- Shell 工具需显式白名单危险命令
- 所有外部输入需验证，记录拒绝原因

## 参考

- Change Flow: `workflow/change-flow.md`
- Git 分支策略: `workflow/git-branching.md`
- CI/CD: `workflow/ci-cd.md`
- 架构设计: `design/README.md`
- panda 客户端: `./app/panda/AGENTS.md`

---
> Source: [dolphinZzv/dolphin](https://github.com/dolphinZzv/dolphin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
