## ztools

> - 新增或实质修改的方法、函数，必须在声明正上方使用多行 JSDoc，简要说明职责。

# AGENTS.md

## 注释规范

- 新增或实质修改的方法、函数，必须在声明正上方使用多行 JSDoc，简要说明职责。
- JSDoc 必须为每个参数提供 `@param`，并提供 `@returns`；方法可能主动抛错时补充 `@throws`。
- `void`、`Promise<void>` 方法也需要写明 `@returns`，例如“无返回值”或“操作完成后结束的 Promise”。
- 方法内部应在准备、校验、状态切换、资源发布、回滚、清理等关键步骤前添加单行注释。
- 单行注释应解释步骤目的、约束或失败边界，不要逐行复述代码行为。
- 修改实现时同步更新相关注释，禁止保留与当前行为不一致的旧注释。

## E2E 验证

- 使用 Playwright Electron 运行端到端测试：`pnpm test:e2e`。
- 命令会先构建 ZTools 主程序，再由 Playwright 自动启动设置插件开发服务器（`127.0.0.1:15177`）和隔离的 Electron 测试实例，无需手动启动服务。
- E2E 使用临时数据目录，不得读写真实的 `~/.ztools`；新增用例必须继续传入隔离的数据与旧数据目录。
- 排查插件界面时直接截取插件 WebContentsView，不要与宿主窗口截图拼接。
- 默认只截取当前可见区域；只有测试明确要求时才增加全页面或滚动截图逻辑。
- 截图前必须等待页面正文中的稳定特征出现，不能只等待侧栏、标题等可能提前渲染的内容。
- 测试截图和 trace 输出到 `test-results/playwright`，该目录不提交版本库。
- Electron E2E 使用 Electron 自带的 Chromium，不需要执行 `playwright install chromium`。

---
> Source: [ZToolsCenter/ZTools](https://github.com/ZToolsCenter/ZTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
