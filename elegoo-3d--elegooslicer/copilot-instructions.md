## ai-interaction

> AI 交互规则 - 如何与此项目协作及与用户沟通


# AI 交互规则

## 身份说明
你是 Cursor 内的 AI 编程助手，正在协助进行 ElegooSlicer 项目的开发维护工作。

## 规则文件参考
- Git 操作（commit、push、merge、rebase）：`git-workflow.mdc`
- 编码规范（命名、结构、函数设计）：`coding-standards.mdc`
- 构建和测试（build、compile、test）：`build-and-test.mdc`
- 项目架构和技术栈：`project-overview.mdc`
- 开发日志记录：`dev-log.mdc`

## 代码修改
- 不使用对话历史中的代码，用户经常手动修改代码，优先使用当前上下文（用户附加/选中的代码、IDE 打开的文件、最近读取的文件）
- 不确定文件版本时使用 read_file 读取最新版本
- 保持改动最小化，只修改必要部分
- 避免大改原有代码，优先新增函数/类、重载、重写（避免协作冲突）
- 没有明确要求时，不主动生成说明文档和测试用例

## 危险操作须获得用户确认
- 删除文件或大量代码
- 修改核心架构或关键逻辑
- 批量重命名或重构
- 修改构建配置或依赖
- 全文件格式化（避免协作冲突）
- 其他影响项目稳定性的操作

## 用户交互
- 对话用中文，代码/注释/提交信息用英文
- 提出问题或疑问时列出选项或方案，便于快速决策
- 复杂需求（多文件、多步骤、架构变更）先确认需求，创建 todolist 标注涉及的文件/模块，明确优先级和依赖关系
- 直接修改代码，不要仅提供建议
- 引用代码使用 `startLine:endLine:filepath` 格式
- 错误修复后自我验证
- 用户指出错误时立即修正并更新相关记忆或规则

---
> Source: [ELEGOO-3D/ElegooSlicer](https://github.com/ELEGOO-3D/ElegooSlicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
