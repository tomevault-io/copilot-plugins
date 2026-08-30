## myainote

> > **此檔案是本 wiki 的憲法。每次與本 vault 互動前先讀本檔，並嚴格遵循其中規則。**

# LLM Wiki — 第二大腦架構規則

> **此檔案是本 wiki 的憲法。每次與本 vault 互動前先讀本檔，並嚴格遵循其中規則。**
> 你（LLM）負責所有寫入、整理、cross-link、bookkeeping。使用者只負責找來源、提問、決策方向。

---

## 三層架構

| 層     | 路徑                | 誰寫     | 性質                   |
| ------ | ------------------- | -------- | ---------------------- |
| Raw    | `raw/`              | 使用者   | 原始來源，**永不修改** |
| Wiki   | `wiki/`             | LLM      | 合成知識，持續演化     |
| Schema | `CLAUDE.md`（本檔） | 雙方協作 | 規則、慣例、workflow   |

外加兩個導航檔：

- `index.md` — 內容目錄（content-oriented）
- `log.md` — 時序日誌（append-only）

---

## 資料夾結構

```text
Notes/
├── CLAUDE.md              # 本檔（schema）
├── index.md               # 內容目錄
├── log.md                 # 時序日誌
├── raw/
│   ├── tech/              # 技術類來源（依主題資料夾）
│   │   ├── devops/
│   │   │   ├── argocd-tutorial/
│   │   │   │   └── argocd-tutorial.md
│   │   │   └── Docker/
│   │   │       ├── Docker/
│   │   │       │   └── Docker.md
│   │   │       └── Docker 部屬範例/
│   │   │           ├── Docker 部屬範例.md
│   │   │           └── (images)
│   │   └── ...（各主題皆為同名資料夾，文章與圖片同層）
│   ├── Languages/         # 程式語言
│   ├── ASP Web API/
│   └── 筆記/
└── wiki/
    ├── tech/
    │   ├── entities/      # 每個 entity 為同名資料夾
    │   │   ├── argocd/
    │   │   │   └── argocd.md
    │   │   ├── typescript/
    │   │   │   └── typescript.md
    │   │   └── dotnet-web-api/
    │   │       ├── dotnet-web-api.md
    │   │       └── (images)
    │   ├── concepts/      # 每個 concept 為同名資料夾
    │   └── summaries/     # 每個 summary 為同名資料夾
    ├── personal/
    └── synthesis/
```

## 資料夾結構規範

### 核心原則

每篇文章 = **同名資料夾**，`.md` 與 `assets/` 資料夾**同層**。圖片統一放入 `assets/`。

```text
Article/
├── Article.md      ← 文章本體
└── assets/         ← 圖片資料夾（與 .md 同層）
    ├── image.png
    └── diagram.svg
```

移動整個 `Article/` 資料夾不會破壞任何圖片的相對路徑。

### 適用範圍

- `raw/` 所有來源文章（含無圖片者，統一建資料夾）
- `wiki/` 所有 entity / concept / summary / synthesis 頁面

### 圖片引用格式

使用標準 Markdown 相對路徑，**不使用** Obsidian wikilink：

```markdown
✓  ![alt text](assets/image.png)
✗  ![[image.png]]
✗  ![](image.png)   （圖片不在 .md 同層，必須加 assets/ 前綴）
```

### 新增檔案 SOP

1. 建立同名資料夾：`mkdir "ArticleName"`
2. 在資料夾內建立 `ArticleName.md`
3. 若有圖片，建立 `ArticleName/assets/` 並放入圖片
4. 圖片引用寫 `![alt](assets/filename.png)`

### 禁止事項

- 文章 `.md` 放在資料夾外（舊格式，已廢棄）
- 圖片直接放在與 `.md` 同層（不進 `assets/`）
- 在 wiki 頁面用 `![[image.png]]` wikilink 引用圖片

不要動 `.obsidian/`（Obsidian vault 設定）。

---

## 命名規範

- 檔名一律 **kebab-case**：`argocd.md`、`gitops-workflow.md`
- 工具/技術概念優先用英文；個人主題（情緒、習慣）可用中文檔名
- entity 頁 = **名詞單數**（`kubernetes.md` 而非 `kubernetes-stuff.md`）
- summary 頁命名：`<source-basename>-summary.md`
- synthesis 頁用描述性短句：`gitops-vs-traditional-cicd.md`

---

## Frontmatter 規範

每個 wiki 頁面開頭必加：

```yaml
---
type: entity | concept | summary | synthesis
domain: tech | personal | cross
tags: [標籤陣列]
sources: [raw/tech/legacy-notes/argocd-tutorial.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

`sources` 列出本頁知識的來源檔（相對 vault root 路徑）。每次更新時 append 新 source。

---

## 內部連結

一律使用 Obsidian wikilinks：

- 簡單連結：`[[argocd]]`
- 別名連結：`[[gitops-workflow|GitOps 流程]]`
- 跨資料夾連結：Obsidian 自動解析，**不要寫完整路徑**

頁面結尾固定加一個 `## 連結` 區塊列出相關頁。

---

## 三大 Operation

### 1. Ingest — 匯入新 source

**觸發詞**：「ingest 這個」「讀這篇」「處理這個」「幫我消化」

步驟：

