## botrundocs

> 建立 Botrun 服務的文件網站，讓 AI Agent（Claude Code、Gemini、ChatGPT 等）能正確回答使用者關於 Botrun 服務的問題。

# BotrunDocs 專案指引

## 專案目的
建立 Botrun 服務的文件網站，讓 AI Agent（Claude Code、Gemini、ChatGPT 等）能正確回答使用者關於 Botrun 服務的問題。

## 文件索引

### 執行類
- **執行計劃：** `docs/執行計劃.md`

### 設計類
- **每次更新流程：** `docs/設計-每次更新流程.md` — 三層管線 + 更新步驟 + 紀錄機制

### 現況類
- **現況分析：** `docs/現況-Botrun線上資源與AI可見性分析.md`

### 研究類
- **AI 搜尋機制：** `docs/研究-AI搜尋機制與爬蟲體系.md`
- **GEO 最佳化：** `docs/研究-GEO生成引擎最佳化.md`
- **llms.txt 與 Schema：** `docs/研究-llms-txt與Schema結構化資料.md`
- **策略實證查證：** `docs/研究-AI搜尋策略實證查證.md`

### 專案資料
- **關鍵資料：** `docs/project_notes/key_facts.md`
- **前台原始碼：** `/Users/chenwei/Documents/GitHub/botrun_front`

### 知識來源
- **CTO 技術洞見部落格：** https://botrun.github.io — 「五言波特人」，含戰略規劃、產品架構、研究分析等文章（採集紀錄：`raw-sources/2026-03-18/cto-blog-botrun-github-io.md`）
- **CTO Medium：** https://medium.com/@bohachu — 技術文章（採集紀錄：`raw-sources/2026-03-11/medium-articles.md`）

## 核心架構：三層內容管線

```
Layer 1：raw-sources/     原始擷取（純資料傾倒，不做判斷）
    ↓
Layer 2：insights/        洞見整理（跨來源比對 + 變更建議）
    ↓
Layer 3：site/            HTML 網站（唯一編輯點，直接改 HTML）
```

- 詳見：`docs/設計-每次更新流程.md`
- 更新循環：對照提問列表 → 採集 → 洞見 → 更新 HTML → git commit → 驗證
- **最高原則：** `_user-questions.md`（使用者提問列表，人機共編，指導所有內容方向）
- **已移除 content/ 目錄**：不再有 Markdown → HTML 生成流程，site/ 就是唯一真相

## 目錄結構
```
BotrunDocs/
├── _user-questions.md ← 使用者提問列表（最高原則）
├── site/              ← 唯一編輯點（直接改 HTML）
│   ├── *.html             各頁面 HTML
│   ├── llms.txt           AI Agent 索引（由 /build 產生）
│   ├── llms-full.txt      完整內容純文字（由 /build 產生）
│   ├── sitemap.xml        搜尋引擎索引（由 /build 產生）
│   └── robots.txt         爬蟲規則（由 /build 產生）
├── docs/              ← 專案文件（計劃、設計、研究，在 .gitignore 中）
│   └── project_notes/
├── updates/           ← 每次更新的紀錄總結（永久保留）
├── raw-sources/       ← Layer 1：原始擷取（本地暫存，不進 git）
├── insights/          ← Layer 2：洞見整理（本地暫存，不進 git）
└── dev-history/
```

## 技術決策
- 純靜態 HTML + CSS（AI 爬蟲 100% 可讀）
- llms.txt + Schema.org JSON-LD（AI Agent 最佳化）
- 繁體中文（台灣用語）
- Firebase Hosting → docs.botrun.ai（自訂域名已設定）
- 資料更新是核心：有新來源即觸發，三層管線確保品質
- 內容方向：使用者提問列表驅動，場景導向（非功能導向）
- 三個 TA：部會使用者、Bot 建立者（Creator）、API 串接工程師
- **直接編輯 HTML**：不再使用 Markdown 中間層，site/ 是唯一真相

## 部署

- **託管：** Firebase Hosting（GCP 專案 `scoop-386004`，Site ID `botrun-docs`）
- **網址：** https://docs.botrun.ai
- **身分驗證：** gcloud ADC（不需要 firebase login，用 `gcloud auth application-default login`）
- **部署指令：** `firebase deploy --only hosting --project scoop-386004`
- **部署 Skill：** 在 Claude Code 中輸入 `/deploy`
- **設定檔：** `firebase.json`、`.firebaserc`
- **AI 可見性改善計畫：** `docs/TODO-AI可見性改善計畫.md`

## 注意事項
- **直接編輯 site/*.html**（HTML 就是唯一真相）
- 改完 HTML 後執行 `/build` 重新產生 llms.txt、sitemap.xml 等 AI 檔案
- 部署前執行 `/build`，部署用 `/deploy`
- raw-sources/ 只放純原始資料，不做判斷
- insights/ 記錄跨來源比對和變更決策
- 參考 botrun_front 原始碼確保功能描述與實際一致
- git commit 結構化訊息作為 changelog

---
> Source: [botrun/BotrunDocs](https://github.com/botrun/BotrunDocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
