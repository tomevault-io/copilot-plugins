## ah

> 本文件旨在讓不同的 AI 代理能快速理解專案狀態、目標、開發環境與接手步驟。

# AGENTS.md — ParkRelief 開發進度與接手指南

本文件旨在讓不同的 AI 代理能快速理解專案狀態、目標、開發環境與接手步驟。

---

## 1. 專案概述

**專案名稱**: ParkRelief — 可攜式長者按摩墊應用

**目標**: 為高齡者在戶外（如公園）提供隨時隨地的疼痛緩解與健康紀錄方案，透過前端 Web 應用與 GUN.js 去中心化資料庫實現即時同步與分享。

**開發方法**: TDD（測試驅動開發）—— 先寫測試，後寫程式碼。

**詳細需求**: 見 `PRD.md`

---

## 2. 當前開發進度

| 階段 | 任務 | 狀態 | 完成度 |
|------|------|------|--------|
| 需求 | 撰寫 PRD.md（需求規格書） | ✅ 完成 | 100% |
| 文檔 | 撰寫 AGENTS.md（接手指南） | 🔄 進行中 | 0% |
| 前端 | 建置 HTML/CSS/JS 樣板 | ⏳ 待開始 | 0% |
| 邏輯 | 實作 GUN.js 儲存與同步 | ⏳ 待開始 | 0% |
| 測試 | 撰寫並通過 Mocha 單元測試 | ⏳ 待開始 | 0% |
| 驗收 | 整體功能驗收與文檔確認 | ⏳ 待開始 | 0% |

---

## 3. 技術棧

### 前端環境
- **語言**: HTML5 / CSS3 / JavaScript (ES6+)
- **執行環境**: 現代瀏覽器（Chrome, Firefox, Safari, Edge）
- **框架**: 無框架（純原生 JS）
- **套件管理**: 無（使用 CDN）

### 核心依賴 (CDN)
```
- GUN.js (https://cdn.jsdelivr.net/npm/gun/gun.js)
  → 去中心化資料同步與儲存
  
- Mocha (https://cdn.jsdelivr.net/npm/mocha/mocha.js)
  → 測試框架
  
- Chai (https://cdn.jsdelivr.net/npm/chai/chai.js)
  → 斷言庫（驗證測試結果）
```

### 部署
- **開發方式**: 直接在瀏覽器開啟 `index.html`（需網路連接 CDN）
- **測試方式**: 在瀏覽器開啟 `tests.html` 執行 Mocha 測試
- **部署環境**: 靜態主機（GitHub Pages、Vercel、簡單 HTTP 伺服器）

---

## 4. 檔案結構

```
/workspaces/ah/
├── README.md              # 原始專案說明
├── PRD.md                 # 需求規格書（詳細功能、驗收準則、技術方案）
├── AGENTS.md              # 本文件（接手指南）
│
├── index.html             # 主應用頁面（產品介面）
├── styles.css             # 樣式表（UI 美化）
├── app.js                 # 核心應用邏輯（啟動/停止按摩、GUN 操作、UI 更新）
│
├── tests.html             # 測試頁面（Mocha 測試跑台）
└── tests/
    └── tests.js           # 單元測試程式碼（TDD 測試案例）
```

---

## 5. 核心功能與設計

### 5.1 應用流程

```
使用者打開 index.html
  ↓
看到「ParkRelief 產品介紹」 + 「開始按摩」按鈕
  ↓
點擊「開始按摩」
  ↓
選擇「部位」(腰部/背部/肩頸) + 「強度」(低/中/高)
  ↓
點擊「確認」
  ↓
頁面顯示「按摩中」與倒數計時（例如 15 分鐘）
  ↓
時間到或手動點「停止」
  ↓
自動建立 painEvent 事件物件 + 寫入 GUN.js
  ↓
UI 顯示「同步成功」/「同步失敗」
  ↓
使用者可點擊「分享連結」複製 GUN URL 給家人
```

### 5.2 核心函式（app.js 中應實作）

| 函式名稱 | 功能 | 參數 | 回傳值 |
|---------|------|------|--------|
| `createPainEvent()` | 建立痛點事件物件 | area, intensity, duration, notes | { id, timestamp, ...} |
| `saveToGUN()` | 儲存事件到 GUN | event 物件 | Promise (成功/失敗) |
| `loadFromGUN()` | 從 GUN 讀取事件 | 無 | Promise [events] |
| `toggleMassageStatus()` | 切換按摩狀態 | "start" / "stop" | void (更新 UI) |
| `updateUIStatus()` | 更新 UI 狀態顯示 | status 字串 | void |
| `generateShareLink()` | 生成分享連結 | 無 | URL 字串 |

