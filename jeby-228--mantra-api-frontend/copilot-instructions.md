## copilot-guidelines

> 專案 AI 協作與程式碼風格規範（參考 Copilot 指引）


# AI 協作規範（Mantra API Frontend）

## 核心原則
- 保持簡單，避免過度設計與過早抽象化。
- 一次只解決一個明確問題，小步可驗證地修改。
- 優先可讀性、可維護性與可移植性，而非過度追求微幅效能。

## 程式碼風格
- 程式碼應盡量自我說明，避免贅述型註解。
- 單一職責：函式與模組應聚焦單一目的。
- 優先重用既有元件與樣式系統（Skeleton UI / Tailwind），非必要不新增自訂 CSS。
- 優先文字化與結構化資料，避免隱式規則與魔法值。

## 命名與結構
- 依語言慣例命名：TypeScript/Svelte 使用 `camelCase`；型別/類別使用 `PascalCase`。
- 函式名稱以動詞開頭並描述行為；變數名稱需可表意。
- 元件與功能切分遵循 Atomic Design 精神（由小到大組合）。

## 錯誤處理與日誌
- 提供明確、可行動的錯誤訊息。
- 使用適當錯誤型別或錯誤結構，避免吞錯。
- 僅記錄關鍵錯誤與必要資訊，避免噪音日誌。

## API 與依賴
- 以 GraphQL 作為主要資料傳輸模式，並與後端契約一致。
- 優先使用成熟、廣泛採用且維護中的套件。
- 依賴需定期更新以取得安全修補與重要功能。

---
> Source: [jeby-228/mantra_API_frontend](https://github.com/jeby-228/mantra_API_frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
