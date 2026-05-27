## hollywood-script-stack

> > 這個檔案是工作記憶。每個新 session 必須先讀這份文件再動任何東西。

# CLAUDE.md — 微短劇人機協作編劇系統

> 這個檔案是工作記憶。每個新 session 必須先讀這份文件再動任何東西。

---

## 這個系統是什麼

**一句話定位**：幫助創作者在投入生產之前，知道這個故事值不值得生產。

這是一個**思考機器，不是生產機器**。

小雲雀（ByteDance，2026/3）已解決劇本 → 成片的生產問題（視覺一致性、8 天完成 60 集）。我們不重新造那個輪子。

我們解決的是生產工具解決不了的問題：

| 問題 | 誰在解決 |
|---|---|
| 劇本 → 成片，視覺角色一致性 | 小雲雀 / Kling / Vidu |
| 這個概念值不值得做 | **本系統** |
| 這 80 集的情感架構是否成立 | **本系統** |
| 角色的心理一致性（不是視覺一致性）| **本系統** |
| 為什麼第 30 集之後故事失去動力 | **本系統** |
| 劇本找到觀眾（投流）| 不在本系統範圍 |

**爆款率 0.16% 的核心問題**：失敗的 99.84% 不是因為沒有用 AI 工具生產，是因為在生產之前，沒有人問過這個故事有沒有一個清晰的情感真相支撐它存在的理由。

---

## 第一原則：gstack 哲學（必讀）

每個 Skill 的工作是：

```
skill = 模式切換 + 對抗性提問 + 有立場的裁定
```

不是：

```
skill = 角色扮演 + 考慮清單 + 輸出模板
```

每個 Skill 設計規則：
1. **開場是一個對抗性問題**，不是介紹
2. **必須給裁定**（GO / PIVOT / DEAD，或具體修改指令）
3. **必須知道自己的邊界**：前置 Skill 未完成，拒絕執行
4. **長度：短**。不是教材，是工具。

---

## 人機協作模型

**AI 的三個角色**：
1. **挑戰假設**：問創作者不想被問的問題
2. **維護敘事狀態**：記錄「誰在第幾集知道什麼」，讓創作者專注於創意決策
3. **翻譯框架**：把好萊塢結構原則轉化為微短劇格式的具體工具

**人的三個角色**：
1. **所有故事決定**：角色、情感、動機——這是人的領域，不可外包
2. **文化判斷**：什麼在這個市場、這個時間點有情感共鳴
3. **創意突破**：在公式邊界找到差異化

**AI 永遠不告訴你寫什麼。AI 讓你清楚地看到你寫的東西的結構後果。**

---

## 核心框架（所有 Skill 的共同語言）

### 情感交易（Emotional Transaction）

每一集 90 秒是一次情感交易：

```
承諾（Offer）    0-15s   開場承諾某種情緒體驗
交付（Delivery） 15-75s  交付那個承諾，加上一個意外的轉折
驚喜（Surplus）         給了觀眾一個沒料到的情感額外收益
欠債（Debt）     最後15s 留下一個未解決的情緒張力——讓觀眾「欠著」
```

**核心洞察**：情感交易清晰的場景，同時是好故事，也是好廣告素材。內部衝突越清晰 → 表情越有力 → 截圖越能賣 → 這是同一件事。

### Scene Spine（場景脊椎）

每個場景用這個句子結構驗證：

> **[角色] 想要 [具體的東西]，但 [阻礙，且阻礙必須連結她的核心創傷]，所以她 [行動/決定]，結果 [情況改變了——一個維度變好，另一個維度變更壞]。**

關鍵條件：**不能只輸，不能只贏。每個場景的轉折必須是一個代價。**

### 視覺潛台詞（Visual Subtext）

「微短劇沒有潛台詞」是錯的。正確理解：**潛台詞必須是視覺性的，不能是對白性的。**

- ✅ 一個停頓、一個眼神、一隻顫抖的手——9:16 特寫格式讓這些比電影更有力
- ❌ 台詞說破感受（「我其實很在乎你」）——殺死潛台詞

禁止的是**說教和說破**，不是潛台詞本身。

### Cliffhanger 三溫度

- 🔥 **熱（Hot）**：立即外部威脅 / 地位衝擊。腎上腺素，轉化率高但衰減快。
- 🧊 **冷（Cold）**：安靜的揭示，改寫所有既有認知的意義。恐懼感，付費衝動最強，需要前期鋪墊。
- 💛 **溫（Tender）**：意外的連結 / 柔軟時刻，讓觀眾害怕接下來會失去它。焦慮感，最被低估。

**規則**：不能連續三集使用同一溫度。全劇沒有「溫」= 觀眾情感投資不足，中段流失率高。

### 好萊塢智慧的適用邊界

**適用（壓縮後有效）**：
- The Wound（角色核心創傷）——決定角色的每一個決定
- Want vs. Need（慾望 vs. 真實需求）——故事是填補這個落差的過程
- Information Architecture（資訊建築）——誰知道什麼，何時揭示，是懸念的底層
- Scene Economy（場景經濟）——每個場景必須同時做兩件事
- Stakes Escalation（賭注升級）——每個障礙讓前一個障礙看起來渺小

**不適用**：
- 三幕結構（分析工具，不是寫作單位）
- Blake Snyder Beats（Feature-only）
- Slow-burn character development（沒有時間）
- Subplot（幾乎沒有空間）

