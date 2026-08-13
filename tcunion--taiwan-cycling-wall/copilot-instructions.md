## taiwan-cycling-wall

> 新對話開始時讀取 `/Volumes/OWC 2T/Codex/MyObsidian/brain/handoffs/約騎系統.md`（先看「立即上手」即可上工）。

# 約騎公布欄 — AGENTS.md

## 會話規則

新對話開始時讀取 `/Volumes/OWC 2T/Codex/MyObsidian/brain/handoffs/約騎系統.md`（先看「立即上手」即可上工）。

收工時更新同一份 handoff，使用 Read/Write 工具直接操作，不走 obsidian MCP。

---

## 專案概述

台灣單車約騎社群平台 PWA（siokiu），以軟木佈告欄 + 便利貼風格呈現約騎活動，支援台灣 22 縣市區域篩選。全繁體中文介面。

正式網址：`https://siokiu.criterium.tw/`

## 技術棧

- **Vite 8** + **React 19** + **TypeScript**
- **Tailwind CSS v4**（設定在 `src/index.css` `@theme` 區塊，**非** `tailwind.config.js`）
- **Zustand** + localStorage persist / **React Router v7**
- **Lucide React**（圖示 — **禁止使用 emoji 作為 UI icon**）
- **Leaflet** + OSRM（路線規劃，無需 API Key）
- **Supabase** Cloud（`jxubndwcralkrbunxokf.supabase.co`，Seoul 區）
- **FB SDK v21** / **Google GIS** / **LINE Login + LIFF** / **Strava → n8n webhook**
- **date-fns** / **vite-plugin-pwa**

## 常用指令

```bash
npm run dev      # 開發伺服器
npm run build    # TypeScript 檢查 + 生產建置
npm run preview  # 預覽生產建置
npm run lint     # ESLint 檢查
```

## 程式碼慣例

- **語言**：註解、變數名稱盡量使用繁體中文；React 元件名用英文（JSX 限制）
- **回應語言**：一律使用繁體中文，包含 commit 訊息、程式碼註解；React 元件名與 JSX 仍用英文
- **Zustand store**：state/action 使用繁體中文命名（如 `使用者`、`登入`、`活動列表`）
- **路由保護**：需登入頁面透過 `RequireAuth` 元件包裹（`App.tsx`）
- **頁面延遲載入**：所有頁面使用 `React.lazy()` + `Suspense`
- **便利貼旋轉**：`hashCode(id) % 5 - 2` 度，class 為 `sticky-rotate-{n2|n1|0|1|2}`
- **無障礙**：icon-only 按鈕必須加 `aria-label`；所有可點擊元素加 `cursor-pointer`
- **動畫**：全域支援 `prefers-reduced-motion: reduce`；transition 使用 150–300ms
- **安全性**：備註欄位禁止輸入 http/https 連結，提交時自動過濾
- **Supabase 查詢**：`select()` 必須列出明確欄位，禁止 `select('*')`
- **Hooks 規則**：`useMemo` / `useCallback` 必須在所有 early return **之前**宣告

## 非直覺技術決策（重要）

### Supabase 建表必須 GRANT 權限
自架 Supabase 建新表後，RLS 預設不足，**務必**執行：
```sql
GRANT ALL ON <table_name> TO anon, authenticated;
```
否則前端會收到 403 或空資料。

### 活動過期邏輯
活動過期 = **約騎日期 + 時間 + 12 小時**（非隔天凌晨）。
`eventStore.ts` 的 `已過期()` 函式解析 `date`（YYYY-MM-DD）與 `time`（HH:MM）後加 12 小時比對現在時間。

### OAuth state 驗證（LINE / Strava）
`預期State` 為 null（sessionStorage 過期）時**直接拒絕**，回傳「登入資料已過期」錯誤。
禁止退回 prefix-only fallback（`state.startsWith('line-')`），以防 CSRF。

### LINE 分享不用 URL scheme
LINE URL scheme（`line://msg/text/...`）因內容過長導致 414 Request-URI Too Large。
現行做法：優先 `navigator.share({ text })`，不支援時複製到剪貼簿並顯示 4 秒 toast。

### Facebook 分享用 sharer.php
`dialog/share` API 需用戶開啟平台功能，相容性差。改用 `facebook.com/sharer/sharer.php?u=`。

### Google JWT UTF-8 解碼
GIS 回傳的 id_token 中文欄位需用 `decodeURIComponent` + 逐位元組 `%XX` 方式處理，否則中文名稱亂碼。

### 模擬活動無法參加
`id` 以 `evt-` 開頭的活動為模擬種子資料，前端一律禁止加入。

### 圖章上傳限制（storageService.ts）
僅接受 `image/png` / `image/jpeg` / `image/webp`，且解碼後超過 2MB 一律回傳原始 dataUrl（不上傳）。

## 登入機制

| Provider | User ID 格式 | 備註 |
|----------|-------------|------|
| Facebook | `fb-{fbId}` | 首次登入自動建立使用者；從 hometown/location 自動比對台灣縣市 |
| Google | `google-{sub}` | GIS One Tap，前端直接解碼 JWT |
| LINE | `line-{userId}` | OAuth 2.0 + PKCE（S256），verifier/state 暫存 sessionStorage |
| Strava | `strava-{athleteId}` | redirect → n8n webhook 換 token，state 暫存 sessionStorage |

