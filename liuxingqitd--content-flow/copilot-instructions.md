## content-flow

> - 任何任务只要修改了应用代码、配置或构建资源，完成前必须通过完整验证并执行 `npm run desktop:build`。

# 项目协作规则

## 本地桌面应用同步

- 任何任务只要修改了应用代码、配置或构建资源，完成前必须通过完整验证并执行 `npm run desktop:build`。
- 构建成功后，必须先退出正在运行的 ContentFlow，再用 `src-tauri/target/release/bundle/macos/ContentFlow.app` 替换 `/Applications/ContentFlow.app`；不能只完成源码修改或构建而跳过本地安装。
- 安装后必须校验本地应用与最新构建产物一致；构建、退出、安装或校验任一步失败，都不能宣称任务完成。
- 只有文档或规则文件变更、且不影响应用代码与构建产物时，可以不重新安装应用。

---
> Source: [liuxingqitd/content-flow](https://github.com/liuxingqitd/content-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
