## harness-starter-kit

> 先讀同一層的 `CLAUDE.md`，那份是主要的安裝指示，裡面關於「怎麼跟不會寫程式的人講話」「裝之前先問哪三件事」「裝完要提醒重開」的部分，在這裡一樣適用。

# 給 Codex 的說明

先讀同一層的 `CLAUDE.md`，那份是主要的安裝指示，裡面關於「怎麼跟不會寫程式的人講話」「裝之前先問哪三件事」「裝完要提醒重開」的部分，在這裡一樣適用。

這份檔案只補充 Codex 跟 Claude Code 不一樣的地方。

## hooks 不能照搬

`hooks/` 底下那三支是照 Claude Code 的機制寫的，直接複製到 Codex 環境不會動。差別在於：

- 事件的名稱和涵蓋範圍不一樣，Codex 這邊能攔到的動作比較少。
- 檔案寫入的動作在兩邊叫法不同，判斷條件要重寫。
- Claude Code 用來傳資料給腳本的格式，Codex 不見得一樣，讀取的那幾行要跟著改。

所以遇到使用者拿這包給 Codex 裝的時候，**先講清楚這件事，不要照抄然後宣稱裝好了**。

可行的做法有兩條，讓使用者選：

1. 只裝 `skills/`，那部分是純文字的做事步驤，兩邊通用。
2. 你依照 Codex 現行的機制，把那三支的判斷邏輯重寫一遍。原理很簡單（記帳、對帳、比對文字），重寫不難，但要實際測過再說裝好了。

## skills 可以直接用

`skills/` 底下的五個資料夾都是純文字說明，沒有綁定任何平台。放進 Codex 讀得到的位置即可。

checkpoint 跟 neat-freak 裡面提到的「子代理」在 Codex 有對應的做法，設定的名稱不一樣，照你這邊的慣例調整。

## 不要假裝裝好了

這整包東西的主題就是「不要相信沒有證據的完成宣稱」。你在幫別人裝的時候更不能犯這個錯。

裝不了的部分講清楚裝不了，重寫過的部分講清楚你改了什麼，測過才說能動。

---
> Source: [agentcrew-academy/harness-starter-kit](https://github.com/agentcrew-academy/harness-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