### 粉絲頁身份切換
- `creatorId` 格式：個人 `fb-{fbId}`、粉絲頁 `page-{pageId}`
- authStore：`目前身份`（personal/page）、`切換到粉絲頁()`、`切換回個人()`、`取得目前發文身份()`
- 粉絲頁列表目前透過手動設定（FB Pages API 需 Business 類型 App + 審查）

### TCU 認證（LINE LIFF）
- 流程：個人中心申請 → 產生 6 位數認證碼（10 分鐘效期）→ LINE@ LIFF 頁面輸入認證碼
- LIFF Endpoint：`https://siokiu.criterium.tw/liff/verify`
- Supabase 表：`user_verifications`（token / status / user_id / line_user_id）
- 輪詢：`useVerificationPolling` hook，每 3 秒，10 分鐘後自動停止

### 會員等級（user_roles 表）
- Supabase 表 `user_roles`：`id` / `name` / `max_active_events` / `sort_order`
- 內建角色：`unverified`（預設）、`verified`（TCU 認證）、`admin`
- `max_active_events`：同時進行中活動上限；`roleService.ts` 的 `計算進行中活動數()` 含粉絲頁 creator_id
- 內建角色不可刪除；可在 Dashboard 新增自訂等級

## 資料持久化

**localStorage（Zustand persist）：**
- `約騎-auth` — 登入狀態 + 所有使用者列表
- `約騎-events` — 活動列表（coverImage 自動使用 stampImage）
- `約騎-region` — 區域選擇
- `約騎-templates` — 約騎範本列表

**Supabase（每次 mount 載入，不 persist）：**
`ride_templates` / `spot_templates` / `route_info_templates` / `notes_templates` / `saved_routes` / `user_verifications` / `user_roles` / `users`

**Supabase Storage：**
- Bucket `stamps`（Public）— 圖章路徑 `events/{eventId}.{ext}`
- 新增/更新活動時，base64 data URL 自動上傳取得公開 URL，存入 `cycling_events.cover_image`
- 本地 store 保留原始 base64 供 UI 顯示；公開 URL 供 Cloudflare OG image 使用

## Cloudflare Pages Functions

| 檔案 | 用途 |
|------|------|
| `functions/event/[[id]].ts` | OG 預覽：偵測爬蟲 UA → 動態 OG meta HTML；人類請求 `context.next()` |
| `functions/api/route-info.ts` | 從 Strava / Ride with GPS 抓路線距離與爬升；CORS 限縮為 `https://siokiu.criterium.tw` |

環境變數 `SUPABASE_ANON_KEY`（無 `VITE_` 前綴）供 Cloudflare 伺服器端使用。

## 環境變數

| 變數 | 說明 |
|------|------|
| `VITE_FB_APP_ID` | Facebook App ID |
| `VITE_GOOGLE_CLIENT_ID` | Google OAuth Client ID |
| `VITE_LINE_CHANNEL_ID` | LINE Login Channel ID |
| `VITE_STRAVA_CLIENT_ID` | Strava App Client ID |
| `VITE_STRAVA_CALLBACK_URL` | Strava token 交換用的 n8n webhook URL |
| `VITE_OAUTH_REDIRECT_URI` | LINE / Strava OAuth 回調 URI |
| `VITE_SUPABASE_URL` | `https://jxubndwcralkrbunxokf.supabase.co` |
| `VITE_SUPABASE_ANON_KEY` | Supabase anon key（前端，搭配 RLS） |
| `VITE_LIFF_ID` | LINE LIFF App ID（TCU 認證） |
| `SUPABASE_ANON_KEY` | 同上，供 Cloudflare Pages Function 使用（無 VITE_ 前綴） |

`.env` 不入版控。

## UI/UX 規範

- 所有按鈕加 `cursor-pointer` + `transition-colors duration-200`
- Button 元件內建 `focus-visible:ring` 聚焦狀態
- 觸控區域最小 44×44px（BottomNavBar）
- `<select>` 加 `name` 屬性；placeholder 結尾用 `…`（非 `...`）
- BottomNavBar 加 `touch-action: manipulation`
- 活動詳情頁返回鍵導向 `/wall`（非 history back）

## Testing

框架：**Vitest** + `@testing-library/react` + `@testing-library/jest-dom`

```bash
npx vitest run   # 單次執行（CI 用）
npx vitest       # watch 模式（開發用）
```

測試檔案放在被測模組旁邊，命名為 `*.test.ts`。

目前涵蓋：
- `src/stores/eventStore.test.ts` — 活動過期邏輯（12 小時緩衝）、便利貼顏色 / 旋轉角度 hash 確定性

**注意**：`time ?? '00:00'` 的防呆僅對 `null`/`undefined` 有效，**空字串** `''` 不會觸發預設值，會產生 NaN 日期（不過期），這是刻意行為（Supabase 欄位不會回傳空字串）。

## 設計系統色彩（`src/index.css` `@theme`）

| Token | 色碼 | 用途 |
|-------|------|------|
| `strava` | #FC4C02 | 主要強調色 / Strava 按鈕 |
| `line` | #00B900 | LINE 按鈕 |
| `google` | #4285F4 | Google 按鈕 |
| `facebook` | #1877F2 | Facebook 按鈕 |
| `region-north/central/south/east` | — | 四區域色（藍/橘/紅/綠） |
| `cork` | #D4A574 | 佈告欄背景 |
| `sticky-yellow/pink/blue/green` | — | 便利貼色 |

---
> Source: [TCUnion/taiwan-cycling-wall](https://github.com/TCUnion/taiwan-cycling-wall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
