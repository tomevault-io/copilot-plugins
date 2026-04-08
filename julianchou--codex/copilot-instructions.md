## codex

> ﻿# AGENTS.md — 專案協作與工程化指引（v1.0 / 2026-03-28）

﻿# AGENTS.md — 專案協作與工程化指引（v1.0 / 2026-03-28）

本文件為本庫的協作與工程化規範（適用於整個專案目錄）。目標：免費優先（Free-first）的 CI / Security / Review / 文件骨架，並避免動到既有業務邏輯。

## 原則
- 僅新增流程檔、文件檔與 PowerShell 腳本；不修改任何業務程式碼（除非另有明確要求）。
- 文件一律使用繁體中文；腳本使用 PowerShell（7+），CI 使用 GitHub Actions。
- 不要放入真實 Secrets；如需使用，請以 `${{ secrets.MY_SECRET }}` 之類 placeholder 表示，並於 Repo 設定中配置。
- 變更應最小、可回滾、可追溯；避免引入與需求無關的檔案。
- 若同目錄或子目錄存在更特定的 `AGENTS.md`，以「更近層級」的規範優先。

## 重要檔案
- `.github/workflows/ci.yml`：通用 CI（Ubuntu Runner + PowerShell），呼叫 `scripts/lint.ps1` / `scripts/test.ps1`。
- `.github/workflows/security.yml`：免費安全檢查（Semgrep、Gitleaks、Trivy FS），上傳 SARIF。
- `.github/dependabot.yml`：每週依賴更新（含 github-actions 與常見生態系）。
- `.github/pull_request_template.md`：PR 模板與自我檢查清單。
- `code_review.md`：程式碼審查清單與建議。
- `docs/engineering-workflow.md`：工程作業流程（分支、PR、CI、安全、依賴更新）。
- `scripts/lint.ps1`：跨語言的 Lint 腳本（盡量使用免費工具；若工具不存在則跳過或以輕量規則檢查）。
- `scripts/test.ps1`：跨語言的 Test 腳本（偵測可用測試框架；無測試則跳過並成功結束）。

## 本地常用指令
- 執行 Lint：`pwsh -File scripts/lint.ps1`
- 執行 Test：`pwsh -File scripts/test.ps1`

## 分支與提交
- 建議分支命名：`feature/*`、`fix/*`、`chore/*`、`docs/*`。
- 提交訊息建議遵循 Conventional Commits（例：`feat: 新增設定檔`、`fix: 修正 CI 條件`）。

## 禁止事項
- 提交任何敏感資訊（金鑰、密碼、私鑰、存取權杖等）。
- 在版本控制中新增編譯產物或大型二進位檔（除非經明確同意）。

---
文件維護者：工程化維運。更新日期：2026-03-28。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianchou)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julianchou)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
