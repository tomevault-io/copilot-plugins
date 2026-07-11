## calculator04

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

這是一個使用純 JavaScript (Vanilla JS) 開發的工程用計算機 Web 應用程式，支援基本四則運算與工程函數（平方、開根號、三角函數）。所有程式碼、註解和文件皆使用正體中文。

**核心技術棧**:
- HTML5, CSS3, JavaScript (ES6+)
- 無外部依賴（純 Vanilla JavaScript）
- 測試框架：Jest 29.7.0

## 重要開發命令

### 測試相關
```bash
# 執行所有測試
npm test

# 執行單一測試檔案
npm test -- tests/unit/basicOperations.test.js

# 監視模式（開發時使用）
npm run test:watch

# 產生測試覆蓋率報告
npm run test:coverage
```

### 執行應用程式
直接在瀏覽器中開啟 `index.html`，無需建置或安裝。

### 除錯
- 在瀏覽器中開啟開發者工具 (F12)
- 使用 Console 查看錯誤訊息
- 所有 JavaScript 程式碼位於 `index.html` 的 `<script>` 區塊中

## 架構設計

### 單一檔案架構
所有應用程式邏輯（HTML、CSS、JavaScript）都集中在 `index.html` 中。這是刻意的設計選擇，遵循「拒絕過度設計」原則。

### 核心函數組織
在 `index.html` 的 `<script>` 區塊中，函數按以下順序組織：

1. **計算機狀態** (`calculatorState` 物件)
   - `displayValue`: 顯示螢幕的值
   - `firstOperand`: 第一個運算元
   - `operator`: 當前運算子
   - `waitingForSecondOperand`: 是否等待第二個運算元
   - `isError`: 錯誤狀態標記

2. **工具函數**
   - `resetCalculator()`: 重置計算機狀態
   - `updateDisplay(value)`: 更新顯示螢幕
   - `displayError(message)`: 顯示錯誤訊息
   - `validateInput(input)`: 驗證輸入
   - `formatResult(value)`: 格式化結果為小數點後 6 位

3. **核心計算函數**
   - `add(a, b)`, `subtract(a, b)`, `multiply(a, b)`, `divide(a, b)`: 基本四則運算
   - `square(x)`, `sqrt(x)`: 平方與開根號
   - `sin(degrees)`, `cos(degrees)`: 三角函數（度數制）
   - `degreesToRadians(degrees)`: 度數轉弧度

4. **事件處理函數**
   - `handleNumberClick(digit)`: 數字按鈕點擊
   - `handleOperatorClick(operator)`: 運算子按鈕點擊
   - `handleEqualsClick()`: 等號按鈕點擊
   - `handleClearClick()`: 清除按鈕點擊
   - `handleUnaryOperatorClick(operator)`: 單運算元運算子點擊

### 測試架構
測試檔案位於 `tests/` 目錄，採用 TDD 方法開發：

- **單元測試** (`tests/unit/`):
  - `basicOperations.test.js`: 基本四則運算測試（27 個測試）
  - `powerOperations.test.js`: 平方與開根號測試（14 個測試）
  - `trigOperations.test.js`: 三角函數測試（28 個測試）

- **整合測試** (`tests/integration/`):
  - `calculator.integration.test.js`: 端對端情境測試（33 個測試）

**重要**: 測試檔案中的函數是從 `index.html` 複製過來的，因為在測試環境中無法直接載入 HTML 檔案中的函數。修改函數時，必須同時更新 `index.html` 和對應的測試檔案。

**雙向同步檢查**: 修改後務必執行 `npm test` 確保所有測試通過，這是驗證同步正確性的最佳方式。

## 專案憲法（NON-NEGOTIABLE）

本專案遵循 `.specify/memory/constitution.md` 定義的五大核心原則，這些原則優先於所有其他開發慣例：

### 1. 可測試性（NON-NEGOTIABLE）
所有功能必須可被獨立測試，具有明確的輸入與預期輸出。

### 2. 測試先行（NON-NEGOTIABLE）
嚴格遵循 TDD 流程：
1. 先撰寫測試
2. 測試失敗（紅燈）
3. 實作功能讓測試通過（綠燈）
4. 重構

**絕不允許先寫實作再補測試。所有測試必須經使用者確認後才開始實作。**

### 3. MVP 優先
優先實作 P1 (Priority 1) 使用者故事，每個故事完成後驗證、部署，再進行下一個。

### 4. 拒絕過度設計
遵循 YAGNI 原則，不為「未來可能需要」的功能預先設計。保持簡單，優先使用最簡單的解決方案。

### 5. 正體中文優先（NON-NEGOTIABLE）
所有文件、註解、使用者介面文字、提交訊息皆使用正體中文。變數與函數命名允許使用英文（程式碼相容性），但必須有正體中文註解說明。

## SpecKit 工作流程

本專案使用 SpecKit 進行需求管理與實作規劃，所有規格文件位於 `specs/001-engineering-calculator/` 目錄：

- `/speckit.specify`: 建立功能規格（`spec.md`）
- `/speckit.plan`: 進行技術規劃（`plan.md`）
- `/speckit.tasks`: 產生任務清單（`tasks.md`）
- `/speckit.implement`: 執行實作
- `/speckit.analyze`: 檢查專案合規性

## 錯誤處理模式

應用程式使用一致的錯誤處理模式：
- 計算函數（`divide`, `sqrt`）在遇到無效操作時拋出 `Error`
- 錯誤訊息使用正體中文（例：「無法除以零」、「無法對負數開根號」）
- UI 層面透過 `displayError(message)` 捕獲錯誤並顯示給使用者
- 錯誤樣式會在 2 秒後自動移除

## 注意事項

### 修改核心函數時
如果修改 `index.html` 中的任何核心計算函數或工具函數，**必須同步更新對應的測試檔案**：
- 修改 `add, subtract, multiply, divide, validateInput` → 更新 `tests/unit/basicOperations.test.js`
- 修改 `square, sqrt` → 更新 `tests/unit/powerOperations.test.js`
- 修改 `sin, cos, degreesToRadians` → 更新 `tests/unit/trigOperations.test.js`

### 三角函數角度制
本專案三角函數使用**度數制**（而非弧度制），這是工程計算機的標準慣例。所有 sin/cos 輸入皆為度數，內部透過 `degreesToRadians()` 轉換。

### 精確度要求
計算結果精確度至小數點後 6 位，透過 `formatResult()` 函數處理，並移除尾隨的零。

### 使用者故事優先級
功能按以下優先級實作：
- **P1**: 基本四則運算（MVP，已完成）
- **P2**: 平方與開根號（已完成）
- **P3**: 三角函數 sin、cos（已完成）

## 版本資訊
- **版本**: 1.0.0
- **測試通過率**: 100% (102/102 測試通過)
- **憲法版本**: 1.0.0

---
> Source: [ROGabdp/calculator04](https://github.com/ROGabdp/calculator04) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
