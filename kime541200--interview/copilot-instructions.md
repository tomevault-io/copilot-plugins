## interview

> - 先讀檔與查證，再修改；只做目前需求必要的最小變更。

# AGENTS.md

## 工作原則

- 先讀檔與查證，再修改；只做目前需求必要的最小變更。
- 保持資料來源單一、路徑與 Markdown 結構穩定，避免建立會失同步的副本。
- 面試故事、數字、職責與成果只能來自使用者已確認的事實；未知資訊要標示為未知，不自行補寫。
- 修改後依對應 Skill 的驗證方式檢查，並回報未完成的外部步驟。

## Skill route

- 不確定目前階段、下一個 Skill 或如何繼續：讀 `ask-how-to`；只提供本專案 Skills 的唯讀路由建議，不自動執行其他 Skill。
- 首次使用、要求初始化或缺少 `my-resume/完整履歷.md`：讀 `initialize-interview-workspace`；只建立 headers-only 履歷骨架，不代填履歷內容。
- 面試準備、JD 對齊、準備計畫或速查表：先讀 `interview-preparation`。
- 啟動或進行互動式 mock interview：先讀 `start-mock-interview`。
- 穩定的 canonical Q&A 快速複習頁：讀 `render-mock-qa-html`。
- A/B/C 主動回憶、隨機抽題或進度追蹤：讀 `build-interview-practice-html`。
- ATS 履歷客製化：讀 `build-tailored-resume`。
- 面試簡報：讀 `build-interview-ppt`。

Python 指令與依賴安裝使用對應 Skill 目錄下的 `.venv`；不在全域環境安裝套件。若 Skill 文件與本文件出現流程細節差異，以較具體且較新的 Skill 文件為準。

## Q&A

- `ask-how-to` 可在任何階段調用，但只負責判斷目前狀態與推薦下一個本專案 Skill，不建立或修改工作區輸出。
- `mock_interview_qa.md` 是每個面試準備包唯一的 canonical Q&A source。
- `start-mock-interview` 擁有 QA card 的新增與更新；renderer Skills 只驗證與呈現，不重新分類題目。
- A/B/C、semantic ID 與 follow-up 關係由 canonical Markdown 維護；HTML 只負責呈現或練習，不建立第二份題庫。
- 穩定內容使用 `render-mock-qa-html`；主動回憶與進度追蹤使用 `build-interview-practice-html`。
- 互動式模擬面試由 `start-mock-interview` 管理：先取得使用者同意，一次只問一題；針對真實回答提供 STARS 精煉與具體回饋。

---
> Source: [kime541200/interview](https://github.com/kime541200/interview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
