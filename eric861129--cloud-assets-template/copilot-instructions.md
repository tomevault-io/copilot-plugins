## cloud-assets-template

> - **名稱**: Cloud-Assets

# 🤖 Gemini Project Context

## 📌 專案概述
- **名稱**: Cloud-Assets
- **目的**: 自動化 GitHub 圖床。
- **核心流程**: 上傳圖片 -> GitHub Action 觸發 `optimize.py` -> 轉 WebP + 刪除原圖 -> 更新 `index.html` -> 自動 Commit 回傳。

## 🛠️ 技術細節
- **主要腳本**: `optimize.py` (處理圖片轉換與 HTML/MD 生成)。
- **圖片品質**: WebP 設定為 Quality 80。
- **目錄結構**:
  - `/.github/workflows/`: 自動化腳本配置。
  - `/Blog/`: 存放部落格相關圖片。
  - `/Share/`: 存放一般分享圖片。

## 📜 開發規則
1. 嚴禁保留原始 `.jpg` 或 `.png` 檔案，必須由腳本統一轉換為 `.webp`。
2. 圖片路徑不應包含敏感資訊。
3. `index.html` 的生成依賴 `optimize.py` 中的 `generate_index_html` 函數。
4. 推送時應包含 `[skip ci]` 標記，避免 GitHub Actions 陷入無限迴圈（目前的 Workflow 已透過路徑過濾處理）。

## 🔗 外部依賴
- **CDN**: `https://cdn.jsdelivr.net/gh/eric861129/Cloud-Assets@main/`
- **UI**: Bootstrap 5.3.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eric861129)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eric861129)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
