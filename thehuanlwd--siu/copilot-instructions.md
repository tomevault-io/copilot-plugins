## siu

> - 工作目录：`F:\AI code\SIU`

# AGENTS.md

## 响应语言

始终使用简体中文回复用户。

## 运行环境

- 操作系统：Windows 10
- 终端：PowerShell
- 工作目录：`F:\AI code\SIU`

## 命令限制

- 严禁使用 `grep` 等 PowerShell 不支持的 Linux 指令。
- 搜索文件优先使用 `rg`；如果不可用，再使用 PowerShell 原生命令。
- 严禁使用终端命令删除、移动文件。
- 任何涉及删除、移动文件的请求，都需要明确告知用户，让用户手动操作。

## 项目结构

```text
demo/
  src/                     前端 UI 与交互
  functions/api/            Cloudflare Pages Functions
  server/core/              可迁移后端核心逻辑
  supabase/schema.sql       Supabase 表结构
  CLOUDFLARE_PAGES_DEPLOY.md
```

根目录文档：

- `INIT.md`：人类开发者初始化和运行说明。
- `SIU_Pages_Functions_开发计划.md`：Pages Functions 迁移开发计划。
- `AGENTS.md`：代理/AI 编码助手需要遵守的项目规则。

## 本地运行

```powershell
cd demo
npm run build
npx wrangler pages dev dist --ip 127.0.0.1 --port 8788
```

## 验证

```powershell
cd demo
npm run build
npx tsc --noEmit
```

## 环境变量

本地 Pages Functions 使用：

```text
demo/.dev.vars
```

该文件包含密钥，不能提交。

需要的变量：

```text
OPENAI_API_URL
OPENAI_API_KEY
OPENAI_MODEL
GITHUB_TOKEN
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
PROMPT_VERSION
DEBUG_AI_CONTEXT
```

## Git 注意事项

不要提交：

- `demo/.env`
- `demo/.dev.vars`
- `demo/.wrangler/`
- `demo/*.log`
- `demo/dist/`
- `demo/node_modules/`
- `.ps_history/`

提交前检查：

```powershell
git status --short --ignored
```

---
> Source: [thehuanlwd/siu](https://github.com/thehuanlwd/siu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
