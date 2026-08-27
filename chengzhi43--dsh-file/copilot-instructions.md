## dsh-file

> 当项目主分支（master）有功能新增或 bug 修复时，必须更新项目版本号（`package.json` 的 `version` 字段，以及 README 徽章中的版本号）。

# AGENTS.md

## 版本号约束

当项目主分支（master）有功能新增或 bug 修复时，必须更新项目版本号（`package.json` 的 `version` 字段，以及 README 徽章中的版本号）。

版本号遵循**语义化版本（SemVer）**规范，格式为 `X.Y.Z`（主版本号.次版本号.修订号）：

- **主版本号（Major）**：做了不兼容的 API 修改时递增。
- **次版本号（Minor）**：做了向下兼容的功能性新增时递增。
- **修订号（Patch）**：做了向下兼容的问题修正时递增。

递增主版本号时，次版本号和修订号归零；递增次版本号时，修订号归零。

## 项目结构

- `src/index.ts` — host 半（Node 进程，文件系统 RPC），tsc 构建为 `dist/index.js`
- `src/client/` — client 半（浏览器 React，侧边栏文件树 + 编辑器），esbuild 构建为 `dist/client.js`
- `docs/` — README 引用的图片资源（logo 等）
- `test/` — 测试（node:test）

## 常用命令

```sh
npm install          # 安装依赖
node build.mjs       # 构建 host + client
npm test             # 运行测试（node --test "test/*.test.ts"）
```

---
> Source: [chengzhi43/dsh-file](https://github.com/chengzhi43/dsh-file) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
