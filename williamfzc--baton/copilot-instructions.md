## baton

> 每次更新文件时，必须确认文件顶部的注释（File Header）是否准确反映了文件的当前功能。如果文件功能发生了重大变化或重构，请务必更新注释。

# Agent Rules

## 1. 文件头维护 (Header Maintenance)
每次更新文件时，必须确认文件顶部的注释（File Header）是否准确反映了文件的当前功能。如果文件功能发生了重大变化或重构，请务必更新注释。

## 2. 变更验证 (Change Validation)
每次完成代码变更后，必须运行以下命令进行验证：
- `bun run check`: 执行类型检查、代码检查（Lint）和格式检查。
- 如果有错误，必须立即修复，确保代码库始终处于健康状态。

## 3. 交互设计 (Interaction Design)
- 优先使用极简的交互方式（如数字选择）。
- 遵循 ACP (Agent Client Protocol) 协议标准，不随意拦截或修改协议约定的流程。

---
> Source: [williamfzc/baton](https://github.com/williamfzc/baton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
