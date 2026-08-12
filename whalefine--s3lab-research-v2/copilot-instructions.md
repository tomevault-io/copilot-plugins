## research-narrative-thesis

> 本研究整體歷程、改動方向、軟硬體流程與論文敘事主軸（SGLATrack → softmax-free CARE → Q8.8 → Python golden → ROM → Verilog → APR）


# 研究敘事與技術路線（論文／協作參考）

本檔記錄本研究**從模型改動到晶片收尾**的主線、設計取捨與產物意義，供撰寫論文方法與系統章節、以及與 Agent 協作時對齊語境使用。若使用者未提到「研究流程／論文敘事／本專案階段」，可不強制套用全文，但**討論 SGLATrack、CARE、fixed-point、Verilog 或 APR 時**應優先對照本檔以免敘事矛盾。

正式論文標題與章節語氣仍以 `.cursor/rules/thesis_rule.mdc` 為準。

---

## 研究目標（一句話）

以 **Softmax-Free Vision Transformer** 為核心的視覺追蹤模型，經 **定點（Q8.8）化** 與 **Python 硬體對齊檢查**，完成 **RTL → APR**，並以 **DRC／LVS clean** 作為研究收尾條件。

---

## 階段一：在 SGLATrack 上探索 Softmax-Free Attention

### 1-A 問題起點

標準 softmax attention 公式為 `softmax(QKᵀ/√d)V`，其中 `exp()` 與 `row-wise division` 在硬體中代價高昂（需要 LUT 或 CORDIC 實作 exp，面積與延遲均大），是 ViT 加速器的主要瓶頸。本研究目標為在保住 SGLATrack 追蹤精度的前提下，找到一個可直接對應 RTL 的 attention 替代方案。

---

### 1-B 嘗試過的四條路線（依序）

#### ① Squaremax（reference [61]）
- **機制**：`Attn = normalize(ReLU(QKᵀ)²) V`，以 ReLU² 取代 exp，normalization 改為 shift + 小 decoder（不需要 divider）。
- **複雜度**：仍為 **O(N²)**（必須計算完整 N×N attention 矩陣）。
- **硬體友善度**：高——無 exp，無 LUT。
- **為何未採用**：O(N²) 的 memory bandwidth 成本在 token 數多時仍難以接受；無 per-head gating，attention 表達力較弱，追蹤指標有下降。

#### ② SimA（Simple Softmax-free Attention，reference/sima）
- **機制**：對 Q、K 做 **L1 normalization**，再做 `q @ (kᵀ @ v)`（linear attention，**O(N)**）。
- **複雜度**：O(N)——先算 kᵀv（d×d 矩陣），再乘 q，token 數不影響主要計算量。
- **硬體友善度**：中——L1 norm 需要絕對值累加，相較可接受，但 normalization 邊界在 fixed-point 下需額外處理。
- **為何未採用**：移植到 SGLATrack 後追蹤性能下降明顯；L1 norm 的語意與 SGLATrack 訓練假設不吻合。

#### ③ MALA（Magnitude-Aware Linear Attention，reference/MALA）
- **機制**：`q = ELU(q)+1`，`k = ELU(k)+1`，線性注意力 + additive correction + **RoPE** 位置編碼。
- **複雜度**：O(N)。
- **硬體友善度**：低——ELU 內含 exp（與 softmax 同樣問題）；RoPE 需要 sin/cos 計算，硬體成本大。
- **為何未採用**：兩個難實作的非線性（ELU+RoPE）同時存在，等同放棄硬體友善的目標。

#### ④ CARE-Transformer（原版，reference/CARE-Transformer）
- **機制**：`q = ELU(q)+1`，`k = ELU(k)+1`，線性注意力 `kv = kᵀv/N`，`x = q @ kv * z`，其中 `z = 1/(q @ k_mean ᵀ)`。
- **複雜度**：**O(N)**——先算 kv（d×d），再乘 q，具有 per-head gating（z）保持 attention 銳利度。
- **硬體友善度**：中——架構設計優秀，但 kernel function **ELU+1 仍含 exp**。
- **為何未採用原版**：ELU 的 exp 是唯一硬體障礙；若能替換 kernel function，CARE 架構本身非常適合 RTL 實作。

---

### 1-C 最終選型：CARE + ReLU6 + Q8.8

