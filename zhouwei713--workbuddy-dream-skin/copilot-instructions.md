## workbuddy-dream-skin

> > **本文件面向 AI 协作者（Claude、Codex、Cursor 等）**：约束大模型在编辑本仓库时**不能破坏什么**、**必须先读什么**、**收工前必须跑什么**。

# WorkBuddy Dream Skin 维护说明（AI 协作者约束）

> **本文件面向 AI 协作者（Claude、Codex、Cursor 等）**：约束大模型在编辑本仓库时**不能破坏什么**、**必须先读什么**、**收工前必须跑什么**。
>
> 人类贡献者应该看 [`CONTRIBUTING.md`](./CONTRIBUTING.md)（PR 流程、preflight 门禁、如何加主题）而不是这里。
>
> **_This file is written for AI collaborators. Human contributors: please read [`CONTRIBUTING.md`](./CONTRIBUTING.md) instead._**

本文件供后续大模型和维护者使用。开始修改前请先阅读 `README.md`、`docs/ARCHITECTURE.md`、`docs/DEVELOPMENT.md` 和 `.github/AI/HANDOFF.md`。

## 不可破坏的边界

1. 保持外置 CDP 注入架构。
2. 禁止修改 WorkBuddy 安装目录、`WorkBuddy.exe`、`app.asar`、签名文件或更新程序。
3. CDP 必须只绑定本机回环地址。
4. 使用端口前必须确认监听进程就是目标 `WorkBuddy.exe`。
5. 装饰节点必须保持 `pointer-events: none`。
6. 恢复流程必须可用，重复注入必须保持幂等。
7. 不得让审计脚本发送消息、保存设置、创建内容、执行授权、升级账号或退出登录。

## 修改流程

1. 先用 `rg` 查找相关 CSS、脚本和已有兼容规则。
2. 使用 `apply_patch` 修改文本文件。
3. 用户已有图片、主题和审计产物应当保留。
4. 修改用户可见行为时更新 `CHANGELOG.md` 和 `VERSION`。
5. 运行 PowerShell 语法检查、Node.js 语法检查和载荷检查。
6. 启动或刷新换肤后必须运行 `verify-workbuddy-skin.ps1`。
7. 涉及页面或悬停状态时，执行对应专项审计。
8. 如果调测过程中启动了临时服务，任务结束前必须关闭。

## 验证最低要求

```powershell
$errors = @()
Get-ChildItem .\scripts\*.ps1 | ForEach-Object {
  [void][Management.Automation.Language.Parser]::ParseFile($_.FullName, [ref]$null, [ref]$parseErrors)
  $errors += $parseErrors
}
if ($errors.Count) { $errors | Format-List; exit 1 }

node --check .\scripts\injector.mjs
node .\scripts\injector.mjs --check-payload
.\scripts\verify-workbuddy-skin.ps1
```

根据修改范围追加下列审计：

```powershell
.\scripts\audit-workbuddy-hover.ps1
.\scripts\audit-workbuddy-composer.ps1
.\scripts\audit-workbuddy-scenes.ps1
.\scripts\audit-workbuddy-pages.ps1
.\scripts\audit-workbuddy-details.ps1
.\scripts\audit-workbuddy-settings.ps1
```

## 版本和交接

1. `VERSION` 必须与注入器报告的版本一致。
2. 新主题必须能够保存为命名预设，并通过一条命令切换。
3. 新增主题字段时要保持 Schema 1 的兼容转换。
4. 修改架构、运行目录、命令或已知限制时同步更新 `.github/AI/HANDOFF.md`。
5. 不要把原始参考仓库内容重新放回本项目。参考仓库位于相邻目录 `../Codex-Dream-Skin-main`。

---
> Source: [zhouwei713/WorkBuddy-Dream-Skin](https://github.com/zhouwei713/WorkBuddy-Dream-Skin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