### 5.3 GUN 資料模型

儲存路徑: `gun.get('ParkRelief').get('painEvents')`

事件結構:
```javascript
{
  "painEvent_1700000001": {
    id: "painEvent_1700000001",
    timestamp: 1700000001,
    location: "公園",
    painArea: "腰部",
    intensity: 7,
    duration: 15,
    notes: "背痛，按摩後緩解",
    deviceStatus: "已停止",
    syncStatus: "成功"
  },
  "painEvent_1700000002": { ... }
}
```

---

## 6. TDD 測試規劃

### 6.1 測試架構
- **測試檔案**: `tests/tests.js`
- **測試跑台**: `tests.html`（Mocha + Chai CDN）
- **運行方式**: 在瀏覽器開啟 `tests.html` 即自動執行

### 6.2 必須通過的 3 個基礎測試

#### Test 1: 建立事件物件結構驗證
```javascript
describe('PainEvent 物件', () => {
  it('應建立正確的事件結構', () => {
    const event = createPainEvent('腰部', 7, 15, '按摩後感覺好多了');
    expect(event).to.have.property('id');
    expect(event).to.have.property('timestamp');
    expect(event.painArea).to.equal('腰部');
    expect(event.intensity).to.equal(7);
  });
});
```

#### Test 2: GUN 儲存與讀取驗證
```javascript
describe('GUN 同步', () => {
  it('應成功儲存事件到 GUN 並讀取', (done) => {
    const event = createPainEvent('肩頸', 5, 10, '輕度');
    saveToGUN(event).then(() => {
      return loadFromGUN();
    }).then((events) => {
      expect(events).to.be.an('array');
      expect(events.length).to.be.greaterThan(0);
      done();
    }).catch(done);
  });
});
```

#### Test 3: UI 狀態切換驗證
```javascript
describe('UI 狀態管理', () => {
  it('應在按摩開始時改變狀態', () => {
    toggleMassageStatus('start');
    const statusElement = document.getElementById('status');
    expect(statusElement.textContent).to.include('按摩中');
  });
});
```

### 6.3 執行測試
```bash
# 在瀏覽器開啟測試頁面
# 1. 啟動本地 HTTP 伺服器（可選）
cd /workspaces/ah
python3 -m http.server 8000

# 2. 在瀏覽器打開
# http://localhost:8000/tests.html
# 或直接用 file:// 協議
# file:///workspaces/ah/tests.html

# 3. 觀察 Mocha 測試結果（綠色 ✓ 表示通過，紅色 ✗ 表示失敗）
```

---

## 7. 開發流程 (TDD 方式)

### 階段 1: 紅色 (Red)
1. 在 `tests/tests.js` 寫下測試，執行時會失敗（因為函式還未實作）
2. 瀏覽器 Mocha 顯示紅色 ✗

### 階段 2: 綠色 (Green)
1. 在 `app.js` 實作最小化的功能讓測試通過
2. 瀏覽器 Mocha 顯示綠色 ✓

### 階段 3: 重構 (Refactor)
1. 改進程式碼可讀性、註解、性能
2. 確保測試仍通過（保持綠色 ✓）

### 重覆進行
- 每新增功能，重複 Red → Green → Refactor

---

## 8. 如何接手與運行

### 8.1 快速開始（新手接手時）

#### Step 1: 檢視需求
```bash
# 閱讀需求規格書
cat PRD.md

# 理解目前進度與架構
cat AGENTS.md
```

#### Step 2: 檢查環境
```bash
cd /workspaces/ah

# 確認檔案存在
ls -la *.md *.html *.js tests/

# 可選：啟動本地 HTTP 伺服器
python3 -m http.server 8000
```

#### Step 3: 在瀏覽器查看
```
# 主應用
http://localhost:8000/index.html
或 file:///workspaces/ah/index.html

# 測試頁面
http://localhost:8000/tests.html
或 file:///workspaces/ah/tests.html
```

### 8.2 新增功能流程

假設要新增「使用者紀錄備註」功能：

1. **寫測試** (`tests/tests.js`):
   ```javascript
   it('應允許使用者輸入備註', () => {
     const notes = '背痛緩解';
     const event = createPainEvent('腰部', 5, 10, notes);
     expect(event.notes).to.equal('背痛緩解');
   });
   ```

2. **執行測試** (瀏覽器 `tests.html`):
   - 看到紅色 ✗（測試失敗）

3. **實作功能** (`app.js`):
   ```javascript
   function createPainEvent(area, intensity, duration, notes = '') {
     return {
       id: `painEvent_${Date.now()}`,
       timestamp: Date.now(),
       painArea: area,
       intensity: intensity,
       duration: duration,
       notes: notes,  // 新增備註欄位
       syncStatus: '新建'
     };
   }
   ```

