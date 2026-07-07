## mac-video-transfer-web

> 跨平台規則正本：`~/.agents/institution/`（先讀 core/PRINCIPLES.md，照其指示附版本標記）。

# mac-video-transfer-web — 薄索引
跨平台規則正本：`~/.agents/institution/`（先讀 core/PRINCIPLES.md，照其指示附版本標記）。

## 專案專屬
- Build/test 指令：無 package.json；本地啟動用 `python3 -m http.server 8080` 或 `npx http-server -p 8080`
- 架構一句話：純靜態網頁（HTML+CSS+JS），使用 FFmpeg WASM 在瀏覽器端壓縮影片，無伺服器。
- 本專案禁區：ffmpeg/ 目錄為 WASM 二進位，不要手動修改。

---
> Source: [miku4ocean/mac-video-transfer-web](https://github.com/miku4ocean/mac-video-transfer-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
