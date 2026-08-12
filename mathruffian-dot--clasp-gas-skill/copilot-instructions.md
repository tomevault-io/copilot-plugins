## clasp-gas-skill

> 這個 repo 提供一個跨 agent 通用的 Agent Skill：**clasp ＋ Google Apps Script 連線**。

# AGENTS.md

這個 repo 提供一個跨 agent 通用的 Agent Skill：**clasp ＋ Google Apps Script 連線**。

## 技能位置

- 技能本體：[`skills/clasp-setup/SKILL.md`](skills/clasp-setup/SKILL.md)
- 平台差異：[`skills/clasp-setup/references/platform-notes.md`](skills/clasp-setup/references/platform-notes.md)

## 什麼時候載入

使用者提到以下任一情境時，讀 `skills/clasp-setup/SKILL.md` 並照著執行：

- 接 Apps Script、clasp 登入、用 clasp 改線上程式碼
- 把試算表變成可以填的網頁表單
- 部署 GAS 網頁應用程式、拿 `/exec` 網址
- Apps Script 相關的錯誤排除（`admin_policy_enforced`、「網頁不存在」、指令不存在）

## 在這個 repo 裡工作時

- `SKILL.md` 是唯一的內容來源，`README.md` 只是人類版說明。改動流程或指令時**兩邊都要同步**。
- 指令語法以 **clasp v3** 為準（`create-script`、`open-script`、`open-web-app`、`create-deployment`、`list-deployments`、`show-authorized-user`）。
  變更前請對照 <https://github.com/google/clasp> 的 README 與原始碼，不要憑記憶——網路上多數教學仍是 v2。
- 不要在任何檔案裡放入真實的 scriptId、deploymentId、Google 帳號或其他個資，範例一律用 `<placeholder>`。

---
> Source: [mathruffian-dot/clasp-gas-skill](https://github.com/mathruffian-dot/clasp-gas-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
