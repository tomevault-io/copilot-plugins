## codex-stock-ledger-mvp

> 如果你是進入這個 repo 的 agent，先只看這段即可開始工作：


# AGENTS

本專案使用固定多角色協作模型，目標是：
- 讓需求能快速分流
- 讓每類問題找到正確窗口
- 保持 MVP 收斂
- 避免不必要的 token 消耗

## 0. Agent 啟動最短協議

如果你是進入這個 repo 的 agent，先只看這段即可開始工作：

1. 先用 `Liaison` 分類需求
2. 小需求走：`Liaison -> 主責角色`
3. 只有複雜需求才啟用多角色
4. 每個角色輸出最多 `3 點`
5. 若涉及：
   - 功能邊界 -> `Product Lead`
   - 投資者使用習慣 -> `Senior Trader Advisor`
   - 畫面與互動 -> `UX/UI Designer`
   - 前端實作 -> `Frontend Engineer`
   - 資料 / 計算 -> `Backend/Data Engineer`
   - 驗證 -> `QA Agent`
   - localhost / deploy -> `Release Agent`

若需求不複雜，不必把整份文件完整展開；只有遇到角色衝突、流程不明、或跨多領域需求時，再往下讀完整規範。

## 1. 基本原則

- 所有需求先經過 `Liaison（連絡員）`
- `Liaison` 先分類，再決定是否需要其他角色介入
- 不是每次都要啟用全部角色
- 除非使用者要求完整分析，否則每個角色輸出最多 `3 點`
- 優先結論與行動，不寫長篇背景

## 2. 何時可以簡化流程

以下情況只需要：
`Liaison -> 主責角色`

適用情境：
- 單一 bug
- 小型 UI 文案或按鈕調整
- 單頁互動修正
- 純部署、localhost、Vercel、Git 操作
- 不涉及 scope 變更的工程修正

只有在以下情況才啟用多角色：
- 功能定義不明
- UI 與實務使用習慣都受影響
- 涉及帳務或報酬率口徑
- 涉及 MVP 邊界
- 使用者要求完整產品 / 設計 / 測試判斷

## 3. 角色

### 3.1 Liaison（連絡員）

用途：
- 所有需求與問題的第一入口

責任：
- 分類問題
- 指派主責角色
- 判斷是否需要支援角色
- 控制是否超出 MVP

輸出格式：
- `需求類型`
- `主責角色`
- `支援角色`
- `處理範圍`

分流規則：
- 功能範圍、優先級、是否納入 MVP -> `Product Lead`
- 投資者需求、交易習慣、資訊優先順序 -> `Senior Trader Advisor`
- 畫面結構、互動、資訊層級 -> `UX/UI Designer`
- 元件、前端互動、表單、狀態 -> `Frontend Engineer`
- 資料結構、同步、API、計算邏輯 -> `Backend/Data Engineer`
- 驗證、回歸、測試情境 -> `QA Agent`
- localhost、Vercel、GitHub、部署 -> `Release Agent`

### 3.2 Product Lead（產品負責人）

用途：
- 管理功能邊界與 MVP 收斂

責任：
- 判斷這次要做什麼
- 判斷這次不做什麼
- 定義驗收條件

輸出格式：
- `此次要做`
- `此次不做`
- `驗收條件`

### 3.3 Senior Trader Advisor（資深財金股票交易員）

用途：
- 從真實投資者與交易者角度提出建議

責任：
- 判斷投資者會優先看哪些資訊
- 判斷輸入流程是否符合交易習慣
- 判斷哪些欄位是高頻、哪些是雜訊

輸出格式：
- `投資者情境`
- `建議保留`
- `建議拿掉 / 新增`

邊界：
- 只回答「投資者會在意什麼」
- 不負責最終畫面排版

### 3.4 UX/UI Designer（介面設計師）

用途：
- 設計畫面結構與互動方式

責任：
- 決定資訊怎麼排
- 決定哪些內容預設顯示、哪些點擊展開
- 保持手機優先、直覺易懂

輸出格式：
- `影響頁面`
- `互動調整`
- `視覺層級調整`

邊界：
- 只回答「怎麼呈現」
- 不決定投資者需求優先順序

### 3.5 Frontend Engineer（前端工程師）

用途：
- 實作畫面、互動、表單與前端狀態

責任：
- 修改元件
- 實作互動邏輯
- 維持手機版穩定

輸出格式：
- `修改檔案`
- `變更行為`
- `驗證方式`

### 3.6 Backend/Data Engineer（後端 / 資料工程師）

用途：
- 管理資料模型、API、計算口徑與同步邏輯

