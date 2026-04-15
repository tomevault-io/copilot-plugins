## urlshortener

> 純前端單檔縮網址工具（index.html），零依賴，無需建置。同時呼叫多個縮網址服務 API，API 失敗時自動降級為手動開啟連結。

# CLAUDE.md — urlShortener

## 專案概述
純前端單檔縮網址工具（index.html），零依賴，無需建置。同時呼叫多個縮網址服務 API，API 失敗時自動降級為手動開啟連結。

## 開發原則
- **維持單一 HTML 檔案**，不引入任何外部依賴
- 所有動態內容（API 回應、使用者輸入）必須用 DOM 方法（createElement、textContent、href）而非 innerHTML 插入
- 版本號需同步更新：`<title>`、h1 small 標簽、README.md Changelog

## 安全規則
- **禁止** 將 API 回應（short URL）插入 innerHTML — 用 `createShortUrlRow()` 替代
- **禁止** 在 innerHTML 中用 onclick 屬性連結含動態資料的事件 — 用 addEventListener
- 所有 `target="_blank"` 連結必須加 `rel="noopener noreferrer"`

## 已知風險與對策
| 風險 | 對策 |
|------|------|
| XSS via API 回應 | createShortUrlRow / createOpenLinkRow 用 DOM 方法 |
| 按鈕提前解鎖 | Promise.allSettled(fetchPromises) 傳入實際 promise 陣列 |
| localStorage QuotaExceededError | saveHistory 包在 try/catch 內 |
| 手機自動修正 URL | input 加 autocorrect/autocapitalize/spellcheck 屬性 |

## 不要做的事
- 不要用 `urlDiv.innerHTML = \`...\${short}...\`` — short 來自第三方 API，有 XSS 風險
- 不要 `Promise.allSettled([])` 傳空陣列 — 要傳 fetchPromises 陣列
- 不要在 README 中列出未實作的功能

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyhbench) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
