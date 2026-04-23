## starcode-snippets

> - 使用 TypeScript 类型系统确保代码安全

# 编码指南

## 通用规范
- 使用 TypeScript 类型系统确保代码安全
- 避免在代码中硬编码常量值，优先使用配置或常量定义
- 保持每个文件的功能单一，遵循单一职责原则
- 充分利用 VSCode API 提供的功能，避免重复实现

## 命名约定
- 文件名使用 camelCase 格式
- 类名使用 PascalCase 格式
- 方法和变量使用 camelCase 格式
- 接口名使用 I 前缀，如 `ISnippet`
- 私有成员使用下划线前缀，如 `_storage`
- 常量使用全大写下划线分隔，如 `DEFAULT_TIMEOUT`

## 代码风格
- 不使用分号结尾
- 使用单引号作为字符串定界符
- 缩进使用两个空格
- 大括号总是使用，即使是单行代码块。尤其是if语句，即使只有一行，也要写大括号
- 导出的函数和类应该有文档注释

## 错误处理
- 使用 try/catch 块处理异步操作可能的错误
- 对用户操作提供清晰的错误信息
- 对于关键操作应实现错误重试机制
- 使用日志记录错误，便于调试

## 异步编程
- 优先使用 async/await 语法，代码更清晰易读
- 避免嵌套 Promise 或回调
- 对于并发操作，使用 Promise.all 或 Promise.allSettled
- 对于长时间运行的操作（如云同步），提供进度反馈

## VSCode API 使用
- 遵循 VSCode 扩展开发最佳实践
- 使用 VSCode 提供的 API 而非自定义实现
- 对于 UI 操作，提供适当的反馈和状态更新
- 遵循 VSCode 命令命名约定
- 使用 `vscode.window.withProgress` 显示长时间操作的进度
- 合理使用 `vscode.commands.executeCommand` 调用内置命令

## Git 操作规范
- 使用 `simple-git` 库进行 Git 操作
- 对 Git 操作进行适当的错误处理和重试机制
- 在执行危险操作前进行备份
- 提供清晰的同步状态反馈给用户

## 数据迁移和版本兼容
- 新功能需要考虑向后兼容性
- 数据格式变更时提供迁移工具
- 支持多版本数据格式的导入导出
- 在迁移过程中提供数据备份选项

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StarCode000) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
