## astock-market-engine

> **禁止** 使用 `Start-Process -FilePath "npx"` 或 `Start-Process -FilePath "npm"`。

# AGENTS.md — 项目开发须知

## Windows PowerShell 注意事项

**禁止** 使用 `Start-Process -FilePath "npx"` 或 `Start-Process -FilePath "npm"`。
Windows 上 `npx`/`npm` 是 `.cmd` 批处理文件，`Start-Process` 直接调用会用文本编辑器打开而不是执行。

正确写法（任选一种）：

```powershell
# 方案 1：通过 cmd.exe 调用（推荐）
Start-Process -FilePath "cmd.exe" -ArgumentList "/c","npm","run","dev" -WorkingDirectory "$ProjectRoot\frontend" -WindowStyle Hidden

# 方案 2：显式使用 .cmd 后缀
Start-Process -FilePath "npx.cmd" -ArgumentList "next","dev","--port","3000" -WorkingDirectory "$ProjectRoot\frontend" -WindowStyle Hidden
```

## 启动服务

- 完整启动（前端+后端）：运行 `.\start-all.ps1`
- 仅重启前端：在 `frontend/` 目录下运行 `npm run dev`
- 仅重启后端：在项目根目录运行 `python -m backend.main`

---
> Source: [gergchen/astock-market-engine](https://github.com/gergchen/astock-market-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
