## learn-gas

> - 新增一般工具程式時，優先使用 TypeScript 或 Python。

# 協作規則

- 請使用繁體中文回覆與撰寫文件。
- 新增一般工具程式時，優先使用 TypeScript 或 Python。
- Apps Script 執行程式預設使用依功能分檔的 `.gs`；只有完整建立 TypeScript 與 bundler 建置流程時才使用 `.ts`，不得直接將原始 `.ts` 交給 clasp 推送。
- 程式碼必須包含必要的繁體中文註解。
- 不得提交 `.clasprc.json`、OAuth 憑證、API 金鑰、存取權杖或私人 Script ID。
- 修改 Apps Script OAuth scopes 時，採用完成需求所需的最小權限。
- 執行 `clasp push`、部署、建立 GitHub 遠端或公開發布前，必須先取得使用者確認。
- 對使用者第一次提到 `clasp push` 時，先說明它是把本機已完成並通過測試的程式與資訊清單同步到指定 Apps Script 專案，不等於執行程式、寄信、建立觸發器或部署；固定稱為「推送到Apps Script(clasp push)」，不得只丟出英文命令。
- 面向一般使用者或初學者的技術術語，先用白話解釋用途，再固定寫成「中文名稱(English)」，不得只使用英文術語或縮寫；例如「紀錄檔(Log)」。程式碼、命令、檔名、屬性名稱、產品名稱及正式介面文字維持原文，但必須另用中文說明用途。

## 技能包安裝規則

- 使用者要求安裝、更新、檢查或移除本技能包時，先完整讀取根目錄的 `INSTALL.md`，並以該文件作為唯一安裝規格。
- 依目前實際執行的 Agent 用戶端選擇安裝路徑，不得用目前選用的模型名稱判斷。若無法確認用戶端，先詢問使用者，不得猜測。
- 除非使用者明確要求專案限定安裝，否則本技能包預設採使用者層級的全域安裝，讓技能可在 Learn-GAS 儲存庫外的學員專案使用。
- 一次只安裝到目前用戶端對應的一個技能目錄，不得為了保險同時寫入 Claude Code、Codex 與 Antigravity 的所有路徑。
- `skills/` 是技能包原始內容，不是任何 Agent 已完成安裝的證據。安裝時必須保留四個技能目錄與 `learner-facing-terminology.md` 的相對位置。
- 寫入前先核對來源、完整目的地、安裝範圍與同名檔案。遇到既有同名技能或內容差異時停止，說明影響並取得使用者確認，不得直接覆寫或刪除。
- 只有使用者明確要求安裝或更新時才可寫入 Agent 的技能目錄；檢查、說明或閱讀專案不代表取得安裝授權。
- 檔案複製完成不等於技能已可使用。必須依 `INSTALL.md` 在對應用戶端重新載入或開啟新工作階段，並從實際技能清單確認四個技能都可被發現，才能回報安裝完成。

---
> Source: [iamraven-tw/Learn-GAS](https://github.com/iamraven-tw/Learn-GAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
