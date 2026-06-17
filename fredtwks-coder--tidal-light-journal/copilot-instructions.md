## tidal-light-journal

> 汐光旅冊 Tidal-Light-Journal

# AGENTS.md

## 專案名稱

汐光旅冊 Tidal-Light-Journal

## 專案定位

汐光旅冊是一個把現實學習任務轉化成旅程體驗的網站型學習遊戲。

核心閉環：

```text
任務卡 → 專注計時 → 旅途偏離紀錄 → 成果反思 → 旅光幣簡化結算 → 旅冊寫入 → 首頁顯示今日進度
```

## 回覆與程式註解語言

- 請使用繁體中文說明修改內容。
- 程式碼命名使用英文。
- UI 顯示文字使用繁體中文。
- 程式註解可使用繁體中文，但函式、變數、檔案、資料型別請使用英文命名。

## 技術限制

V1-0 階段固定使用：

- React
- Vite
- TypeScript
- React Router
- localStorage
- CSS / CSS Modules

V1-0 階段不得使用：

- 後端 API
- 雲端同步
- IndexedDB
- 帳號登入
- 圖片上傳
- PWA
- 抽卡
- 戰鬥
- 排行榜
- 社群

## V1-0 目標

只完成核心閉環：

```text
任務卡 → 專注計時 → 旅途偏離紀錄 → 成果反思 → 旅光幣簡化結算 → 旅冊寫入 → 首頁顯示今日進度
```

V1-0 的重點是「可執行、可測試、可保存資料」，不是完整正式版產品。

## 必須遵守的產品原則

1. 汐光旅冊是「學習工具為主，遊戲回饋為輔」。
2. 任務是中樞，專注、成果、反思、獎勵與旅冊都應盡量連回任務卡。
3. 遊戲化回饋應該放在學習成果之後，不要讓金幣或獎賞蓋過學習本身。
4. 偏離紀錄是低壓自我回報，不是監控或懲罰。
5. 旅冊是長期價值，必須能保存任務、專注、反思與獎勵紀錄。
6. 首頁應讓使用者快速知道「今天可以做什麼」，不要做成複雜手遊大廳。
7. 專注頁應極簡，不要放太多干擾資訊。
8. V1-0 先用 placeholder 圖與簡化樣式，不追求最終美術。

## 開發規則

- 不要自行加入未列在 `SPEC_V1_0.md` 的功能。
- 每次修改前先說明要改哪些檔案。
- 每次修改後列出修改摘要。
- 優先保持程式簡單、可讀、可維護。
- UI 先用 placeholder 圖與簡化樣式，不追求最終美術。
- 資料模型必須符合 `SPEC_V1_0.md`。
- 金幣必須透過 `RewardTransaction` 記錄，不可只直接修改 `WalletSnapshot`。
- 計時器結算必須用時間戳，不可只用 `seconds += 1` 作為最終結算依據。
- localStorage key 固定使用 `tidal_app_state`。
- 若新增元件，請放到對應資料夾。
- 若需要調整資料模型，必須先說明原因與影響範圍。
- 不要刪除既有功能，除非先說明原因並保留原行為。

## 執行指令限制

- Codex 可以執行 `npm run build` 作為主要驗證。
- 除非使用者明確要求，Codex 不要自動執行 `npm run dev`。
- 若需要確認 dev server，請只說明使用者可自行執行 `npm run dev`，並在看到 Local URL 後自行停止。
- Codex 完成每個 Phase 時，以 `npm run build` 成功作為主要驗證依據。
- 不要讓 Codex 任務長時間等待 `npm run dev`，因為 dev server 本來就會持續執行。

## 禁止事項

V1-0 階段不要實作：

- 後端 API
- 登入 / 註冊
- 雲端同步
- IndexedDB
- 圖片上傳
- 頭像裁切
- 頭像框
- PWA
- 抽卡
- 戰鬥
- 排行榜
- 社群
- 完整商城
- 完整地圖系統
- 完整角色好感度
- 完整自訂獎賞系統
- 完整日曆拖移排程
- 複雜成就系統
- 真實金流

## 建議資料夾結構

```text
src/
  assets/
    images/
  components/
    common/
    layout/
    tasks/
    focus/
    journal/
    settings/
    dashboard/
  pages/
    HomePage.tsx
    TasksPage.tsx
    FocusPage.tsx
    JournalPage.tsx
    SettingsPage.tsx
  hooks/
  data/
  utils/
  styles/
  types/
```

## 必要路由

V1-0 只建立以下五個主要路由：

```text
/          HomePage
/tasks     TasksPage
/focus     FocusPage
/journal   JournalPage
/settings  SettingsPage
```

## 工作回報格式

每次完成修改後，請回報：

1. 修改了哪些檔案。
2. 新增了哪些檔案。
3. 每個檔案用途。
4. 如何測試。
5. 是否能 `npm run dev`。
6. 下一階段建議。

---
> Source: [fredtwks-coder/Tidal-Light-Journal](https://github.com/fredtwks-coder/Tidal-Light-Journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
