## gomoku-plan-docs

> 以純 Web 端五子棋遊戲作為第一個試水作品，驗證開發流程、建立完整遊戲體驗、探索未來的技術儲備。同步撰寫開發日誌（devlog）作為個人品牌內容。

# Gomoku Web Game — 專案規劃

> v0.0.1
> 2026-02-20

## 專案目標

以純 Web 端五子棋遊戲作為第一個試水作品，驗證開發流程、建立完整遊戲體驗、探索未來的技術儲備。同步撰寫開發日誌（devlog）作為個人品牌內容。

---

## Repo 結構

| Repo | 說明 | 可見性 |
|------|------|--------|
| `gomoku-plan-docs` | 專案規劃、開發日誌 | Public |
| `gomoku-client` | 前端：遊戲畫面、互動邏輯、AI 對戰 | Public |
| `gomoku-server` | 後端：多人配對、房間管理、對戰同步 | Public |

AI 第一版跑在 client 端（Web Worker），不需要獨立 repo。

> 跨 repo 的工程規範（commit format、scope、分支命名、安全規範）統一收斂在 `ENGINEERING-CONVENTIONS.md`。

---

## 技術選型

| 層級 | 選擇 | 原因 |
|------|------|------|
| 前端框架 | Vue 3 + TypeScript | 最熟悉的技術棧 |
| 棋盤渲染 | Canvas 2D | 五子棋視覺需求單純，夠用且輕量 |
| AI 運算 | Web Worker | Minimax + Alpha-Beta 不卡 UI |
| 即時通訊 | FastAPI 原生 WebSocket | 回合制只傳座標，不需要 Socket.IO 的重功能，輕量好 debug |
| 後端 | FastAPI (Python) | async 友善、輕量，順便練 Python |
| 容器化 | Docker + docker-compose（本機開發用） | 練手感、環境一致，部署交給平台自動處理 |
| 部署 | 前端 Vercel / 後端 Zeabur | Zeabur 免費方案含 $5/月額度，原生支援 FastAPI，台灣團隊 |

> **為什麼不用 Socket.IO？** 五子棋是回合制，每步就傳一個座標，不需要 room / namespace / 自動重連等重功能。前端用瀏覽器原生 `WebSocket` API 即可，少一層抽象。未來做隨機配對、觀戰時再升級到 python-socketio。
>
> **為什麼選 Zeabur 而不是 Railway / Fly.io？**
> - Zeabur Free Trial 不需綁卡，每月 $5 額度（1 vCPU / 2GB RAM），MVP 階段夠用
> - 原生偵測 FastAPI，不需要 Dockerfile 就能部署
> - 台灣團隊，中文文件與社群支援，踩坑溝通成本低
> - Railway 免費 trial 只有 30 天一次性 $5；Fly.io WebSocket 閒置 10 分鐘斷線需額外處理
>
> **Docker 定位：** 本機開發用 docker-compose 保持學習目標，線上部署讓 Zeabur 自動偵測處理，兩條路不衝突。

---

## 第一版 MVP 功能（四週上線）

### Phase 1：單機對戰（第 1-2 週）

- [ ] 15×15 棋盤 Canvas 渲染與落子互動
- [ ] 勝負判定（五連線，先不做禁手規則）
- [ ] Minimax + Alpha-Beta AI 對手（搜尋深度 4 層）
- [ ] AI 運算使用 Web Worker，避免畫面凍結
- [ ] 難度選擇（Easy = 深度 2 / Normal = 深度 4 / Hard = 深度 6）
- [ ] RWD 手機可玩

### Phase 2：多人對戰（第 3 週）

- [ ] FastAPI 原生 WebSocket 連線，房間制
- [ ] 建立房間 → 產生連結 → 分享 → 加入即開打
- [ ] 回合制狀態同步（JSON 訊息傳座標）
- [ ] 斷線重連機制（前端偵測斷線後自動重連 + 狀態恢復）
- [ ] 回合計時器（每步 30 秒，防掛機）

### Phase 3：基礎體驗打磨（第 4 週）

- [ ] 落子音效、簡單動畫回饋
- [ ] 對局結束畫面（勝負結果、重來一局）
- [ ] 分享連結 OG meta（貼到 LINE / Discord 有預覽）
- [ ] 部署上線

---

## 未來擴充方向

### 第二階段：驗證留存

- 帳號系統（Google OAuth）
- ELO 積分排名
- 隨機配對（不只朋友對戰）
- 對局紀錄回放
- 升級 WebSocket 方案至 python-socketio + Redis adapter（支援多房間 / 配對隊列）

### 第三階段：驗證付費

- 自定棋盤主題 / 棋子外觀（付費解鎖）
- 賽季制排名
- 觀戰功能

### 第四階段：擴展玩法

- 五子棋變體規則（禁手、Swap2 開局）
- 異步對戰（下完一步關掉，對方有空再回）
- 更強 AI（搬到 server 端跑更深搜尋）

---

## 第一週 Task（Server 端）

目標：建立後端基礎架構，能跑起來、能接 WebSocket。

### Day 1-2：專案初始化

- [ ] 建立 `gomoku-server` repo
- [ ] 初始化 Python 專案結構：
  ```
  gomoku-server/
  ├── Dockerfile
  ├── docker-compose.yml
  ├── requirements.txt
  ├── app/
  │   ├── main.py
  │   ├── game.py
  │   ├── room.py
  │   └── ws_handlers.py
  ```
