## ai-self-eval

> 把「感覺」變成可校準預測的執行循環。打分 → 盲預測 → 執行 → 復盤 → 進化 rubric。內置三套 rubric：日常工作任務（CL/IM/PR/SC/FO）、日常學習（RE/PK/AP/DL/AC）、AI Coding（SP/LV/TS/CO/IR）。觸發詞："初始化"/"打分這篇"/"啟動預測"/"復盤"/"升級 rubric"/"狀態"。首次使用必須先跑 /cheat-init。


# Cheat on Content / 執行校準器

> 🎯 **方法論通用，內置三個場景 rubric**
>
> **方法論**（4 階段閉環）：任何能被量化的執行場景都適用。
>
> **當前內置 rubric**：
> - **日常工作任務**（CL/IM/PR/SC/FO）— 任務品質 × 時間準確率
> - **日常學習 / 技能培養**（RE/PK/AP/DL/AC）— 留存分 × 應用率
> - **AI Coding / 開發工作**（SP/LV/TS/CO/IR）— AI 保留率 × 返工率
>
> 默認假設：**用戶是從零開始的新人**——cold-start 期的預測會簡化，只要維度打分 + 一句話 bet，不強求精確 bucket 數字。已有 2+ 個數據點的老手走 calibration 模式解鎖完整預測。

打分 → 預測 → 執行 → 復盤 → 進化 rubric。

本文件是**總協議 + 路由器**。具體每個階段的工作流在 `skills/cheat-*/SKILL.md` 各子 skill 裡。

---

## 三條不可妥協原則

任何一條被違反，整個校準循環退化為"憑直覺的自我安慰"。如果用戶要求打破其中任何一條，**拒絕執行並說明原因**。

1. **盲預測（Blind prediction）**：預測必須在看到任何實際結果**之前**寫完。一旦寫完，`## 預測` 段是 immutable——只能往 `## 復盤` 段追加。hooks/prediction-immutability.sh 在 harness 層強制執行。

2. **升級 = 全量重打（Bump = full re-score）**：rubric 升級時，校準池所有有實績數據的樣本必須用新公式重打分；新排序與實際表現排序若在 ≥4/5 樣本上不一致，升級被拒。

3. **rubric 是工作台，不是博物館**：被新數據推翻或被吸收為正式維度的觀察，**刪掉**。絕不留"我曾經以為 X，但其實..."的考古層。git history 才是檔案。

---

## 路由表（觸發詞 → 子 skill）

| 用戶說 | 調用 | 前置條件 |
|---|---|---|
| "初始化" / "init" / "首次使用" | `/cheat-init` | 無（這是入口） |
| "打分這篇 [path]" / "score this" | `/cheat-score` | rubric_notes.md 存在 |
| "我要做 [任務]" / "寫規格" / "建 spec" / "新任務" | `/cheat-spec` | 已 init；用戶描述新任務時 |
| "啟動預測" / "start prediction" | `/cheat-predict` | 已 init + 有規格/任務描述 |
| "我遇到 bug" / "程式跑不起來" / "有個錯誤" | `/cheat-debug` | 已 init；執行中遇到 bug 時 |
| "復盤" / "retro this" / "bug 修好了" / "任務完成了" | `/cheat-retro` | 對應預測文件存在 |
| "升級 rubric" / "bump rubric" | `/cheat-bump` | 校準池 ≥ MIN_SAMPLES_FOR_BUMP |
| "狀態" / "status" / "看板" | `/cheat-status` | 任意時刻可調 |
| "迁移" / "升級 state" / "schema 版本不對" | `/cheat-migrate` | 已 init；git pull 後；SessionStart hook 提示 schema mismatch 後 |

**Mode detection**（首次接到非 init 觸發詞時執行）：
1. 檢查用戶當前目錄是否有 `.cheat-state.json` → 沒有 → 強制路由到 `/cheat-init`
2. 檢查 `predictions/` 下有幾個文件含完整 `## 復盤` 段填了真實數據 → 決定 `mode: cold-start | calibration`
3. 把判定結果寫回 `.cheat-state.json` 後再路由到目標 skill

