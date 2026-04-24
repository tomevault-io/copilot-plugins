## postwallad

> **郵局牆面廣告版位出租銷售平台**（虎之呼吸科技有限公司）

# PostWallAd — 工作交接文件

## 專案說明
**郵局牆面廣告版位出租銷售平台**（虎之呼吸科技有限公司）
服務區域：基隆市全區、新北市汐止區、新北市瑞芳區

## 技術棧
- 純靜態前端：HTML / CSS / Vanilla JS
- Bootstrap 5.3.3 + Font Awesome 6（Kit: `https://kit.fontawesome.com/f6651e765b.js`）
- 後端：Google Apps Script（Code.gs）
- 資料庫：Google Sheets（Spreadsheet ID: `159dxUpnxrXp-V3uR4c6BqWhZUep-4_8MpENLI7dimeY`）
- 跨域方案：JSONP（Apps Script 不支援 CORS）

---

## 檔案結構
```
postwallad/
├── index.html       # 首頁（輪播、特色介紹、服務區域、三步驟）
├── locations.html   # 郵局列表頁（篩選、搜尋、卡片展示）
├── spaces.html      # 版位詳細頁（已完成）
├── site.js          # 主程式（輪播 + 郵局列表 + 版位頁邏輯）
├── spaces.js        # 舊版版位渲染（已廢棄，勿使用）
├── Code.gs          # Google Apps Script 後端 API
├── style.css        # 全站樣式（含列印、版位卡片、報價單）
└── CLAUDE.md        # 本文件
```

---

## Google Sheets 工作表欄位

### locations（郵局列表）
`location_id, 郵遞區號, 局名, 縣市, 行政區, 地址, 電話號碼, 封面圖網址, File ID, File ID圖片網址`

### ad_spaces（廣告版位）— 完整欄位（依截圖確認）
| 欄 | 欄位名稱 | 說明 |
|---|---|---|
| A | space_id | 版位編號 |
| B | location_id | 局所 ID |
| C | 寬cm. | 寬度 |
| D | 高cm | 高度 |
| E | 面積m2 | 面積 |
| F | 才數 | 才數 |
| G | 郵局廣告成本價格(整數面積*500) | 成本參考 |
| H | 出租價格 | 郵局底價 |
| **I** | **出租報價/月** | **月租金（報價計算用，乘月數）** |
| J | 印刷輸出材質 | 材質 |
| K | 印刷成本單價/才 | 單價 |
| L | 印刷成本 | 成本 |
| M | 印刷成本（含復原） | 成本×2 |
| N | 印刷輸出價格x2 | 印刷底價 |
| **O** | **印刷輸出報價** | **印刷費（固定，不乘月數）** |
| P | 吊車 | 吊車費 |
| **Q** | **吊車費用（裝＋拆＋道路申請）/1天** | **施工費（固定）** |
| R | 施工成本（裝＋拆） | 施工成本 |
| **S** | **施工報價** | **施工報價（固定）** |
| T | 總成本 | 內部成本 |
| U | 總報價 | 最終售價（黃色欄） |
| ... | File ID圖片網址 | 版位照片 |
| ... | 狀態下拉(可刊登/不可登) | 狀態 |

### 報價公式（重要）
```
總報價 = I（出租價格/月）× 月數 + O（印刷費）+ Q（施工費用）+ S（施工價格）
```
- **欄 I**：月租金，唯一乘以月數的項目
- **欄 O、Q、S**：固定費用，不隨月數變動
- 欄名可能含全/半形符號，統一用 `includes()` 模糊比對取值

範例：A020 × 1個月 = 15,000×1 + 15,000 + 5,000 + 7,500 = **87,500**
範例：A020 × 2個月 = 15,000×2 + 15,000 + 5,000 + 7,500 = **102,500**

### bookings（訂單）
`booking_id, space_id, 客戶, 開始日期, 結束日期, 月數, 成本, 總金額, 狀態（進行中／已結束）`

### Carousel-titile（輪播）
`Slogan, 行動號召, 封面圖網址, File ID, File ID圖片網址`

---

## API 端點（Code.gs doGet）
- `?action=carousel`   → 輪播圖資料
- `?action=locations`  → 郵局列表
- `?action=spaces`     → 版位列表
- `?action=bookings`   → 訂單資料
- 所有請求須帶 `&callback=函式名稱`（JSONP）

---

## 完成進度

### ✅ 已完成
- 首頁（index.html）：輪播圖、特色介紹、服務區域、三步驟、SEO / JSON-LD
- 郵局列表（locations.html）：JSONP 載入、縣市/行政區篩選、搜尋、「只顯示有可用版位」checkbox、卡片展示、可用版位數 badge
- 後端 API（Code.gs）：所有 action 路由正常
- 版位詳細頁（spaces.html）：郵局資訊 + Google Maps 嵌入、版位卡片列表、月份輸入即時報價、加入詢價/取消 toggle、詢價 Modal（含刪除單項）、圖片點擊放大
- Icon 系統：全站從 Bootstrap Icons 遷移至 Font Awesome 6（`fa-solid fa-*` 格式）

### ⚠️ 待確認
- ad_spaces 欄 I、O、Q、S 的確切欄位名稱（現用 `includes()` 模糊比對）
- 部分版位 NT$ 格式的數值是否由 Apps Script 以數字或字串回傳

---

## 重要程式碼慣例

```javascript
// JSONP 載入（8秒逾時保護）
jsonp('spaces', function(data) { ... });

// XSS 防護（所有使用者可見字串都要過）
escHtml(str)

// 版位可用性判定（模糊比對「狀態」欄位）
isAvailable(space)  // 狀態 === '可刊登'

// 圖片 URL 轉換（Drive → 可用格式）
validUrl(val)  // 回傳 lh3.googleusercontent.com 或 drive.google.com/thumbnail 格式

// 報價計算（I×月數 + O + Q + S，用 includes() 找欄位名）
calcSpaceTotal(space, months)

// 數值解析（處理 NT$ 格式字串或純數字）
parseNum(val)  // 去除非數字字元後 parseFloat
```

---

## 聯絡資訊
- 公司：虎之呼吸科技有限公司
- Email：service@breathtiger.com
- LINE：https://lin.ee/Rtr3Tlq4
- 官網：https://www.breathtiger.com

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/breathtiger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