- [ ] 撰寫 Dockerfile 與 docker-compose.yml（本機開發用）
- [ ] `docker compose up` 能成功啟動 FastAPI server
- [ ] 設定基本的 CORS 與 health check endpoint
- [ ] 確認 Zeabur 能自動偵測並部署（push 到 GitHub 測試）

### Day 3-4：房間系統

- [ ] 實作房間資料結構（room_id, players, board_state, current_turn）
- [ ] WebSocket endpoint：`/ws/{room_id}` — 連線進入指定房間
- [ ] 訊息格式定義（JSON）：`create_room` / `join_room` / `leave_room`
- [ ] 房間滿兩人自動開始遊戲
- [ ] 用 dict 存房間狀態（第一版不需要 DB）

### Day 5-6：遊戲邏輯

- [ ] 實作棋盤狀態管理（15×15 二維陣列）
- [ ] WebSocket 訊息：`place_stone` → 接收座標、驗證合法性、廣播給對手
- [ ] 實作勝負判定（檢查橫/直/斜四個方向的五連線）
- [ ] 回合控制（輪流下棋、防止非自己回合落子）

### Day 7：測試與收尾

- [ ] 用簡單的 HTML client 手動測試 WebSocket 連線
- [ ] 確認兩個 client 能連進同一房間、輪流下棋、判定勝負
- [ ] 撰寫第一篇 devlog

---

## 第一週 Task（Client 端，可同步進行）

目標：把棋盤畫出來，能單機落子。

### Day 1-3：棋盤渲染

- [ ] 建立 `gomoku-client` repo（Vue 3 + TypeScript + Vite）
- [ ] Canvas 2D 繪製 15×15 棋盤格線
- [ ] 點擊 Canvas 計算對應格子座標
- [ ] 落子渲染（黑白交替）
- [ ] 高亮最後一手落子位置

### Day 4-5：基礎遊戲邏輯

- [ ] 前端勝負判定（與後端獨立，單機模式用）
- [ ] 禁止重複落子
- [ ] 遊戲結束畫面（顯示勝負、重新開始按鈕）
- [ ] RWD 處理（Canvas 尺寸自適應）

### Day 6-7：AI 雛形

- [ ] 建立 Web Worker 基礎結構
- [ ] 實作最簡單版 Minimax（深度 2，先能動就好）
- [ ] 評估函數 v1（連線模式計分：活四 > 死四 > 活三 > 活二）
- [ ] 人機對戰模式能完整跑一局

---

## AI 回應時間標準

| 等級 | 搜尋深度 | 目標回應時間 | 備註 |
|------|---------|-------------|------|
| Easy | 2 | < 200ms | 即時感 |
| Normal | 4 | < 800ms | 可接受 |
| Hard | 6 | < 2s | 上限，超過需降深度或優化 |

- 以中低階手機為測試基準（不是你的開發機）
- 超過 3 秒視為體驗失敗，需加 loading 動畫或降級處理
- 第一版先用 `performance.now()` 計時，記錄不同裝置的表現

---

## 部署成本估算

| 服務 | 方案 | 月費 | 免費額度 | 備註 |
|------|------|------|---------|------|
| Vercel（前端） | Free | $0 | 100GB 流量/月 | 靜態站很難超過 |
| Zeabur（後端） | Free Trial | $0 | $5 用量/月，1 vCPU / 2GB RAM | 不需綁卡 |
| Zeabur（後端） | Developer | $5/月 | 含 $5 額度，2 vCPU / 4GB RAM | 超過免費額度時升級 |

- WebSocket 長連線吃記憶體，每條連線約 10-50KB，2GB RAM 理論上足夠 MVP 階段
- Zeabur 按實際用量計費（CPU / Memory / Network），五子棋回合制流量極低
- 網路流量免費額度 100GB/月，五子棋每步只傳幾十 bytes，幾乎不可能超過
- 第一版目標：月支出 $0，用 Free Trial 撐過驗證期
- 設定 Zeabur Project Budget 提醒，避免意外超額

---

## 數據追蹤（上線時加入）

最小可用指標，用 GA4 或簡單 event logging：

- [ ] 每日開局數（DAU 的近似值）
- [ ] AI 對戰 vs 多人對戰比例（判斷哪個模式有需求）
- [ ] 平均一局時長（太短 = 太簡單，太長 = 可能有問題）
- [ ] 房間連結轉換率（分享數 vs 實際加入數，衡量社交傳播力）
- [ ] AI 難度分佈（玩家偏好哪個難度，調整預設值）

不需要第一天就完美，上線時埋基本事件就好，之後再迭代。

---

## License

採用 **MIT License**。原因：

- 最寬鬆，不嚇跑潛在貢獻者
- 五子棋原始碼不是商業壁壘，產品判斷和迭代速度才是
- 兩個 repo 統一用 MIT

---

## Devlog 規劃

每個里程碑寫一篇，預計第一個月產出 3-4 篇：

1. **專案啟動** — 為什麼選五子棋、技術選型思路、架構設計
2. **棋盤與 AI** — Canvas 渲染、Minimax 實作過程（不懂演算法的前端怎麼用 AI 輔助搞定）
3. **多人連線** — WebSocket 房間制設計、Python 後端踩坑記錄
4. **上線覆盤** — Zeabur 部署流程、第一批使用者回饋、下一步計畫

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wuzhe0912) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