---

## Skill Stack 架構（v2 — 三個思考模式）

本系統不是線性流水線。三個模式可以從任意點進入：

```
DESIGN（設計）←→ EXECUTION（執行）←→ DIAGNOSTIC（診斷）
```

### 設計模式 Skills

| Skill | 核心問題 | 替代了 |
|---|---|---|
| `/story-core` | 這個故事有沒有存在的理由？（PREMIUM：情感真相 / MARKET：投流邏輯）| concept-test + trope-lock |
| `/story-architecture` | 這部劇的五個決定命運的時刻是什麼？ | episode-engine |

### 執行模式 Skill

| Skill | 核心問題 | 替代了 |
|---|---|---|
| `/episode` | 這一集在整部劇的架構裡是什麼位置？它的情感交易是什麼？| vertical-draft + hook-audit + cliff-audit |

### 診斷模式 Skill

| Skill | 核心問題 | 全新 |
|---|---|---|
| `/diagnose` | 故事從哪集開始失去動力？根本原因是三個之一：Character Drift / Escalation Stagnation / Information Collapse | ✅ |

### 精品升維 Skill（PREMIUM 限定）

| Skill | 核心問題 | |
|---|---|---|
| `/formula-break` | 在哪裡突破公式？突破多少是安全的？ | 保留 |

---

## 行業事實（時效性資訊，與原則分開）

> ⚠️ 以下資訊有時效性。原則（上方）是永久的，這裡的數字需要定期更新。

### 市場規模（2026 年）
- 中國 2026 預測：超過 1,200 億人民幣（$16.5B）；全球 2026 預測：$14B
- 用戶規模：6.96 億（截至 2025 年 6 月），佔中國網民近七成
- 2026 Q1 全行業上線 12.8 萬部，AI 生成佔 95% 以上
- 2026 年預測年產量：500 萬部（2025 年為 5 萬部，100 倍成長）
- 爆款率：**0.16%**

### 生產工具現況（2026 年）
- **小雲雀**（ByteDance，2026/3）：10 萬字劇本一鍵成片，Seedance 2.0 驅動，解決視覺角色一致性問題
- **Kling 3.0**（Kuaishou）、**Vidu**（Shengshu）：主要競爭工具
- 全 AI 生成：~2,000 人民幣 / 整部劇，1 人 1 天

### 成本結構
```
投流（買量廣告）：82.5%
製作成本：       7.5%
演員片酬：       5%
技術/平台費：    3.5%
劇本相關：       1.5%
```

### 一集結構（不變）
```
Hook(10-15s) + Development(60-90s) + Cliffhanger(10-15s)
總時長：90-120 秒，9:16 垂直格式，60-80% 鏡頭特寫
```

### 5 Block 結構（基於付費節點）
```
Block 1（第 1-10 集）：免費鉤子，第 5-8 集第一個爽點
Block 2（第 11-30 集）：第一個付費牆，衝突升級
Block 3（第 31-50 集）：中段反轉，至少一次改寫認知的揭示
Block 4（第 51-70 集）：加速，外部 + 情感衝突同時推進
Block 5（第 71-100 集）：高潮收束，預留第二季鉤子
```

### 監管現況
- 中國 NRTA 強制備案；紅線：不健康婚戀觀、仇富情緒、違反社會主義核心價值觀

---

## 好萊塢行業事實（參考知識）

- Feature：Pitch → Option → Step Deal（Outline → Draft → Rewrite → Polish）
- TV Writers' Room：Blue Sky → Story Area → Outline → First Draft；三個 approval gate
- Writers' Room 層級：Staff Writer → Story Editor → ... → Showrunner

---

## 檔案結構

```
/                           # Hollywood Script Stack（待重寫）
│
├── micro-drama/            # 微短劇人機協作編劇系統（v2）
│   ├── story-core/SKILL.md         # 設計：概念 + 套路 + 路線
│   ├── story-architecture/SKILL.md # 設計：五個槓桿時刻 + 資訊建築圖
│   ├── episode/SKILL.md            # 執行：單集寫作（整合 draft + audit）
│   ├── diagnose/SKILL.md           # 診斷：三個根本原因框架
│   ├── formula-break/SKILL.md      # 精品升維（PREMIUM 限定）
│   └── develop/SKILL.md            # 完整流程總覽
│
├── docs/
│   ├── workflow.md
│   └── micro-drama-research.md
│
├── README.md
├── CLAUDE.md
└── .github/workflows/validate.yml
```

---

## 不要做的事

- 不要在小雲雀已解決的問題上重新造輪子（視覺一致性、生產流水線）
- 不要假裝本系統能解決投流問題——它不能
- 不要讓任何 Skill 變成「考慮清單 + 輸出模板」
- 不要在設計模式的 Skill 沒有完成的情況下進入執行模式
- 不要把行業事實（時效性）和框架原則（永久性）混在一起

---

## 關鍵參考資料

- gstack repo: https://github.com/garrytan/gstack
- 小雲雀 AI: https://xyq.jianying.com/
- Seedance 2.0: https://seed.bytedance.com/en/blog/official-launch-of-seedance-2-0
- WGA Basic Agreement 2023: https://www.wga.org/contracts/contracts/mba

---
> Source: [vincertp/hollywood-script-stack](https://github.com/vincertp/hollywood-script-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
