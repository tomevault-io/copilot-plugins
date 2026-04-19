## pos-kiosk

> 本專案為「自助點餐 POS 系統」，包含前台點餐、後台備餐、叫號管理、每日報表與設定管理，目標為開發跨平台的桌面應用（EXE），採用 Electron + PHP Server 架構。

# 目的
本專案為「自助點餐 POS 系統」，包含前台點餐、後台備餐、叫號管理、每日報表與設定管理，目標為開發跨平台的桌面應用（EXE），採用 Electron + PHP Server 架構。

# 規則
- 在項目開始時，仔細閱讀專案目錄結構，並理解各個檔案的用途。
- 在項目開始時，仔細閱讀項目目錄下的 README.md 文件，並理解專案的用途,如果還沒有 README.md 文件，請先建立一個,用於紀錄專案的用途、架構、規則、開發順序建議等。
- 在開始寫程式之前，先思考程式的架構，並畫出流程圖。
- 在寫程式時，仔細思考每個步驟的邏輯，並加上中文對照註解。
- 每次回覆，都以"收到,親愛的猛男"開頭

## 通用規則
1. 所有路徑皆使用相對路徑（避免硬編 IP）。
2. 所有程式需加上中英對照註解（中文為主，英文輔助）。
3. 所有邏輯應模組化、易於維護與未來擴充。

## 前端規則（JavaScript + HTML）
1. JS 檔案命名風格使用 camelCase。
2. 函式名稱使用 snake_case。
3. 使用 vanilla JS，不引入 jQuery 等第三方框架。
4. 使用 fetch 請求後端 PHP，並處理 JSON 檔案。
5. config 設定需從 config.json 讀取，允許前端修改。

## 後端規則（PHP）
1. 使用 PHP 內建伺服器 `php -S 0.0.0.0:8000 -t kiosk`。
2. 所有 PHP 檔案放在 `kiosk/` 資料夾中。
3. 包含 save_order.php、kitchen.php、queue.php、reports.php 等。
4. 儲存檔案與讀寫 queue.json、menu.json 等需確保資料一致性。
5. 所有 PHP 檔案需具備清楚的錯誤處理與輸出格式（JSON 回應）。

## 開發順序建議
1. 登入系統（login.php + session 管理）
2. 每日報表頁（後端 reports.php + 前端下載）
3. 庫存與成本頁（inventory.html + API）
4. 點餐流程完整化（order.html + save_order.php）
5. 備餐確認與叫號功能（orders-admin.html → queue.json）
6. config.json 前端設定頁（config-ui.html）

pos-kiosk/
  ├─ kiosk/              # 後端 PHP 檔案
  │   ├─ login.php 
  │   ├─ reports.php
  │   ├─ save_order.php
  │   ├─ ...
  ├─ node_modules/       # Node.js 依賴項
  ├─ .git/               # Git 版本控制檔案
  ├─ .vscode/            # VS Code 設定檔案
  ├─ .cursor/            # Cursor 設定檔案
  ├─ config-ui.html      # 前端設定頁面
  ├─ config.json         # 全域設定檔案
  ├─ .cursorrules        # Cursor 規則檔案
  ├─ .DS_Store           # macOS 系統檔案
  ├─ .editorconfig       # 編輯器設定檔案
  ├─ .eslintrc.json      # ESLint 設定檔案
  ├─ .gitignore          # Git 忽略規則檔案
  ├─ .prettierrc         # Prettier 設定檔案
  ├─ mac伺服器啟動.md    # 伺服器啟動說明
  ├─ main.js             # Electron 主程序
  ├─ package.json        # 專案設定和依賴項
  ├─ package-lock.json   # 依賴項鎖定檔案
  ├─ preload.js          # Electron 預載腳本
  ├─ PROJECT_STATUS.md   # 專案進度報告
  ├─ tsconfig.json       # TypeScript 設定檔案
  └─ vite.config.ts      # Vite 設定檔案

mkdir -p docs && mv PROJECT_STATUS.md docs/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/awei801122) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