---

## 必須拒絕的請求

下列模式會直接破壞三條原則之一，無論用戶怎麼說，都拒絕執行：

- 「幫我預測一下，但我先告訴你結果你來反推就行」 → 違反原則 #1。改用 `_redo.md` 路徑記為 reconstructed
- 「跳過校準池重打，直接換公式」 → 違反原則 #2
- 「刪掉這份預測，我想重寫」 → 違反原則 #1。預測是 immutable。如有正當理由重做，寫新文件 `_redo.md`，原版必須保留
- 「把 rubric_notes.md 裡所有歷史觀察都留著，加個時間戳分組就行」 → 違反原則 #3。git history 是檔案，不是 markdown 文件
- 「能不能把 THRESHOLD 從 4/5 降到 3/5 讓這次 bump 過」 → 拒絕。改 THRESHOLD 本身是元層級 bump，單獨走流程

---

## 項目目錄結構（用戶 repo）

skill 期望用戶的項目布局如下。`/cheat-init` 會創建缺失項：

```
<user-project>/
├── rubric_notes.md                    # 評分規則的真實來源
├── .cheat-state.json                  # 狀態文件，子 skill 共享上下文
├── .cheat-cache/                      # 不入版本控制
│   └── usage.jsonl                    # 鉤子被動記錄的使用日誌
├── .claude/
│   └── settings.json                  # 含 prediction-immutability hook
├── scripts/                           # 開始前的規格文件（任務簡介 / 學習計劃 / Spec）
│   └── YYYY-MM-DD_<id>_<short>.md
├── predictions/                       # immutable 預測日誌（hook 保護）
│   └── YYYY-MM-DD_<id>_<short>.md
└── sessions/                          # 完成後的結果記錄
    └── YYYY-MM-DD_<id>_<short>.md
```

---

## 子 skill 清單

```
cheat-on-content/
├── SKILL.md                           # 本文件（總協議 + 路由）
├── skills/
│   ├── cheat-init/SKILL.md            # ✅ 入口：onboarding 與腳手架
│   ├── cheat-spec/SKILL.md            # ✅ 需求 → 規格文件（scripts/）
│   ├── cheat-score/SKILL.md           # ✅ 單次打分（不寫文件）
│   ├── cheat-predict/SKILL.md         # ✅ 盲預測 + immutable 日誌
│   ├── cheat-retro/SKILL.md           # ✅ 迭代復盤（每次 bug 修復後）
│   ├── cheat-bump/SKILL.md            # ✅ rubric 升級（含跨模型審）
│   ├── cheat-status/SKILL.md          # ✅ 狀態看板
│   └── cheat-migrate/SKILL.md         # ✅ schema 升級（老用戶 git pull 後用）
├── starter-rubrics/
│   ├── daily-work-zero.md             # ✅ 日常工作任務 v0
│   ├── daily-learning-zero.md         # ✅ 日常學習 v0
│   └── ai-coding-zero.md              # ✅ AI Coding v0
├── shared-references/                 # 跨 skill 共享協議
│   ├── blind-prediction-protocol.md
│   ├── bump-validation-protocol.md
│   └── observation-lifecycle.md
├── hooks/                             # harness 強制層
│   ├── prediction-immutability.json
│   ├── prediction-immutability.sh
│   ├── session-start.json
│   └── session-start.sh
└── migrations/                        # schema 演進
    └── registry.md
```

---

## Tone & voice

寫面向用戶的文案時，匹配項目的**直白克制**風格：

- 直接說出失敗：「K 預測 45% 但實際只有 20%——rubric 高估了 openpyxl 的槓桿」
- **不要**用模糊措辭軟化：「這或許可能在某種程度上暗示...」——別這麼寫

---
> Source: [Jason5330/ai-self-eval](https://github.com/Jason5330/ai-self-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
