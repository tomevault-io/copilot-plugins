## ai-fit-room

> - 類型：Chrome MV3 擴充功能（使用 `sidePanel` 與 `contextMenus`）。

# Repository Guidelines

## 專案結構
- 類型：Chrome MV3 擴充功能（使用 `sidePanel` 與 `contextMenus`）。
- 重要檔案：
  - `manifest.json`
  - `src/background/background.js`（背景服務工作執行緒）
  - `src/sidepanel/index.html`、`src/sidepanel/index.js`（側邊欄 UI 與邏輯）
  - `src/content/content_script.js`（可選，供頁面互動觸發訊息）

## 載入與開發
- 無建置流程，直接以「未封裝」載入：
  1) 前往 `chrome://extensions`
  2) 開啟「開發人員模式」
  3) 點擊「載入未封裝項目」，選取此倉庫根目錄
- 變更檔案後在擴充功能管理頁面點「重新載入」即可

## 主要功能
- 右鍵圖片顯示自訂選單，只在 `img` 上顯示；點擊後：
  - 以使用者手勢同步開啟 side panel（全域共享同一個面板）
  - 將圖片來源（`srcUrl`）寫入 `chrome.storage.local.lastImageContext`
- Side panel：
  - 上傳基底圖（`lastImage`）：支援檔案上傳、OS 原生貼上、複製 Base64 與圖片到剪貼簿
  - 右鍵載入覆蓋圖（`overlayImage`）：與基底圖分離顯示、支援複製與「清除圖片」
  - 自動同步：監聽 `storage.onChanged`，偵測 `lastImageContext` 後自動抓圖並儲存為 `overlayImage`
  - 自動合成：當同時具備基底圖、覆蓋圖與 API 設定時，自動呼叫合成
  - 合成結果：顯示結果圖片，提供「下載圖片」與「複製圖片」

## 事件流程（概要）
1. 背景：使用者於圖片上右鍵 → context menu onClicked
   - `chrome.sidePanel.open({ tabId })`
   - `chrome.storage.local.set({ lastImageContext: { srcUrl, pageUrl } })`
2. Side panel 啟動：
   - 啟動時讀取 `overlayImage` 以復原右鍵區塊
   - 監聽 `storage.onChanged`：
     - 若收到 `lastImageContext` → 抓取圖片 → 寫入 `overlayImage`
     - 若 `overlayImage` 更新 → 更新 UI，並嘗試自動合成
3. 自動合成條件：同時存在 `lastImage.dataUrl`、`overlayImage.dataUrl`、以及 `apiConfig.endpoint`、`apiConfig.key`

## API（Google Gemini generateContent）
- 設定項目：在面板中輸入 Endpoint、API Key、提示詞（Prompt），儲存在 `apiConfig`
- 請求內容：`parts` 包含文字（可選）與兩張影像（基底＋覆蓋）
- 欄位格式：目前送出 snake_case（`inline_data`/`mime_type`）；解析回應同時支援 `inlineData` 與 `inline_data`
- Host 權限：`https://generativelanguage.googleapis.com/*`

## UI 行為（重點）
- 基底圖與右鍵覆蓋圖分離管理：右鍵圖不覆蓋上傳圖
- 右鍵覆蓋圖區塊提供「清除圖片」
- 合成結果按鈕：
  - 「下載圖片」（原 `copy-compose-base64` 按鈕）：依 MIME 自動決定副檔名
  - 「複製圖片」：使用 OS 原生剪貼簿 API 複製圖片 Blob

## 儲存鍵（storage.local）
- `lastImage`：基底圖 `{ name, type, dataUrl }`
- `overlayImage`：覆蓋圖 `{ name, type, dataUrl, srcUrl }`
- `lastImageContext`：背景寫入的右鍵圖片來源 `{ srcUrl, pageUrl }`
- `apiConfig`：`{ endpoint, key, prompt }`
- `composedImage`：最後一次合成結果（data URL）

## 權限與注意事項
- `permissions`：`sidePanel`、`contextMenus`、`storage`、`clipboardRead`、`clipboardWrite`
- `host_permissions`：`https://generativelanguage.googleapis.com/*`
- CORS：部分站點圖片可能無法跨網域抓取，面板會提示失敗並顯示來源 URL
- 使用者手勢：`chrome.sidePanel.open()` 與讀寫剪貼簿需在明確手勢內觸發

## 程式風格與提交
- JavaScript（無 TypeScript），2 空白縮排，函式/變數使用 camelCase
- Commit 採用簡潔、有意義的訊息（可參考 Conventional Commits）
- PR／變更請附上：目的、影響範圍與簡短測試方式

---
> Source: [Jakevin/ai-fit-room](https://github.com/Jakevin/ai-fit-room) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
