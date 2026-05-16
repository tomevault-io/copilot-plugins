## job-management-system

> > Updated: 2026-04-18

# Job-Management-System AGENTS.md

> Creator: Show  
> Updated: 2026-04-18

## 回覆規則

- 一律使用繁體中文（台灣用語）。
- 以淺顯方式說明，不要堆太多專業術語。
- 除非使用者明確要求，否則不要直接觸發部署。

## 開發規則

- 開始處理專案前，先啟用 Serena 專案並使用 Serena 語義工具找資料。
- 查文件時優先用 Context7。
- 驗證畫面時固定用 Chrome DevTools。
- 功能修改遵循 TDD，先補測試，再改程式，再驗證。

## 專案基本資訊

- 專案類型：React 19 + Vite 前端專案（Firebase Auth + Firestore）。
- GitHub：`https://github.com/Showchen168/Job-Management-System.git`
- 建置指令：`npm run build`（CI 上會改用 `pnpm run build`）
- 輸出目錄：`dist/`
- 本機開發：`npm run dev`（port 5173）
- 本機預覽：`npm run preview`（port 4173）
- Playwright 預覽網址：`http://127.0.0.1:4173`
- `vite.config.js` 的 `base` 設為 `/Job-Management-System/`，GitHub Pages 子路徑對應這裡，**不要亂改**。

## 系統架構

這個系統由兩個獨立部署的元件組成：

| 元件 | 類型 | 部署在哪 | 說明 |
|---|---|---|---|
| **這個 repo**（前端） | Vite 靜態網站 | **GitHub Pages** | 使用者實際看到的 UI |
| `jms-mcp`（附屬 MCP 服務） | Node.js API/MCP | **PC Docker** | 給 AI 工具讀寫 JMS 資料用，不是這個 repo |

→ 這個 repo **只負責 GitHub Pages 的前端**。
→ PC Docker 上的 `jms-mcp` 是**另一個 repo**（`/Users/show/Desktop/Claude code agent/Projects/MCP-Server/jms-mcp`），要改那邊請切過去。

## 部署資訊（GitHub Pages，CI 自動部署）

### 部署機制

- 走 GitHub Actions：`.github/workflows/deploy.yml`
- 觸發條件：**push 到 `main` 分支**（或手動 `workflow_dispatch`）
- 流程：GitHub Runner 裝 pnpm → `pnpm install --frozen-lockfile` → `pnpm run build` → 上傳 `dist/` → Deploy to GitHub Pages
- Node 版本：20（CI 上）
- 套件管理：pnpm 10（CI 上，本機 `npm` 也可以）
- 公開網址：`https://showchen168.github.io/Job-Management-System/`

### 環境變數（存在 GitHub Secrets）

CI 在 build 時從 GitHub repo secrets 注入以下變數：
- `VITE_FIREBASE_API_KEY`
- `VITE_FIREBASE_AUTH_DOMAIN`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_STORAGE_BUCKET`
- `VITE_FIREBASE_MESSAGING_SENDER_ID`
- `VITE_FIREBASE_APP_ID`
- `VITE_FIREBASE_MEASUREMENT_ID`
- `VITE_ROOT_ADMINS`

Firebase 專案 ID：`job-management-system-16741`（僅供 cross-reference，實際值以 secret 為準）。

注意：Vite 在 **build 階段**就把 `VITE_*` 烙進 bundle，所以要改這些值必須**改 GitHub Secrets 並重跑 Action**，不是改 repo 裡的 `.env`。

### 如何部署（正常流程）

1. 本機把修改 commit → push 到 `main`
2. GitHub Actions 自動 build + deploy（約 1~2 分鐘）
3. 到 Actions 頁面看 job 綠燈
4. 打開公開網址驗證

### 如何臨時手動觸發

- GitHub repo → Actions → `Deploy to GitHub Pages` → `Run workflow`

### ❌ 不要做的事

- **不要用 `pc-docker-deploy` 技能部署這個前端** — 它跑在 GitHub Pages，不是 PC Docker
- **不要用 `nas-deploy` 技能** — 同上，不在 NAS
- **不要把 `dist/` 手動 commit 進 repo** — CI 會自己產

## 部署後驗證

push 完之後要做的事：

1. 到 GitHub Actions 看 `Deploy to GitHub Pages` 綠燈
2. 打開 `https://showchen168.github.io/Job-Management-System/`
3. 用 Chrome DevTools：
   - 頁面標題正確
   - Console 沒明顯錯誤
   - Firebase Auth 可以登入
   - Firestore 資料有載入
4. 抓 bundle 檔名驗證新版真的有上：
   ```
   curl -sS https://showchen168.github.io/Job-Management-System/ | grep -oE "/assets/index-[A-Za-z0-9_-]+\.(js|css)"
   ```
5. 驗證本次新加的 function 名或字串有進 bundle：
   ```
   curl -sS https://showchen168.github.io/Job-Management-System/assets/index-XXXX.js | grep -c "<本次新增的 identifier>"
   ```

## 建議驗證流程（push 之前）

1. `npm run build` 本機先跑過
2. `npx vite preview` 打開看主要頁面
3. 用 Chrome DevTools 檢查登入流程、Firestore 讀寫
4. 跑需要的 Playwright 測試
5. commit + push → 交給 CI

## 待補資訊

下面之後確認了請補上：

- 若啟用 custom domain（例如 `jms.showmico888.net`）要在這裡記錄
- 若有 PR Preview 環境（Cloudflare Pages Preview、Netlify 等）要記錄

---
> Source: [Showchen168/Job-Management-System](https://github.com/Showchen168/Job-Management-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