- **改動邏輯**：在 CARE 架構上，將 `ELU()+1` kernel function 替換為 **ReLU6**。
  - **ReLU → ReLU6 的理由**：純 ReLU 輸出無上界，在 Q8.8 定點格式下可能 overflow；ReLU6 將輸出截在 [0, 6]，與 Q8.8 的整數部分（8 bit 表示最大 127）配合更佳，數值穩定。
  - **結果**：O(N) 線性注意力、無 exp、輸出有界、per-head gating 保留 → 四個硬體友善條件全部滿足。
- **模型檔**：`python/lib/models/sglatrack/vit_CARE_relu6_fixed.py`（ViT-B/16，depth=12，embed=768，Q8.8 截斷插入每一個中間結果）。
- **論文可寫重點**：
  - 為何 softmax-free（exp 硬體成本）
  - 四種方案的比較表（O(N²) vs O(N)、kernel function、硬體成本、追蹤精度）
  - 為何最終選 CARE + ReLU6（O(N) + 無 exp + bounded + gating）
  - ReLU vs ReLU6 的差異及對 Q8.8 的影響
  - 數字比較引用 benchmark-results.mdc

---

## 階段二：固定點插入主架構（FP32 → Q8.8）

- **工作內容**：在已定案的 CARE 類架構上，將運算流程改為 **fixed-point** 語意一致之模擬／訓練後流程。
- **量化格式**：固定為 **Q8.8**（與專案中 fixed-point／硬體友善設計 skill 對齊）。
- **論文可寫重點**：哪一層／哪條路徑插入定點、截斷與尺度約定、與 FP32 的誤差傳播界線。

---

## 階段三：Python Check（硬體語意一致性與 Golden）

- **目的**：在撰寫／驗證 Verilog 前，用 Python **模擬「硬體會怎麼算」**，確認數值流程正確。
- **輸出產物**：
  - **Weight**：以 **binary 格式** 輸出並以 **txt** 儲存；**權重來源使用 NPY 而非 PTH**。
    - **理由**：後續撰寫 Verilog 時需知道 **在各段落／模組邊界要餵入哪一段 weight**；NPY 便於與記憶體映射、位址區段、ROM 初值對齊，勝過直接依賴 PyTorch `.pth` 的抽象封裝。
  - **Activation**：同樣以適合對照的格式輸出（binary／txt），作為 **golden reference**。
    - **理由**：撰寫 Verilog 時 **逐步對答案**，確認每一級 pipeline／每個 function 區塊輸出與 Python golden 一致。
- **論文可寫重點**：軟硬體協同驗證方法、golden 生成流程、與 RTL 對位的介面（word width、endianness、區塊邊界）。

---

## 階段四：ROM 先行與 Verilog 整合現況

- **順序**：需 **先把權重寫入 ROM（或等價之常數表／初始化映像）**，才能進行 **完整頂層／系統級** Verilog 整合與時序完整的驗證。
- **限制**：在 ROM／完整記憶體映像尚未就緒前，**完整 testbench／端到端仿真** 不易撰寫或意義不完整。
- **現況**：**各功能模組（function-level）的 Verilog 已分別完成**；待 ROM 與介面對齊後，再接成可仿真的完整 datapath／control。
- **論文可寫重點**：硬體模組化策略、memory hierarchy、為何採 ROM 先行與驗證分層。

---

## 階段五：APR 與結案條件

- **後續流程**：RTL 與驗證收斂後進行 **APR（自動佈局繞線）**。
- **研究收尾條件**：**DRC、LVS 皆 clean**（必要時於論文註明製程／PDK／signoff 工具鏈版本與約束）。
- **論文可寫重點**：實體設計結果、面積／時脈／功耗若可得則放入結果章。

---

## 給 Agent 的協作提示

1. **模型／量化**：修改 PyTorch 時優先對齊 **CARE + Q8.8** 與既有 skill（如 `sglatrack-fixedpoint-q8-8`、`sglatrack-qat`）的目錄與慣例。
2. **產物格式**：輸出給硬體的 weight／activation，維持 **binary + txt** 與 **NPY 權重管線** 的敘事一致。
3. **Verilog**：區分 **已完成之 function block** 與 **待 ROM／頂層串接後才完成的 TB**；避免假設端到端仿真已可跑。
4. **論文引用**：方法與貢獻敘述應可對應到上述五階段；數字比較以 `benchmark-results.mdc` 與實際實驗為準。

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
