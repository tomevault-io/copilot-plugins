## xinchoujs

> **保持代码风格一致性**：参考现有代码（如 `src/main.ts`）的缩进、命名、注释风格，使用 ESLint/Prettier 等工具进行代码格式化。


**保持代码风格一致性**：参考现有代码（如 `src/main.ts`）的缩进、命名、注释风格，使用 ESLint/Prettier 等工具进行代码格式化。
2.  **模块化设计**：继续保持服务层（`services/`）、数据访问层（`db/`）、UI 层（`ui/`）的职责分离。
3.  **TypeScript 强类型**：充分利用 TypeScript 的类型系统，定义清晰的接口和类型，提高代码可维护性和健壮性。
4.  **错误处理**：在异步操作（如数据库访问、IPC 通信）中，确保有完善的错误捕获和处理机制，将错误信息友好地反馈给用户或记录到日志。
5.  **IPC 通信规范**：Electron 主进程和渲染进程之间的通信（IPC）应定义清晰的通道名称和数据格式，避免直接暴露敏感信息或功能。
6.  **数据库操作**：使用 ORM 或封装好的数据访问层进行数据库操作，避免在业务逻辑中直接编写 SQL。
7.  **单元测试**：为核心业务逻辑和服务编写单元测试，确保功能的正确性。
8.  **性能优化**：关注数据量较大时的性能问题，例如批量计算、数据导入导出等，考虑使用 Web Workers 或优化数据库查询。
9.  **安全性**：虽然是本地应用，但仍需注意文件路径处理、用户数据存储等方面的安全。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/25283531) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
