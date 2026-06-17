## llm-wiki-dashboard

> ﻿# Wiki 維護規範（LLM Wiki Schema）

﻿# Wiki 維護規範（LLM Wiki Schema）

此檔案定義本知識庫的固定架構與工作流程。LLM 在此資料夾工作時，必須遵守以下規範。

## 1. 架構

- `raw/`：原始來源（唯讀）
  - `raw/sources/`：文章、PDF、逐字稿、剪貼內容等
  - `raw/assets/`：來源圖片、圖表、附件
- `wiki/`：由 LLM 維護的知識頁（可新增、可更新）
  - `wiki/index.md`：內容導向索引
  - `wiki/log.md`：按時間追加的操作日誌（append-only）
  - `wiki/總覽.md`：目前主題與進度總覽
  - `wiki/來源/`：每份來源的摘要與重點
  - `wiki/主題/`：議題、概念、比較、洞察
  - `wiki/實體/`：人物、組織、產品、術語等實體頁
  - `wiki/分析/`：問答結果、綜合分析、階段結論
  - `wiki/templates/`：頁面模板

## 2. 語言與格式

- 所有 wiki 筆記一律使用「繁體中文」。
- 檔名可用繁中，內容採 Markdown。
- 內部連結盡量使用 `[[頁名]]`（Obsidian 友善）。
- 每頁建議包含：
  - 標題
  - `最後更新` 日期
  - 核心摘要
  - 關聯頁面
  - 來源引用（若適用）

## 3. Ingest（來源納入）標準流程

當使用者要求「處理新來源」時，LLM 依序執行：

1. 讀取 `raw/sources/` 的指定來源。
2. 建立或更新 `wiki/來源/<來源名稱>.md`。
3. 更新受影響的主題頁、實體頁、分析頁。
4. 補上必要交叉連結（雙向連結優先）。
5. 更新 `wiki/index.md`。
6. 以固定格式追加一筆到 `wiki/log.md`。

### log 格式（固定）

```md
## [YYYY-MM-DD] ingest | <來源名稱>
- 來源檔案：`raw/sources/<檔名>`
- 新增頁面：[[頁A]]、[[頁B]]
- 更新頁面：[[頁C]]、[[頁D]]
- 重點：一句話摘要
```

## 4. Query（查詢）標準流程

當使用者提問時：

1. 先讀 `wiki/index.md` 判斷相關頁。
2. 再讀取相關頁面進行綜合回答。
3. 回答應附可追溯來源（對應 wiki 頁或 raw 來源）。
4. 若內容具長期價值，建立到 `wiki/分析/` 的新頁。
5. 追加 log：

```md
## [YYYY-MM-DD] query | <問題簡述>
- 參考頁面：[[頁A]]、[[頁B]]
- 產出：[[分析/<檔名>]]（如有）
- 結論：一句話摘要
```

## 5. Lint（健康檢查）標準流程

定期檢查並修補：

- 互相矛盾的敘述
- 已過時但未標註的主張
- 缺少入站連結的孤兒頁
- 重要概念已出現但尚未建頁
- 索引與實際檔案不一致

完成後追加 log：

```md
## [YYYY-MM-DD] lint | 例行檢查
- 問題數：N
- 已修正：...
- 待處理：...
```

## 6. 變更原則

- `raw/` 視為來源真相，除非使用者要求，不修改其內容。
- `wiki/` 可重構，但需保留可追溯性。
- 每次大量更新後，必須同步更新 `wiki/index.md` 與 `wiki/log.md`。

---
> Source: [KennyKang7012/llm-wiki-dashboard](https://github.com/KennyKang7012/llm-wiki-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
