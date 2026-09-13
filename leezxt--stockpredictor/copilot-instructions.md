## stockpredictor

> 此檔案在 Codex 啟動時自動載入，作為本專案的常駐指引。

# AGENTS.md — StockPredictor

此檔案在 Codex 啟動時自動載入，作為本專案的常駐指引。

## 語言設定

- **一律使用繁體中文（zh-TW）回應**，包含說明、規劃、錯誤分析、commit 訊息草稿、PR 描述。
- 維持完整的中文字符與標點正確性，不要把中文字轉成拼音或同音字。
- 以下內容**保持原文**，不要翻譯：
  - 程式碼識別字（變數、方法、類別、套件名稱）
  - 檔案路徑、URL、套件版本
  - 既有的英文程式碼註解（除非使用者要求翻譯）
  - 設定檔的 key（例如 `spring.datasource.url`）
  - Git 指令、Maven 指令、Shell 指令本身
- 新寫的程式碼註解：若該檔案既有註解風格是中文則用中文，是英文則用英文，**不要混用**。
- 技術術語沿用業界常見譯名；無公認譯名時保留英文（例如 `Repository`、`DTO`、`HikariCP`）。

## 專案概況（給 Codex 的快速 context）

- **類型**：Spring Boot 3.x + H2 嵌入式資料庫的台股分析工具
- **主套件**：`org.gtalent`
- **進入點**：`src/main/java/org/gtalent/Main.java`
- **資料來源**：FinMind API（台股、法人、融資券、籌碼、財報）、TWSE
- **核心模組**：
  - `ScannerService` / `RadarService`：選股與雷達評分
  - `ScoreEngine` / `StrategyScorer`：多面向評分（趨勢、動能、籌碼、波動、市場廣度、基本面）
  - `BacktestEngine` / `KellyCalculator`：回測與凱利公式倉位
  - `DatabaseSchemaInitializer` / Repository：Flyway schema 管理與資料存取（**不**走 Hibernate ddl-auto）
- **設定檔**：`src/main/resources/application.properties`（評分權重可熱調，DevTools 會自動 restart）

## 工作守則

- 編輯 `application.properties` 的權重時，先確認對應的程式碼讀取點（通常在 `AppConfig` 或對應 service 的 `@Value`）。
- Spring Boot 管理單一 `DataSource`；各 Repository 共用此連線池，請勿建立第二組 DataSource。
- 修改評分邏輯前，先看 `StrategyScorer` / `ScoreEngine` 既有規則，避免重複實作。
- 動到 FinMind 相關呼叫時，注意 API 速率限制與 token 認證流程（`FinMindClient`）。

## 工作交接流程

- 每次開始工作時，**必須先讀取根目錄的 `WORK_HANDOFF.md`**，確認上次工作內容、未完成事項、已知問題與建議下一步，再開始檢查或修改程式碼。
- 每次結束工作前，**必須更新 `WORK_HANDOFF.md`**，記錄本次工作大綱。
- 工作大綱至少包含：
  - 本次目標
  - 已完成項目
  - 修改檔案
  - 驗證結果
  - 未完成事項或已知問題
  - 下一次建議起點
- 若本次沒有修改程式碼，也要記錄調查結果、決策或中止原因。
- 更新時保留仍有參考價值的未完成事項，已失效的內容應明確標示已解決或移除，避免下次依照過期資訊工作。

---
> Source: [leezxt/StockPredictor](https://github.com/leezxt/StockPredictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
