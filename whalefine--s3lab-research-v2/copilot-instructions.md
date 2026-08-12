## thesis-rule

> Thesis writing rules for structure, chapter scope, references, and title consistency


# 論文撰寫規則（僅在你提到要寫論文時啟用）

## 啟用條件
- 只有在使用者明確提到「要寫論文 / 正在寫論文 / 論文章節 / 摘要 / 引言 / related work / 方法 / 實驗 / 結論」等論文撰寫語境時，才套用本規則。
- 若對話內容不是論文撰寫任務，忽略本規則。

## 論文標題（固定）
- 中文標題：`應用於 UAV 視覺追蹤之 16 奈米 Softmax-Free Vision Transformer 硬體加速器`
- 英文標題：`A 16 nm Softmax-Free Vision Transformer Hardware Accelerator for UAV Visual Tracking`

## 標題使用規範
- 涉及封面、摘要首頁、題目欄、投稿資訊時，必須使用上述完整標題，不可自行改寫。
- 需要縮寫時，僅可在內文首次出現後以「本文」或「本研究」指代，不可更動正式題名。
- 中英文標題需一一對應，不可混用其他版本。

## 總原則
- 清楚區分「別人的工作」與「自己的工作」：前者放背景與文獻脈絡，後者放方法、實作與結果。
- 用詞一致：以「章／節」「圖」「式」「reference」為主要術語。
- 章節安排遵守常見骨架：先背景與相關工作，再方法，最後結果與結論。

## 常見整體骨架
- 建議順序：摘要 -> Abstract -> Content -> List of Figures -> List of Tables -> Chapter 1 Introduction -> Chapter 2 Software Implementation -> Chapter 3 Hardware Implementation -> Chapter 4 Conclusion and Future Works -> Reference。
- 目錄頁標題固定為「Content」，不用「Table of Contents」或「目錄」。
- Chapter 1：鋪背景、補基礎、整理 related work、導向 motivation。
- Chapter 2：完整解釋自己的方法、流程、模型、架構或系統設計。
- Chapter 3：呈現自己的實驗、硬體實作、資源消耗、效能與比較表。
- Chapter 4：收斂成結論與未來工作。

## 段落推進公式
- 通用段落：定義主題 -> 說明重要性 -> 引用代表方法 -> 分析限制 -> 導向本研究。
- 文獻回顧段落：方法做了什麼 -> 優點是什麼 -> 限制是什麼 -> 為何仍不足以解決本文問題。
- 自己方法段落：模組功能 -> 設計原因 -> 關鍵細節 -> 這個設計帶來的效果。
- 結果段落：先交代設定 -> 再報告數字 -> 再解釋原因 -> 最後點出與目標的關聯。

## Chapter 1（緒論／背景與動機）
- 將他人研究、相關背景與被引用文獻脈絡寫在本章，先把脈絡說清楚。
- 撰寫 introduction 與 motivation，並列出三點最重要的 contribution（可驗證且可對應後文章節）。
- 對關鍵文獻需同時說明優點與限制，並交代與本研究關聯。
- 文獻核對卷期頁、DOI、版本與重複引用，引用樣式以指導老師規定為準。
- 建議順序：應用背景 -> 基礎技術 A -> 基礎技術 B -> related work/特定議題 -> motivation -> thesis organization。
- Chapter 1 不要過早展開自己的實作細節；細節放在後續章節。
- related work 不可只列文獻名稱，必須交代方法、優點、限制與本文關聯。

### Chapter 1 節次結構（從 5 篇實際論文歸納）
- Chapter 1 通常包含 7～9 個小節。
- **第一節**固定為應用情境介紹（如「Introduction to UAV and Their Applications」）。
- **倒數第二節**固定為 Motivation（明確列出前人方法不足之處與本研究貢獻）。
- **最後一節**固定為 Thesis Organization（逐章說明各章內容，一段式，每章一句）。
- Thesis Organization 是每篇論文的標準結尾，不可省略。
- Related work 通常以獨立小節呈現，或附在對應技術介紹節內（如「1.3 Introduction to AIDER dataset and its related work」），不另開 Chapter 2 放 related work。

## Motivation 與 Contribution 模板
- Motivation：前人方法有效，但仍有 A/B/C 限制 -> 實際應用需要 X/Y/Z -> 因此本文提出...。
- Contribution 固定整理成 3 點，且每點都需能對應到後文章節與實驗結果。
- 三點常見分工：方法/模型/演算法新意、系統/架構/硬體實作新意、效能/部署/應用價值新意。

## Chapter 2（Software Implementation：只做自己的）
- 只寫自己的研究如何實作（方法、架構、流程、細節）。
- 基礎通用背景可精簡，與本研究直接相關細節可詳述。
- 建議順序：overall architecture/workflow -> 模組拆解 -> 設計理由 -> 中間分析或消融結果。
- 先給整體架構圖或流程圖，再逐節拆解。
- 每個模組需交代：做什麼、為什麼這樣設計、怎麼做、帶來什麼好處。

