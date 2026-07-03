## mailquill

> 本地 Gmail 財務歸檔/報表工具。純本地、純 rule-based、零 LLM。

# mailquill — 開發須知

本地 Gmail 財務歸檔/報表工具。純本地、純 rule-based、零 LLM。
架構與用法見 `README.md`、設計見 `docs/superpowers/`。

## 環境
- 用專案 venv 執行：`.venv/bin/python -m pytest`、`.venv/bin/python -m mailquill.cli <cmd>`
- 不要用系統 `python3`（相依套件只裝在 `.venv`）。

## ⚠️ Token 節省：不要把全部歷史資料撈出來掃
偵錯、調分類或驗證時，**禁止把整份資料或所有帳單一次全部讀出/印出**——這會快速燒掉 token。請改用：
- **SQLite 聚合查詢**：用 `GROUP BY` / `LIMIT` / `SUM`，不要 `SELECT *` 整表，也不要把整個 `transactions.csv`（數百~數千筆）讀進來印。
- **取樣**：只看幾筆代表性資料（`LIMIT 10` 或單一商家），不要列出全部。
- **PDF 只解析需要的那一份**：驗證 parser 時針對「單一檔案」，不要每次都把 6+ 個月所有 PDF 重新解密+抽文字。
- 需要彙總數字時，輸出**統計結果**（總額、各類別金額、未分類筆數），而非逐筆明細。
- 改 `categories.yaml` 後用 `mailquill rebuild`（讀現有 CSV 重新分類），**不要重抓 Gmail / 重解析 PDF**。

## 慣例
- 機密與資料檔不進 git：`config.yaml`/`rules.yaml`/`passwords.txt`/`credentials.json`/`token.json`/`*.pdf`/`transactions.csv`/`*.db`/`report.html`（已在 `.gitignore`）。
- 改動先開分支、TDD、跑全測試後再 merge。
- 新增銀行 parser：`mailquill/parsers/<bank>.py` + 註冊 + 去識別化樣本測試（見既有 parser docstring）。

---
> Source: [kylinfish/mailquill](https://github.com/kylinfish/mailquill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