4. **執行測試** (瀏覽器 `tests.html`):
   - 看到綠色 ✓（測試通過）

5. **重構與測試** (確保無迴歸):
   - 改進程式碼格式與註解
   - 重新執行全部測試，確保無損壞

### 8.3 常見問題排查

| 問題 | 原因 | 解決方案 |
|------|------|---------|
| GUN 連線失敗 | CDN 未載入或網路斷線 | 檢查瀏覽器控制台，確認 `gun.js` 已載入；檢查網路連線 |
| 測試無法執行 | Mocha/Chai CDN 未載入 | 在 `tests.html` 中檢查 CDN 連結；重新整理頁面 |
| UI 無反應 | JS 錯誤或事件綁定失敗 | 打開瀏覽器開發工具 (F12)，查看 Console 標籤中的錯誤 |
| 資料未同步 | GUN 連線或配置問題 | 打開多個瀏覽器視窗，檢查資料是否跨視窗同步 |

---

## 9. 程式碼風格指南

### 命名規範
- **函式**: camelCase (例如 `createPainEvent`, `toggleMassageStatus`)
- **常數**: UPPER_SNAKE_CASE (例如 `MAX_DURATION = 60`)
- **變數**: camelCase (例如 `currentStatus`, `eventList`)
- **HTML ID/Class**: kebab-case (例如 `id="pain-area-select"`)

### 註解風格
```javascript
// 簡短描述做什麼
const x = 5;

/**
 * 建立痛點事件物件
 * @param {string} area - 疼痛部位 (腰部/背部/肩頸)
 * @param {number} intensity - 強度等級 (1-10)
 * @param {number} duration - 按摩時間 (分鐘)
 * @param {string} notes - 可選備註
 * @returns {Object} 事件物件 {id, timestamp, painArea, ...}
 */
function createPainEvent(area, intensity, duration, notes = '') {
  // 實作...
}
```

### 錯誤處理
```javascript
// 使用 try-catch 捕捉非同步錯誤
async function saveToGUN(event) {
  try {
    // 儲存邏輯
    await gun.get('ParkRelief').set(event);
    updateUIStatus('同步成功');
  } catch (error) {
    console.error('GUN 儲存失敗:', error);
    updateUIStatus('同步失敗');
  }
}
```

---

## 10. 版本控制與提交

### Git 工作流
```bash
# 1. 查看當前分支與狀態
git status

# 2. 新建功能分支
git checkout -b feature/add-notes-input

# 3. 進行開發與測試（TDD 流程）

# 4. 提交變更
git add .
git commit -m "feat: 新增備註輸入功能，通過 3 個 Mocha 測試"

# 5. 推送與開 PR
git push origin feature/add-notes-input
```

### 提交信息格式
```
<type>: <subject>

<body>

<footer>
```

**Type** 可為: `feat` (功能), `fix` (修復), `test` (測試), `docs` (文檔), `refactor` (重構)  
**Subject**: 簡明描述，使用祈使句  
**Body**: 詳細說明（可選）  
**Footer**: 關聯 Issue/PR（可選）

---

## 11. 接手檢查清單

新的 AI 代理接手時，應完成以下檢查：

- [ ] 已閱讀 `PRD.md` 瞭解完整需求
- [ ] 已檢查 `/workspaces/ah/` 下所有檔案
- [ ] 已確認 `app.js` 中的核心函式清單
- [ ] 已理解 TDD 流程（Red → Green → Refactor）
- [ ] 已瞭解 GUN.js 資料模型與儲存路徑
- [ ] 已知道如何運行測試（瀏覽器 `tests.html`）
- [ ] 已查看 Git 提交歷史（若有的話）
- [ ] 能在瀏覽器正常開啟 `index.html` 與 `tests.html`

**若以上都確認完成，代表已準備好接手開發！**

---

## 12. 聯繫與支援

若需要協助或有疑問，請：
1. 先查看 `PRD.md` 的相關章節
2. 查看本文檔的「常見問題排查」段落
3. 在程式碼中加入 `console.log` 或使用瀏覽器開發工具 (F12) 除錯
4. 檢查 GUN.js 官方文檔: https://gun.eco/docs

---

## 13. 更新紀錄

| 日期 | 版本 | 更新內容 |
|------|------|---------|
| 2025-11-19 | 1.0 | 初版建立，涵蓋專案概述、技術棧、開發流程、接手指南 |

**最後更新**: 2025-11-19  
**狀態**: 待開始實作  
**下一步**: 建置 `index.html` / `styles.css` / `app.js` 樣板 + 撰寫基礎單元測試

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Phoebe624)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Phoebe624)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
