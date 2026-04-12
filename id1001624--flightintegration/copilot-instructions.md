## flightintegration

> - **核心業務**: 本專案專注於**國際航空路線**，包含在台灣轉機的航班。

## 專案核心規則 (Project Core Rules)

### 1. 專案範圍 (Project Scope)
- **核心業務**: 本專案專注於**國際航空路線**，包含在台灣轉機的航班。
- **排除範圍**: 所有功能開發與數據處理**必須排除**純台灣國內航線及離島航線。

### 2. 資料來源真實性 (Source of Truth)
- **機場地區**: 機場的國家/地區資訊，唯一真實來源是後端資料庫 `airports` 資料表的 `country` 欄位。所有前端顯示和後端邏輯都必須以此為準。
- **前端 `_getAirportRegion`**: `frontend/src/api/services/flightService.js` 中的 `_getAirportRegion` 函式已被正式標記為**技術債**。在後續的優化任務中，應優先將其移除，並改為使用後端提供的 `country` 資料。我將不再基於此函式的邏輯提出任何建議。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/id1001624)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/id1001624)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
