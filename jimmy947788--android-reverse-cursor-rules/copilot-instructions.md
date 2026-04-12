## android-reverse-cursor-rules

> Frida 腳本與動態觀測規則（JavaScript/TypeScript）


# Frida 腳本規範

## 基本原則
- **預設以「觀測」為主**：記錄呼叫堆疊、參數、返回值、threadId、caller module
- **簡潔性優先**：寫簡單直接的代碼，避免不必要的複雜度
- **早期返回**：使用早期返回避免嵌套條件，提高可讀性
- **函數式風格**：優先使用函數式、不可變風格，除非變得過於冗長

## 配置與旗標
- **可關可開旗標**：盡量採用「可關可開」旗標
  ```javascript
  const ENABLE_TRACE = true;
  const ENABLE_DUMP = false;
  const ENABLE_VERBOSE = false;
  ```

## Hook 策略
- **Java 層**：優先 hook 高層 API（PackageManager、ClassLoader、Cipher、MessageDigest）
- **Native 層**：優先 hook exports / 符號清楚的 API；必要時才用 offset
- **防呆機制**：任何 hook 請自帶防呆
  - try/catch 包裹
  - 判斷 null、pointer validity
  - 避免在 hot path 大量 console.log（改用 ring buffer 或採樣）

## 註釋與文檔
- **函數註釋**：在每個函數開始處添加註釋，描述其功能
- **JSDoc 註釋**：對 JavaScript 使用 JSDoc 註釋和現代 ES6 語法
- **TODO 註釋**：如果遇到現有代碼中的 bug，添加以 "TODO:" 開頭的註釋，概述問題

# 記錄格式（統一）
- **每條 log 含**：時間、tid、module!symbol/offset、args、ret
- **堆疊追蹤**：使用 Backtracer.ACCURATE（能用就用），不行再 FALLBACK
- **性能考量**：避免在 hot path 大量 console.log（改用 ring buffer 或採樣）

# 代碼質量要求
- **描述性命名**：使用描述性的變數和函數名
- **模組化設計**：鼓勵模組化設計以提高可維護性和可重用性
- **錯誤處理**：實現穩健的錯誤處理和日誌記錄
- **詳細規則**：請參考 `05_code_quality.mdc`

# Frida 執行環境配置（強制）
- **詳細配置**：請參考 `25_environment_config.mdc`

# 禁止事項
- 不輸出可直接用於未授權繞過金融/身分驗證的完整攻擊腳本。
- 若使用者要求 bypass：改提供「檢測點定位、風險面、可觀測性方案、最小化實驗」。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimmy947788)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jimmy947788)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
