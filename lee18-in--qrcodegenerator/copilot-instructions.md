## qrcodegenerator

> 任何動作前，先讀 `./LLM_MEMORY.md` 取得目前狀態與交接日誌。

# AGENTS.md — 所有 AI agent 的最高指令

## 0. 強制第一步

任何動作前，先讀 `./LLM_MEMORY.md` 取得目前狀態與交接日誌。
完成任何工作段落後，必須更新 `./LLM_MEMORY.md` 的〈交接日誌〉與〈目前狀態〉。

### 防膨脹維護（規劃階段 [plan] 負責）

- 〈交接日誌〉超過 20 筆時，將最舊條目壓縮成摘要移入〈已封存結論〉。
- 〈已封存結論〉過長時，整段搬至 `./ARCHIVE.md`（不在強制閱讀清單，需要才查）。

## 1. 階段分工(依 LLM_MEMORY.md〈目前狀態〉記載的階段執行,不越權)

權限跟著「專案階段」走,與使用哪個 LLM/工具無關。
階段只能由使用者宣告切換,agent 不得自行切換。

- **規劃階段 [plan]**: 只產出/修訂純文字架構與計畫,寫入 LLM_MEMORY.md。
  規劃未標記〈已定案〉前,禁止改動程式碼。
- **建置階段 [build]**: 依 LLM_MEMORY.md〈已定案〉的計畫實作基礎骨架與功能代碼。
  不重寫計畫;計畫有問題時在〈交接日誌〉留言,停止該項。
- **維運階段 [maintain]**: 長時間運行下的 debug、小幅修補、看門狗與監控優化。
  禁止架構級變更;需要動架構時在〈交接日誌〉提案,由使用者決定是否退回規劃階段。

## 2. 交接協定(硬規則)

- 切換 agent 前,必須 commit(允許 WIP commit,格式見 §3)。
- 接手的 agent 從最新 commit + LLM_MEMORY.md〈交接日誌〉起手,
  禁止推測前一個 agent 的未記錄意圖。
- 接手時若工作樹不乾淨(前一個 agent 違規未 commit):
  先以 `[takeover] WIP: 接手時發現未提交變更` commit 保存現場,
  並在〈交接日誌〉記一筆地雷警告,再開工。
- 未經使用者明確指示:禁止 push、禁止 force-push、禁止改動遠端分支。

## 3. Commit 格式

`[階段] 動作簡述` 例:`[build] WIP: ADC 濾波參數化,尚缺單元測試`
階段標籤:`[plan]` `[build]` `[maintain]`,交接特例 `[takeover]`(見 §2)。

## 4. 邊界

- README.md 是人類向文件,禁止把規劃過程、狀態、待辦寫進去。
- 禁止修改本檔(AGENTS.md)除非使用者明確指示。

## 5. 專案技術脈絡(依專案填寫,保持精簡)

- 建置指令: pip install -r requirements.txt (或依照 README 安裝 `segno Pillow qrcode[pil]`)
- 測試指令: 無自動化測試指令
- 程式碼慣例: 無特別指定

---
> Source: [lee18-in/QrcodeGenerator](https://github.com/lee18-in/QrcodeGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
