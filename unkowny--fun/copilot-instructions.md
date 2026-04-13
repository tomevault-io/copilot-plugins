## fun

> 这是一个零依赖的纯前端单文件静态网站项目 —— **ASCII Camera**。使用浏览器 `getUserMedia` API 捕获摄像头画面，并在 Canvas 上实时渲染为 ASCII 字符画。

# AGENTS.md

## Cursor Cloud specific instructions

### 项目概述

这是一个零依赖的纯前端单文件静态网站项目 —— **ASCII Camera**。使用浏览器 `getUserMedia` API 捕获摄像头画面，并在 Canvas 上实时渲染为 ASCII 字符画。

仓库仅包含两个文件：
- `camera.html` — 全部代码（HTML/CSS/JS 内联在一个文件中）
- `CNAME` — GitHub Pages 自定义域名配置（`fun.lazy-people.com`）

### 开发服务器

无需安装任何依赖。启动本地开发服务器：

```bash
python3 -m http.server 8000
```

然后在浏览器中打开 `http://localhost:8000/camera.html`。

### 注意事项

- **无构建步骤、无包管理器、无 lint 工具、无自动化测试**。项目是纯静态 HTML 文件。
- 摄像头功能需要在 `localhost` 或 HTTPS 环境下才能使用（浏览器安全策略要求）。
- Cloud VM 环境没有物理摄像头，点击 "START LIVE" 会弹出中文提示 "无法启动摄像头，请检查权限。"，这是预期行为。
- UI 交互（设置面板、滑块、颜色模式切换等）在无摄像头环境下仍可正常测试。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unkownY)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/unkownY)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
