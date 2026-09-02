## fortivpn-client-cli

> 本文件適用於整個 repository。`CLAUDE.md` 應維持為指向本文件的符號連結，避免不同 coding agents 使用不同規範。

# AGENTS.md

本文件適用於整個 repository。`CLAUDE.md` 應維持為指向本文件的符號連結，避免不同 coding agents 使用不同規範。

## 專案概覽

- 這是 macOS 專用的 Swift CLI，執行檔名稱為 `fortivpn`。
- CLI 讀取 FortiClient 既有的 SSL/IPsec profiles，並透過目前使用者的 FortiTray Unix socket 操作連線。
- `skills/fortivpn` 是跨平台 Agent Skill：macOS 使用本專案 CLI，Windows 使用 Fortinet 官方 `FortiVPN.exe --cli`。
- 本工具不建立、修改或解密 FortiClient profiles。
- 使用者文件以繁體中文（台灣）撰寫；程式碼識別字與 commit message 使用英文。

## 開發與驗證

修改 Swift 程式碼後至少執行：

```sh
swift format --in-place --recursive Sources Tests Package.swift
swift format lint --recursive Sources Tests Package.swift
swift test
swift build -c release
```

修改 Homebrew formula 時另外執行：

```sh
brew style Formula/forticlient-cli.rb
brew audit --strict a3660980/fortivpn-client-cli/forticlient-cli
```

## 安全規則

- 不得提交真實 VPN profile 名稱、公司 hostname、內網 IP、帳號、密碼、PSK、token、憑證或 `AuthInfo`。
- 測試資料使用 `example.com`、RFC 5737 測試 IP 與虛構帳號。
- macOS CLI 的密碼只能透過 `--password-stdin` 用於當次操作，不得寫入 log、設定檔、測試 fixture 或命令列參數。
- Windows Skill 不得將密碼轉交給官方 `--password` 參數，也不得自動使用 `--savecredentials`；應改由 FortiClient 已保存的認證或官方互動流程處理。
- 保留 FortiTray socket 的類型、擁有者與權限檢查。
- 單元測試不得連線或中斷使用者的 VPN。只有使用者明確要求實機整合測試時，才能執行 `fortivpn up` 或 `fortivpn down`，並須恢復原本狀態。
- 不得為多 profile 環境猜測 profile；非互動操作必須使用 `--profile` 或既有預設設定。

## 相容性規則

- 保持 `-H`、`-help`、`--help`、`-h` 相容。
- 保持 JSON 欄位與離開碼向後相容；若必須變更，先更新測試、README 與 CHANGELOG。
- SSL 與 IPsec 必須共用 profile 選取流程，但不得宣稱尚未實機驗證的驗證模式已通過。
- 不得用 log 推測連線狀態；狀態應來自 FortiTray IPC。
- 自動更新通知只能出現在互動模式；`--json`、非 TTY、CI 與明確停用更新檢查時不得顯示 UI。
- 更新只能透過 Homebrew 執行，不得下載未驗證的可執行檔直接覆蓋目前 binary。

## 替使用者操作 VPN

- `fortivpn status` 在未連線時會輸出有效狀態但回傳離開碼 `1`；應解析輸出中的 `state`，不要把它誤判為安裝或執行失敗。
- 只有使用者明確說「開 VPN」、「連 VPN」，或目前任務明確需要公司／私有網路且已用 `fortivpn status --json` 確認 VPN 中斷時，才執行連線；已連線時不要重連。
- 連線前用 `fortivpn profiles --json` 確認可用 profiles。只有一個時可以自動使用；多個且沒有預設值時必須請使用者選擇。
- 若另一個 profile 已連線，不得自行中斷或切換，應先說明衝突並詢問使用者。
- 連線使用 `fortivpn up --profile "<name>" --timeout 60 --json`，完成後再執行 `fortivpn status --json` 驗證。
- 使用者明確說「關 VPN」時，先查詢狀態；已中斷時直接回報。若已連線，使用狀態中的實際 profile 執行 `fortivpn down --profile "<active-profile>" --json`，再查詢狀態確認。
- 不得改用 macOS 系統設定、其他 VPN app 或自行修改 FortiClient profile。
- 若 SAML、OTP、FIDO、憑證或瀏覽器互動尚未完成，請使用者完成官方登入流程後再查詢狀態；不得嘗試繞過驗證。
- 不要反覆重試失敗的連線。保留錯誤輸出並向使用者說明下一步。

## Agent Skill

- `skills/fortivpn/SKILL.md` 是供跨平台 coding agent 安裝的公開 Skill；觸發條件需涵蓋明確 VPN 操作，以及公司／私有網路任務疑似因 VPN 中斷而失敗的情境。
- `SKILL.md` 只保留共用規則與平台路由；macOS、Windows 細節分別維護於 `references/macos.md`、`references/windows.md`，Agent 只能載入目前平台的文件。
- 自動診斷可以直接查詢狀態；自動恢復連線只能使用明確指定、既有預設或唯一的 profile。不得在多 profile 環境猜測。
- macOS Skill 的指令與 CLI 參數必須保持同步；修改 `up`、`down`、`status`、`profiles`、`config`、`doctor` 或 JSON 行為時，應同步檢查 `references/macos.md`。
- Windows 只使用固定安裝位置、Authenticode 簽章有效且簽署者為 Fortinet 的官方 `FortiVPN.exe`。不得只依賴 PATH、不得提供本專案 Windows binary，也不得退回 GUI 自動點擊、舊版工具或未公開 IPC。
- Skill 不得包含個人或公司專用的 profile 名稱、hostname、IP、帳號或驗證資料。
- 修改 Skill 後，使用 `skill-creator` 的 `quick_validate.py` 驗證，並以 `npx skills add` 的本機來源模式確認可被通用 Agent Skills CLI 發現與安裝。
- `fortivpn skill install|status|update` 必須固定操作 `a3660980/fortivpn-client-cli` 的 `fortivpn` Skill，不得接受任意 repository、套件或 shell 參數；預設使用全域範圍，`--project` 才改用目前專案。
- Agent Skills runner 的自動選取順序為 `npx`、`npm exec`、`pnpm dlx`、`yarn dlx`、`bun x --bun`；必須檢查最低相容版本並略過 Yarn Classic 與舊版 npx/npm。`FORTIVPN_SKILLS_RUNNER` 只能接受上述 runner 的固定列舉值。
- Bun runner 必須使用 `--bun` 強制 Bun runtime，避免只有 Bun、沒有 Node 的環境因 Node shebang 失敗。
- CLI 找不到相容 runner 時只能提供安裝提示，不得自行安裝 Node.js、Bun 或任何 package manager。修改 runner 參數時必須為每一種 runner 補單元測試。

## Release 流程

1. 更新 `CLI.version` 與 `CHANGELOG.md`。
2. 執行完整測試與 release build。
3. 建立並推送版本 tag。
4. 下載 GitHub tag archive，計算 SHA-256。
5. 更新 `Formula/forticlient-cli.rb` 的 `url` 與 `sha256`。
6. 執行 Homebrew style、audit、install 與 test。
7. 推送 formula 更新並確認 GitHub Actions 通過。

---
> Source: [a3660980/fortivpn-client-cli](https://github.com/a3660980/fortivpn-client-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