1. **讀取** raw/ 內的 source（必要時讀引用的圖檔）
2. **討論** 與使用者交流關鍵 takeaway，問清楚他想強調什麼
3. **寫 summary 頁** → `wiki/<domain>/summaries/<basename>-summary.md`
4. **更新或新建 entity / concept 頁**：
   - 該 source 提到的每個重要工具、概念都需要對應頁面
   - 既有頁面 → 補充新資訊，於 `sources:` 加入本 source
   - 新頁面 → 從零建立，含完整 frontmatter
5. **建立 cross-link**：新舊頁面雙向連結
6. **標註矛盾**：若新 source 與既有頁面衝突，於頁中以
   `> ⚠️ 矛盾：[[old-claim]] 主張 X，[[new-source-summary]] 主張 Y` 標註，並等待使用者裁決
7. **更新 `index.md`**：在對應分類加入新頁
8. **追加 `log.md`**：`## [YYYY-MM-DD] ingest | <source title>` + 1-3 行說明

### 2. Query — 提問

當使用者問問題：

1. 先讀 `index.md` 找線索
2. 讀相關 wiki 頁面（**不直接讀 raw**，除非需要查證原文）
3. 合成答案，引用 `[[wiki-page]]`
4. **若答案有保存價值** → 主動建議：「這個分析要不要存成 synthesis 頁？」
   - 同意 → 寫到 `wiki/synthesis/`，更新 index + log

### 3. Lint — 健檢

**觸發詞**：「lint」「健檢 wiki」「檢查」「整理」

檢查項目：

- 矛盾偵測：跨頁面主張衝突
- 過時主張：新 source 已推翻舊主張
- 孤兒頁面：無 inbound link 的頁
- 缺漏概念：被多次提及但無自己的頁面
- 缺漏 cross-link：應該連結卻沒連
- 提出值得追問的問題與可補的 source

輸出為一份 lint 報告，由使用者裁決逐項修復。

---

## index.md 維護規則

- 結構：依 `wiki/` 子分類列出每頁
- 每行格式：`- [[page-name]] — 一行 hook`
- 每次 ingest / synthesis 寫入後**立刻**更新

## log.md 維護規則

- **Append-only**，永不刪改舊條目
- 每筆開頭：`## [YYYY-MM-DD] <action> | <subject>`
- action 類型：`bootstrap`、`ingest`、`query`、`synthesis`、`lint`、`refactor`
- 一筆 1-3 行，記錄 what + why

---

## 教材寫作規範

### 書本式敘述（強制）

**所有教學文件（raw/ 與 wiki/）都必須以「寫一本書」為目標編寫。**

具體要求：

- **有敘事脈絡**：每個概念不只是條列定義，要說明「為什麼需要它」、「它解決了什麼問題」、「它從哪裡演進而來」
- **段落成句**：章節內容用完整的段落描述，而非純粹的 bullet list；bullet list 只用於對比表格、步驟清單、速查
- **有舉例與反例**：每個重要概念都要有「好的做法」與「不好的做法」對比
- **讀者感知**：寫作時預設讀者是「有基礎但第一次遇到這個概念的開發者」，不預設他已經知道答案
- **過渡句**：章節之間要有銜接，不要讓讀者感覺在讀互不相關的片段

**不合格範例（條列式，禁止作為主體）：**

```markdown
## useState
- 用來管理狀態
- 回傳 [value, setter]
- 初始值放在括號內
```

**合格範例（書本式）：**

```markdown
## 第四章：State — 讓畫面活起來

Props 是外部傳進來的，那元件自己需要「記住」什麼東西時怎麼辦？
這就是 State 的用途。State 是元件私有的、可以隨時間改變的資料。
當 state 改變，React 會自動重新渲染元件，產生新的畫面。
```

---

### 先規劃大綱，再寫內容（強制）

在撰寫任何教學文件之前，**必須先完成整體大綱規劃**，再開始逐章寫作。

流程：

1. **理解來源全貌**：先讀完或掌握所有來源材料的結構
2. **規劃章節架構**：確定章節順序、每章核心問題與對應概念
3. **檢視前後依賴**：確保讀者讀到第 N 章時，所需的前置知識已在前幾章建立
4. **確認後動筆**：大綱定案後才逐章撰寫，不邊想邊寫

大綱應回答三個問題：

- 這個主題的「起點」是什麼（讀者的初始狀態）？
- 每一章解決讀者腦中的哪一個問題？
- 讀完全書後，讀者能做到什麼（終點）？

---

## 語言規範

- wiki 內容用 **繁體中文** 為主
- Technical terms 保留英文（namespace、middleware、component、controller）
- 標題、frontmatter key、檔名用英文
- 引用原文（quote block）保留原語言

---

## 互動原則

1. **使用者只做**：找來源、提問、決策方向
2. **LLM 全包**：閱讀、摘要、合成、cross-link、bookkeeping
3. **每次對話開始**：先讀本檔 + `index.md`
4. **每次對話結束**：適度更新 wiki + `log.md`
5. **檔案從不靜默修改**：每次寫入後告訴使用者動了哪些檔
6. **不確定就問**：分類模糊、頁面是否合併、衝突如何裁決 → 一律先問

---
> Source: [zxcbig7/MyAINote](https://github.com/zxcbig7/MyAINote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
