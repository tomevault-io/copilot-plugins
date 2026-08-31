## meeting-room-board

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案現況

本專案目前只有 [spec.md](spec.md)，尚未有任何程式碼、套件設定檔或建置工具，因此暫無 build/lint/test 指令可記錄；待開始實作後需回來補充。

## 需求規格

[spec.md](spec.md) 是本專案（會議室預約看板）的權威需求規格，內容涵蓋目標、使用者、MVP 功能清單、明確的非目標、技術限制與驗收標準。實作前務必先閱讀，若範圍有變動也要同步更新該檔案。

實作時需牢記 spec.md 中的關鍵限制：
- 純前端應用，無後端伺服器或資料庫；預約資料僅儲存在瀏覽器 `localStorage`，不支援跨裝置/跨使用者同步。
- 不需登入或帳號機制；任何人皆可建立或取消任何預約，不驗證身份。
- 預約起始/結束時間僅能為整點（例如 10:00、11:00），不支援半點或任意分鐘數。
- 目標執行環境僅為桌面瀏覽器，手機/平板版型明確排除在範圍之外。

## 開發規範

- 本專案檔案結構固定為 `index.html` + `app.js` + `test.html` 三個檔案，禁止再拆分出其他檔案，禁止引入外部框架或函式庫（含 CDN 引入）。`app.js` 以一般 `<script src="app.js"></script>` 載入，不使用 ES module。
- 所有介面文字與程式註解一律使用繁體中文。
- 每次修改後，用一句話說明這次改了什麼。

---
> Source: [jameslee1113/meeting-room-board](https://github.com/jameslee1113/meeting-room-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
