## codex-context-used-meter

> 这个仓库是基于 [Codex++](https://github.com/BigPizzaV3/CodexPlusPlus) 的用户脚本，不是 Codex 原生插件，也不是 Codex 插件市场里的插件。只有先安装并启用 Codex++，再把这个仓库里的用户脚本放到 Codex++ 用户脚本目录后才会生效：

# AGENTS.md

## 生效位置

这个仓库是基于 [Codex++](https://github.com/BigPizzaV3/CodexPlusPlus) 的用户脚本，不是 Codex 原生插件，也不是 Codex 插件市场里的插件。只有先安装并启用 Codex++，再把这个仓库里的用户脚本放到 Codex++ 用户脚本目录后才会生效：

```cmd
%APPDATA%\Codex++\user_scripts
```

在当前 Windows 用户下，这个路径也可写成：

```cmd
%USERPROFILE%\AppData\Roaming\Codex++\user_scripts
```

提交到远端的文档、脚本和示例命令应优先使用 `%APPDATA%` 形式，避免写入本机用户名或绝对用户目录。

## Agent 安装入口

当用户用下面这种方式指向本仓库时：

```text
请帮我安装：[Minghou-Lei/codex-context-used-meter](https://github.com/Minghou-Lei/codex-context-used-meter)
```

Agent 应先读取 `README.md` 里的“让 Agent 自动安装”章节，确认这是 Codex++ 用户脚本，不是 Codex 原生插件或插件市场插件，再执行本文件的部署命令。标准安装方式只有一个：把仓库根目录的 `codex-context-used-meter.js` 复制到 `%APPDATA%\Codex++\user_scripts\codex-context-used-meter.js`。

不要去 Codex 插件市场查找或安装，不要修改 Codex 全局配置，不要用 DevTools 临时代码替代安装，不要把脚本放到其它目录作为替代安装方式。除非用户明确要求 Provider 余额框，否则不要创建 Provider 配置、密钥文件或 supervisor。

## 部署命令

从仓库根目录更新本机 Codex++ 脚本时，使用：

```cmd
copy /Y codex-context-used-meter.js "%APPDATA%\Codex++\user_scripts\codex-context-used-meter.js"
```

PowerShell 等价命令：

```powershell
Copy-Item -LiteralPath .\codex-context-used-meter.js -Destination (Join-Path $env:APPDATA 'Codex++\user_scripts\codex-context-used-meter.js') -Force
```

## 修改约定

- 保持仓库内源码与 `%APPDATA%\Codex++\user_scripts` 下的实际生效脚本同步。
- 不要在可提交文件中硬编码本机用户目录绝对路径。
- 修改后至少确认目标脚本已复制到用户脚本目录，再验证 Codex++ 中的实际效果。
- push 到远端前必须检查本次提交内容是否包含隐私数据或本机敏感信息，例如真实用户目录、Token、密钥、邮箱、私网地址、带凭据的 URL。

---
> Source: [Minghou-Lei/codex-context-used-meter](https://github.com/Minghou-Lei/codex-context-used-meter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
