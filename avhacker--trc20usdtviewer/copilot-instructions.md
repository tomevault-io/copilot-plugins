## trc20usdtviewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

這是一個輕量級的單頁網頁應用程式，用於查詢和顯示 TRON 區塊鏈上特定地址的 TRC20 USDT 交易紀錄。整個應用程式是純前端實作，沒有後端伺服器，直接透過 TronGrid API 獲取資料。

## 專案架構

### 檔案結構
- `trc20usdtviewer.html` - 主要 HTML 文件，包含頁面結構
- `js/trc20usdtviewer.js` - 核心邏輯與功能實作
- `css/trc20usdtviewer.css` - 樣式定義

### 技術棧
- 純 HTML/CSS/JavaScript（無框架）
- [Pico.css](https://picocss.com) - 用於基礎樣式框架
- TronGrid API - 用於查詢 TRON 區塊鏈資料

## 核心功能架構

### 1. 多語言系統
- 支援繁體中文（zh-TW）和英文（en）
- 語言配置在 `translations` 物件中（js/trc20usdtviewer.js:21-136）
- 自動偵測瀏覽器語言，優先使用 URL 參數 `lang`
- 透過 `toggleLanguage()` 函數切換語言

### 2. 交易資料載入
- 使用 `fetchAndDisplayTransactions()` 函數（js/trc20usdtviewer.js:635）
- TronGrid API 每次最多返回 200 筆，透過分頁自動載入更多資料
- 預設載入上限為 10,000 筆記錄，可透過 `maxRecords` 變數調整
- 支援暫停/繼續載入功能

### 3. 過濾與排序系統
- 即時過濾條件：
  - 金額範圍（最小/最大金額）
  - 對方地址關鍵字（支援包含/排除模式）
  - 交易類型（收款/付款/全部）
  - 時間範圍過濾
- 排序功能支援時間、類型、金額欄位
- 所有過濾與排序條件會自動同步到 URL 參數

### 4. URL 參數系統
所有查詢條件和 UI 狀態都會儲存在 URL 參數中，方便分享：
- `address` - 查詢的錢包地址
- `min` / `max` - 金額範圍
- `peer` / `peerType` - 對方地址過濾
- `action` - 交易類型
- `startTime` / `endTime` - 時間範圍
- `sortField` / `sortOrder` - 排序設定
- `lang` - 語言設定

### 5. 時間顯示模式
支援三種時間顯示模式（可點擊表頭切換）：
- 本地時間（local）- 顯示使用者時區的時間
- UTC 時間（utc）- 顯示 UTC 標準時間
- 經過時間（age）- 顯示距離現在的時間差（如「3天5小時之前」）

## API 使用

### TronGrid API 端點
1. **查詢交易紀錄**
   - `GET https://api.trongrid.io/v1/accounts/{address}/transactions/trc20`
   - 參數：`limit`, `max_timestamp`

2. **查詢帳戶餘額**
   - `GET https://api.trongrid.io/v1/accounts/{address}`
   - 回傳 TRX 和 TRC20 代幣餘額

3. **查詢能量資源**
   - `POST https://api.trongrid.io/wallet/getaccountresource`
   - 回傳帳戶的能量限制和使用量

### USDT 合約地址
- TRC20 USDT: `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`

## 開發注意事項

### 狀態管理
主要全域變數（js/trc20usdtviewer.js:1-18）：
- `isPaused` - 控制載入暫停狀態
- `totalRecords` - 已載入的交易筆數
- `allTransactions` - 所有已載入的交易資料陣列
- `sortField` / `sortOrder` - 當前排序設定
- `timeDisplayMode` - 時間顯示模式
- `currentLanguage` - 當前語言設定

### 時間處理
- 預設時間範圍：2019-01-01 到 2029-01-01（常數定義在 js/trc20usdtviewer.js:5-6）
- 時間輸入支援智能格式化，自動添加分隔符號
- `parseTimeInput()` 函數處理多種時間格式的解析

### 複製功能
- 所有重要數值都支援點擊複製到剪貼簿
- 使用 `copyToClipboard()` 函數，會顯示提示訊息

### CSS 自訂變數
```css
--primary: #1095c1
--primary-hover: #0d7ea5
--received-color: #2ecc71
--sent-color: #e74c3c
```

## 修改建議

### 新增篩選條件
在 `sortAndFilterTransactions()` 函數（js/trc20usdtviewer.js:369）中新增過濾邏輯，記得同步更新：
1. HTML 表單元素
2. URL 參數處理（`updateURLParams()` 和 `loadURLParams()`）
3. 語言翻譯字串

### 新增 API 功能
在 `fetchAndDisplayTransactions()` 函數中處理新的 API 資料，確保：
1. 處理 API 錯誤回應
2. 更新狀態訊息
3. 考慮分頁載入邏輯

### 多語言擴充
在 `translations` 物件中新增語言：
1. 新增語言代碼和完整翻譯字串
2. 更新 `detectLanguage()` 函數的語言偵測邏輯
3. 測試所有 UI 元素的翻譯顯示

## 測試要點

- 測試不同網路地址的交易資料載入
- 驗證所有過濾條件的組合使用
- 確認 URL 參數的正確儲存與載入
- 測試多語言切換的完整性
- 檢查時間過濾器的各種輸入格式
- 驗證暫停/繼續載入功能

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avhacker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