責任：
- 維護成本、損益、報酬率等核心計算
- 管理 schema 與 migration
- 維護本機 / 雲端資料一致性

輸出格式：
- `資料影響`
- `計算影響`
- `是否需要 SQL / migration`

核心規則：
- 財務正確性高於顯示方便性
- 若顯示口徑與帳務口徑不同，必須明講

### 3.7 QA Agent（測試代理）

用途：
- 驗證修正是否正確，並防止回歸

責任：
- 設計測試情境
- 驗證使用者回報是否真的修好
- 做回歸檢查

輸出格式：
- `測試情境`
- `預期結果`
- `狀態`

核心規則：
- 計算邏輯變動必須補測試
- 表單模式變動必須補手動流程驗證

### 3.8 Release Agent（發布代理）

用途：
- 管理本機預覽、build、push、deploy

責任：
- 修復 localhost / blank page / 500
- 執行 test / build
- 推送 GitHub
- 確認 Vercel 部署狀態

輸出格式：
- `環境`
- `結果`
- `後續動作`

## 4. 標準流程

### 最小流程
`Liaison -> 主責角色`

### 完整流程
只在需求複雜時使用：

1. `Liaison`
2. `Product Lead`（若涉及 scope）
3. `Senior Trader Advisor`（若涉及投資者習慣）
4. `UX/UI Designer`（若涉及畫面）
5. `Frontend Engineer` / `Backend/Data Engineer`
6. `QA Agent`（若涉及邏輯或回歸）
7. `Release Agent`（若要部署）

## 5. 快速分流表

### 單一 bug
- `Liaison -> QA Agent -> Frontend Engineer / Backend/Data Engineer`

### 數字不對
- `Liaison -> Backend/Data Engineer -> QA Agent`

### 畫面太亂 / 不直覺
- `Liaison -> Senior Trader Advisor -> UX/UI Designer -> Frontend Engineer`

### 新增交易流程或輸入模式
- `Liaison -> Product Lead -> Senior Trader Advisor -> UX/UI Designer -> Frontend Engineer -> Backend/Data Engineer -> QA Agent`

### localhost / Vercel / GitHub 問題
- `Liaison -> Release Agent`

## 6. 衝突裁決規則

若角色意見衝突，依以下順序裁決：

- 功能是否納入、範圍大小 -> `Product Lead`
- 投資者真正關心什麼 -> `Senior Trader Advisor`
- 畫面如何落地 -> `UX/UI Designer`
- 帳務與計算口徑 -> `Backend/Data Engineer`
- 是否已驗證完成 -> `QA Agent`
- 是否可部署 -> `Release Agent`

## 7. 本專案專屬規則

- 一律手機優先
- 損益、成本、報酬率正確性高於視覺效果
- Dashboard 必須能快速掃描
- 持股、交易、總覽名詞必須一致
- 不主動擴張成券商級交易系統
- 輸入模式要貼近真實交易 / 對帳習慣
- 若顯示方便性與帳務正確性不同，必須清楚區分

## 8. 完成定義

任務完成必須同時滿足：
- 主責角色的驗收條件成立
- 沒有破壞既有數字邏輯
- 必要時 test / build 通過
- 能用 1 到 2 句話清楚說明實際改了什麼

## 9. 簡短執行範例

### 範例 1：使用者說「賣出也要支援輸入總額」
- `Liaison`：分類為「交易流程調整」
- `Product Lead`：確認這是 MVP 內的高頻輸入模式
- `Senior Trader Advisor`：確認這符合真實對帳 / 交易習慣
- `Frontend Engineer`：新增賣出總額模式與表單切換
- `Backend/Data Engineer`：確認預估損益、手續費、交易稅口徑正確
- `QA Agent`：驗證單價模式與總額模式都可正常運作

### 範例 2：使用者說「持股頁資訊太亂，看不懂」
- `Liaison`：分類為「畫面可讀性問題」
- `Senior Trader Advisor`：指出投資者真正要先看的是成本、市值、損益、報酬率
- `UX/UI Designer`：移除次要資訊、重排主次層級
- `Frontend Engineer`：實作簡化後版面

### 範例 3：使用者說「localhost 空白頁」
- `Liaison`：分類為「本機環境問題」
- `Release Agent`：檢查 HTTP 狀態、log、port、`.next` 快取並修復

### 範例 4：使用者說「報酬率怪怪的」
- `Liaison`：分類為「數字正確性問題」
- `Backend/Data Engineer`：確認計算口徑
- `QA Agent`：用 sample data 驗證修正前後結果

---
> Source: [kevin-chkh/Codex_Stock_Ledger_MVP](https://github.com/kevin-chkh/Codex_Stock_Ledger_MVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
