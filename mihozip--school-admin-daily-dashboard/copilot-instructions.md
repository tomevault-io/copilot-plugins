## school-admin-daily-dashboard

> 建立以 Google Sheet 為資料底座、Google Apps Script 為服務層的校務任務管理系統，安裝時可選擇處室與主任／組長職務，並支援管理台與直向電子紙看板。

# AGENTS.md — 校務行政每日任務管理系統

## 1. 專案目的

建立以 Google Sheet 為資料底座、Google Apps Script 為服務層的校務任務管理系統，安裝時可選擇處室與主任／組長職務，並支援管理台與直向電子紙看板。

## 2. 核心資料契約

`任務清單` 第一列欄位名稱是系統契約。不得任意刪除或重新命名：

1. 任務ID
2. 任務名稱
3. 類型
4. 狀態
5. 優先級
6. 截止日期
7. 截止時間
8. 下一步行動
9. 等待對象
10. 最近進度
11. 負責人
12. 負責人Email
13. 看板顯示
14. 顯示排序
15. 詳細連結
16. 建立時間
17. 更新時間
18. 完成時間
19. 封存

如需新增欄位，應加在既有欄位之後，並同步修改 `TASK_HEADERS`、列與物件轉換、安裝／遷移及管理台表單。

## 3. 處室 profile 契約

- 處室、職務、任務類型與範例集中在 `OFFICE_PROFILES`。
- 不得為新處室複製 CRUD、驗證、安全或看板流程。
- profile 的 `categories` 必須包含「其他」。
- 切換處室不得刪除既有任務；舊任務類型必須保留為有效選項。
- `OFFICE_KEY` 與 `ROLE_KEY` 是穩定代碼；顯示文字可調整，但不可任意變更既有代碼。

## 4. 禁止破壞項目

- 不得以實體刪除取代封存。
- 不得移除 `工作紀錄` 稽核。
- 不得讓電子紙看板寫入 Sheet。
- 不得要求把試算表設為公開。
- 不得在程式或 HTML 硬編碼試算表 ID、密碼、OAuth Token 或學校敏感資料。
- 不得讓任務 ID 因排序或搬動資料列而改變。
- 資料遷移前必須備份。

## 5. 權限與安全

- 所有 Web App 寫入操作必須經過 `assertAuthorized_()`。
- 所有 Web App 寫入操作必須驗證 CSRF Token。
- 所有共享資料寫入必須使用 `LockService`。
- 詳細連結僅允許 `https://`。
- Email 不得作為唯一安全機制；部署權限仍是第一層防護。

## 6. 開發原則

- Sheet 是儲存層，`Code.gs` 是商業邏輯層，HTML 只負責 UI。
- 前端不得直接使用 GViz 或公開 CSV 讀取真實校務資料。
- 看板維持低動畫、黑白高對比、直向可讀性。
- 狀態與優先級必須來自共用選項；類型必須來自目前 profile 或既有資料。
- 日期對外一律使用 `yyyy-MM-dd`，時間使用 `HH:mm`，時區使用 `Asia/Taipei`。

## 7. 修改回報格式

每次修改應說明修改目的、影響檔案、資料契約、權限／部署、測試結果，以及是否需要重新執行安裝或重新部署。

## 8. 最低驗收

- `installSystem()` 能開啟安裝精靈。
- 可選處室與職務並完成安裝。
- 舊資料遷移前有備份，切換 profile 不遺失任務。
- 管理台可新增、編輯、完成及封存任務。
- `工作紀錄` 留下寫入紀錄。
- 看板只讀。
- 多人寫入有鎖。
- 無效處室、職務、狀態、Email、排序與非 HTTPS 連結會被拒絕。

---
> Source: [mihozip/school-admin-daily-dashboard](https://github.com/mihozip/school-admin-daily-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
