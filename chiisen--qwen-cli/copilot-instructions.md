## qwen-cli

> 本文件記錄了 `qwen_cli` 專案開發過程中累積的最佳實踐與原則。AI Agent 應優先遵循此規範。

# GEMINI 專案開發指南 (Project Guidelines)

本文件記錄了 `qwen_cli` 專案開發過程中累積的最佳實踐與原則。AI Agent 應優先遵循此規範。

## 1. 前端開發規範 (Frontend Development)

### 1.1 結構與職責分離 (Separation of Concerns)
*   **原則**：避免將 CSS 樣式與 JavaScript 邏輯直接寫在 HTML 檔案中。
*   **實踐**：
    *   樣式應獨立於 `style.css`。
    *   互動邏輯應獨立於 `script.js`。
    *   HTML (`.html`) 僅負責結構與語意。
*   **例外**：僅在極其簡單的單檔原型 (Single File Prototype) 中允許混寫，但需在複雜度提升時立即重構拆分。

### 1.2 語言在地化 (Localization / i18n)
*   **原則**：使用者介面 (UI) 與註解應優先使用 **繁體中文 (Traditional Chinese, zh-TW)**。
*   **實踐**：
    *   HTML 內文、按鈕標籤、提示訊息 (Toasts/Alerts) 均需中文化。
    *   保留專有名詞（如 `AI`, `CLI`, `GitHub`）以維持專業性。

### 1.3 視覺交互設計 (Visual Interaction Design)
*   **原則**：確保使用者操作有明確且高對比的視覺回饋。
*   **實踐**：
    *   **主題切換**：不同主題間應有顯著的色系差異（如冷色系 vs 暖色系），避免僅做細微的亮度調整。
    *   **狀態指示**：按鈕點擊、載入中等狀態需有明確動畫或樣式變化（如 Glitch 特效、Loading Spinner）。

### 1.4 狀態管理與可維護性 (State & Maintainability)
*   **原則**：避免依賴 CSS Computed Style 來判斷程式邏輯狀態。
*   **實踐**：
    *   使用 HTML `data-*` 屬性（如 `data-theme="red"`）來標記狀態。
    *   JavaScript 僅負責切換 `data-attribute`，樣式變化交由 CSS 選取器（如 `[data-theme="red"] { ... }`）處理。
    *   **全域變數配置**：將可調整的參數（如動畫顏色 `matrixColor`、時間間隔）提取為全域變數或配置物件，便於後續維護與動態調整。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chiisen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chiisen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