### Chapter 2 節次結構（從 5 篇實際論文歸納）
- **2.1 Architecture Overview**：固定為第一節，給出整體架構圖或流程圖，不可省略。
- 中間各節拆解各模組（Dataset Preprocessing、模型實作、量化方法等）。
- **最後一節**固定為 Software Result（呈現軟體實作的準確率、混淆矩陣等），不可放在中間。
- Confusion matrix table 必須在 Chapter 2 的 Software Result 中呈現。

## Chapter 3（Hardware Implementation）
- 只呈現自己的硬體實驗與數據，不寫成他人方法調查章。
- 比較表需以可查證來源為依據（論文、官方實作、作者數字），不可臆造數據。
- 建議順序：Fixed-point/量化處理 -> 硬體加速器架構 -> Power Gating -> 硬體實作分析與比較 -> [Summary]。
- 表後說明不可只寫「本文最好」，需補充他法強項、限制與本文適用條件。
- 多組表格時，每張表優先只回答一個問題（如準確率、資源、延遲、功耗或面積）。

### Chapter 3 節次結構（從 5 篇實際論文歸納）
- **倒數第二節（或倒數第一節）**固定為「Hardware Implementation Analysis and Comparison」，內含：
  - 硬體實作結果總表（頻率、功耗、面積、準確率）
  - **「Comparison with prior methods」**比較表（此表名稱在 5 篇中完全一致，必須使用）
- **最後一節**（較新論文，111 年起）固定為「Summary」，一段式總結本章重點。
- 110 年論文尚未有 Summary 節，111 年後均有，建議加入。

## Chapter 4（結論）
- 結論需總結研究目標、方法、代表性結果與實際價值。
- Future Work 應由目前限制延伸，不可過度空泛。
- 常見拆法：4.1 Conclusion、4.2 Future Work(s)。

## 摘要寫法模板（從 5 篇實際論文歸納）
- 摘要固定四段式，每段對應一個主題：
  - 第一段：應用情境背景（邊緣運算/UAV/問題陳述）
  - 第二段：本文提出的方法（本論文/本研究 使用 X 方法，並說明設計動機）
  - 第三段：硬體實作（在哪個製程實作、使用什麼技術如 Power Gating）
  - 第四段：關鍵量化結果（頻率 MHz、功耗 mW、準確率 %，三者必須出現）
- **關鍵字格式**：`**關鍵字：** 詞1、詞2、詞3、詞4、詞5`（黑體，頓號分隔，5～6個）
- Abstract 為摘要的逐段英文對應，段落數與重點必須一致，不可中英文重點分離。
- **Keywords 格式**：`**Keywords**: term1, term2, term3`（黑體，逗號分隔）

## Reference 與交付
- Reference 清單僅列正文實際引用文獻，避免重複條目。
- 若需交付引用全文 PDF，依指導老師要求整理與上傳。
- 檔名可用引用編號前綴（如 [60]、[61]）對齊正文編號。
- 引用學位論文時，格式須符合老師或系上範本；不建議以學長姐論文作為主要比較對象。

## 標號、圖片與公式（從 5 篇實際論文歸納）
- 小節與圖號固定使用二層（如 1.1、1.2），嚴禁三層（如 1.1.1）。
- **圖號格式**：`Figure X.Y Description [ref].`（例：`Figure 1.3 Flowchart of fire detection algorithm [3].`）
- **表號格式**：`Table X.Y Description.`（例：`Table 3.10 Comparison with prior methods.`）
- 圖號與表號均按章節重新計數（Figure 1.x 在 Ch1，Figure 2.x 在 Ch2，依此類推）。
- 使用他人圖片時需在圖說末標 `[ref]`；預設優先自製圖。
- 數學公式需用可編輯方程式（LaTeX 或 Word 方程式），不可用截圖代替。
- List of Figures 與 List of Tables 為獨立頁面，緊接在 Content 之後，格式與圖號一致。

## 與 Agent 協作建議流程
1. 先確認要修改哪一章，再套用該章規則（Chapter 1 他人脈絡、Chapter 2 自己方法、Chapter 3 自己結果）。
2. 規劃整篇時，優先套用共同骨架：應用背景 -> 基礎技術 -> related work -> motivation -> 自己方法 -> 自己結果 -> conclusion。
3. 撰寫單節時，先判斷其屬性（背景、文獻、方法、結果、結論），再套用對應段落公式。
4. 產出 Motivation 或 Contribution 時，優先壓成 3 點且避免空泛詞。
5. 產出或修改比較表前，先確認文獻來源、連結與欄位定義，僅填入已核對數字。
6. 修改引用或 reference 時，提醒核對 DOI、卷期頁與重複條目。

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
