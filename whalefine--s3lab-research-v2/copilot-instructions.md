## numpy-trunk-to-verilog

> 以 run_backbone_numpy_shared_trunk.py 為 golden，將 numpy trunk 對齊轉 Verilog；backbone 對拍 verilog_backbone2/；head 對拍以 verilog_head2/ 為準（verilog_head/ 僅舊版參考）


# Numpy Trunk → Verilog 對齊規則

本規則與 `verilog_rule.mdc`（語法／可合成底線）並存；若有衝突，**可合成與對拍語意**以本檔與 `verilog_rule.mdc` 較嚴者為準。

## 1. Golden 與產出位置

- **數學與流程依據**：`python/tracking/run_backbone_numpy_shared_trunk.py`（以下稱 **numpy trunk**）。
- **Verilog 實作目錄**：`python/lib/models/verilog/`（僅在此目錄新增／修改 RTL；ROM/RAM 若已在外部生成，介面以現有 top / testbench 為準）。
- **Verilog 目錄分工**（多條路徑並存，勿混淆）：
  - **`python/lib/models/verilog_backbone2/`** — **目前 backbone 對拍基準**（`TEST_backbone.v` 等）；與 numpy trunk backbone 階段 golden 對拍。
  - **`python/lib/models/verilog_head2/`** — **目前 head 對拍基準**：**不跑 backbone RTL**；以 numpy trunk 產出的**準確 activation**（例如 `backbone_after_norm_backbone_out_bi.txt`）串流進 `head_top`，驗證 conv → tail → `cal_bbox`；TB 為 `TEST_head.v`（編譯 `verilog_head2/*.v` + `Sram_tok1` 等）。
  - **`python/lib/models/verilog/`** — **端到端整鏈路**（`TEST.v`，暫不動）：`sglatrack_top` 含 backbone + head。
  - **`python/lib/models/verilog_head/`** — **舊版 head-only**（`Sram_sh1_lo`/`hi`、`mac_dv` 等）；**僅參考或比對舊實作，預設不作為對拍修改目標**。
  - 修改 RTL 時須先確認目標目錄：backbone → `verilog_backbone2/`；**head → `verilog_head2/`**（除非使用者明確指定 `verilog_head/` 或 `verilog/`）；全鏈路 → `verilog/`。
- **Activation 對答案檔**（Q8.8 二進位文字，一行一元素）：  
  `python/output/golden/vit_care_relu6_numpy_trunk_dim32_out/Activation/*.txt`  
  對應檔名多為 `*_bi.txt`（與腳本 `write_bi` 輸出一致）。RTL 除錯時以**此目錄**為 golden，不以「猜 golden」為準。
- **階段對拍義務**：詳見 **§21**（numpy trunk 寫出的 activation 須與 Verilog 同階段輸出對答案）。

## 2. 不硬體化、不當作 RTL 行為依據的程式碼

- numpy trunk 內僅負責 **輸出 activation / weight 的 `*_bi.txt`** 的路徑（例如 `write_bi`、`save_npy`、`write_wbi`、目錄 `_bi_act_dir` / `_bi_wgt_dir`）**不實作進晶片**，也不當作 datapath 規格；它們的用途是 **給 Verilog 模擬對答案** 與產生 ROM 初值檔。
- **主架構外**的輔助流程不強制硬體化：`argparse`、`main`、讀檔、`json`、`print`、`mkdir`、`cv2`（若有）等。

## 3. 可合成性

- 所有要進晶片（或 FPGA 正式 image）的 RTL **必須可合成**，並遵守 `verilog_rule.mdc`（例如 Verilog-2001、禁止用於「硬體邏輯」的 `initial` / `#delay` 等，依該檔全文為準）。
- 若為 **除錯專用**（例如僅模擬用的 `$display`、`ifdef` 區塊、不可綜合的暫時波形 dump），必須：
  - 以註解或 `ifdef` 標題**明寫「不可合成／僅 simulation」**，且
  - **註明為解決何種現象而插入**（例如：對拍哪一節點 golden、追查 argmax 錯位、SRAM 寫入下溢、sigmoid 全飽和等；見 **§10**），並
  - 預設關閉或易於關閉，避免誤綜。

## 4. 修改必須有依據

- 修改 Verilog **不可憑猜**：需註明或可追溯至 **numpy trunk 對應函式與行數**、**golden 檔名與 flatten 順序**、或**已約定的定點規格**（例如 Q8.8、`fp`/`linear` 與 RTL 的對應說明）。
- 若規格不明，應先補文件或先在 numpy／binary check 釐清，再改 RTL。

## 5. 語義不對齊時的優先順序（Python 先可硬體化）

- 若發現 numpy trunk 某段**不易硬體化**或與已定 RTL **語義不一致**，優先 **修改 `run_backbone_numpy_shared_trunk.py`**，使演算法改為：
  - 固定 shape、固定迴圈深度、
  - 明確的定點／截斷／飽和步驟（與未來 RTL 可一一對應），
  - 再更新 golden 與 RTL。
- 避免為了遷就錯的 Python 語意而去寫「無法對拍」的 RTL。

## 6. Verilog 改動幅度

- 以 **最小 diff** 為原則：盡量保留原有模組結構、介面與命名；**只改必要區段**解決對拍或合成問題。
- 非經使用者同意，不做與當前 bug 無關的大重構。

## 7. 訊號 ↔ Golden 檔名註解（強制）

- 在 `python/lib/models/verilog/` 內，凡會對應到 numpy trunk 中間輸出、且可用 Activation golden 對拍的 **模組輸出／暫存陣列／串流**，應在適當位置（模組頭或該訊號旁）加註解，格式建議：

  `// Golden: vit_care_relu6_numpy_trunk_dim32_out/Activation/<檔名>_bi.txt`

  若一訊號對應多檔或子路徑，寫清楚檔名與 tensor 對應關係（例如 token 順序、flatten 為 row-major 等，若 golden 有文件則引用）。

- 若某節點 numpy 有 `save_npy("xxx.npy")` 但 golden 實際為 `xxx_bi.txt`，註解以 **實際存在的 `Activation/` 檔名** 為準。

## 8. 硬體化範圍（排除清單）

- **不必**硬體化：僅供寫檔、寫 log、路徑組合、與 `np.load`/CLI 相關的程式碼（見第 2 節）。
- **必須**能對應到硬體 datapath 的：依 numpy trunk **主流程**會執行到的 `block_forward`、`attention_forward`、`layer_norm`、`linear`、`head_shared_trunk`、`conv2d`、以及（若納入範圍）`cal_bbox` 與 tracker 相關運算等；實作時以「可合成 + 可對拍」為取捨邊界。

## 9. 主架構優先

- 判斷「要不要硬體化」時，以 numpy trunk **正常 main 路徑**會跑到的運算為準；冷門分支、僅在缺少 golden 時才走的 fallback（例如自行算 adaptive）可標為軟體或第二階段，除非使用者明確要求納入 RTL。

## 10. 除錯環境（模擬不在本 repo）

- 假設 **Verilog simulation 在外部環境**執行；除錯可依賴：
  - 使用者**截圖**（波形／訊號），或
  - 在 RTL 暫加 **`$display`**（須標註僅 simulation、見 **§3**），由使用者截圖或貼 log 回傳數值。
- 凡為除錯而加入的 **`ifdef`／`$display` 區塊**，除 **§3** 外，**必須**在區塊上方或模組頭註解寫清：**為了解決／驗證什麼**（症狀或對拍目標）、**與哪個 golden 檔或哪一層節點相關**；避免僅寫「debug」而後續無法判斷是否可刪、與何問題綁定。細節見 **§10.1**（**註解義務**與 **`ifdef` 分區**兩條）。
- 若使用者**無法使用波形圖**（無 FSDB/VPD 或無法開 viewer），除錯方式以 **§10.1** 為準，不應假設能依波形縮小問題。
- Agent 修改 RTL 時應避免依賴「本機跑完 sim」作為唯一驗證，改以 **golden 檔 + 程式依據** 推進。
- 使用者實機之 **VCS 編譯／`./simv`** 流程見 **§22**。

### 10.1 僅能以 `$display` 除錯（無波形圖）

當環境**無法使用波形 viewer**、僅能靠仿真 log 中的 **`$display`／`$strobe`／`$fdisplay`** 觀察訊號時，建議遵守下列做法，以便與 **§16** 前向對拍且不致洗版：

- **取樣時機**：只在有意義的拍列印（例如 `mac_bp && mac_dv` 寫 SRAM 當拍、`done` 拉高當拍、`state` 切換後第一拍）；避免每個 `posedge` 盲目列印。
- **NBA 後數值**：要看 nonblocking 更新後的結果，優先用 **`$strobe`**，或在下一拍再讀已鎖存的 `reg`。
- **SRAM `CLK(~clk)`**：讀 `*_q` 時可改在 **`negedge clk`** 列印（與 macro 讀取邊沿一致），避免 posedge 上位址已變而 `Q` 仍舊的錯覺（與 `head_top` 內 `DUMP_HEAD_BBOX_SRAM_NEGEDGE` 類思路一致）。
- **格式**：定點以 **`%h` 為主**，並在註解或同一行標明 **Q8.8／Q0.8**；負值須明示 **signed 二補數** 解讀，勿將 `0xff..` 誤當無號大正數。
- **列印語言（強制）**：`$display`／`$strobe`／`$fdisplay`／`$fwrite`／`$monitor` 等**仿真 log 內可見字串必須為英文（ASCII）**；使用者執行 `.v` 的環境**不支援中文**，禁止在 format string 或訊息內使用中文（含簡繁）。說明性文字請寫在 `//` 註解，或 Agent 回覆給使用者；RTL／TB 內僅輸出英文標籤（例如 `PASS`、`FAIL`、`bbox_mismatch`、`golden=`）。
- **`ifdef` 分區**：除錯列印一律以 **`ifdef` 編譯開關**封裝（預設關閉），見 **§3**；可按模組拆成多個 macro（例如 `DUMP_HEAD_SIZE_SAT`），按需開啟。**每個 macro 對應的區塊**須有註解說明：**欲診斷的問題**（例如「追查 tail_size 寫入前 `mac_cl` 過大導致 sigmoid 全飽和」）、**建議搭配編譯選項**、以及**若問題已解決是否應移除或改為預設關閉**。
- **註解義務（`ifdef`／`$display` 區塊）**：凡為除錯插入的列印，**不可**只留開關名稱；至少應在 **`ifdef` 上一行區塊註解**或**模組頭對應小節**寫明：
  - **要解決／驗證的現象**（例如 bbox 與 golden 差、`sc_q` 全為下限、與某 `Activation/*_bi.txt` 第一個分歧層級等）；
  - **列印欄位與取樣條件**（例如僅 `S_SIZE && mac_bp && mac_dv`）；
  - **與 numpy／golden 的對照方式**（檔名、flatten、或手動比對步驟）。
  - **Agent**：新增或修改此類除錯區塊時，應一併補齊上列三點，便於他人僅靠 log 與原始碼追溯目的。
- **減量**：每 N 格、單一錨點 `(oh,ow)`、或僅 golden **argmax** 對應 index 再列印，避免 log 過大無法 grep。
- **內部訊號**：可暫用**階層路徑**（僅模擬）或在 RTL 加 **`output`／`reg dbg_*`** 供 TB 列印；若禁止層次引用則在子模組內加 `ifdef` 列印。
- **落盤**：必要時 **`$fdisplay` 寫入檔案**（例如 `dbg_head.log`），便於與 golden 文字檔 **diff** 或小腳本比對。
- **對拍順序**：仍遵循 **§16**；無波形時更要固定「**第一個不一致節點**」的列印條件，便於往返驗證。

Agent 與使用者溝通時，若得知**僅能 display**，應優先建議 **本節取樣與格式**，而非假設能開波形。

## 11. 最終目標

- 端到端目標：**正確的 backbone + head +（若範圍內）tracker** 輸出，且與 numpy trunk 在已定義節點上 **可對拍**（優先與 `vit_care_relu6_numpy_trunk_dim32_out/Activation` 一致）；節點與階段定義見 **§21**。
- 邊界未涵蓋的階段（例如僅 CPU 的讀圖、manifest I/O）應在註解或規格中**明確標出**，避免與「晶片內責任」混淆。

## 12. Weight Golden、定點與 ROM（與 Activation 並列）

- **RTL 讀取時序／暫存對齊**：撰寫或修改 `*.v` 時见 `.cursor/rules/verilog_rule.mdc`——**權重 ROM `CLK(~clk)`、`stream`/`stream_r` 列預取、bias 鎖定與反模式（§7）**；通用讀取契約（**「記憶體／ROM 讀取」**）。**本節**負責 golden 路徑與 ROM 映像對照，不列舉具體時序實作細節。
- **ROM macro 檔名 ↔ `Weight/*_bi.txt` 分片對照**（backbone 0~6、box head）：見 **§12.1**。
- **Verilog 側讀到的權重**並非在模擬／晶片內直接讀原始 **`.npy`**，而是讀 **已寫入 ROM** 的內容；該 ROM 的來源與 numpy trunk 一致，資料鏈為：
  1. `run_backbone_numpy_shared_trunk.py` 透過 `load_w()` 等路徑自 **`--weight-dir`** 讀取 **float32 的 `.npy` 權重**；
  2. 經腳本內 **`write_wbi` → `write_bi`** 做 **fixed-point（與專案約定一致，例如 Q8.8）** 後，輸出為 **`Weight/` 目錄下的 `*_bi.txt`**（一行一元素之二進位字串，與 Activation 產出格式同類）；
  3. 上述 **txt／定點權重檔** 已由流程 **匯入／合成 ROM**（在 repo 外或既有 ROM 生成流程完成）；**RTL 僅以 ROM 位址讀出與 golden `Weight/*_bi.txt` 對齊的資料**。
- 權重對拍／驗證 ROM 內容時，仍以 golden 為準：  
  `python/output/golden/vit_care_relu6_numpy_trunk_dim32_out/Weight/*_bi.txt`（與 numpy trunk `write_wbi` 一致）。
- 若 RTL 讀 ROM 的位址或展平順序需對應某權重檔，註解格式建議：

  `// Golden-Weight: vit_care_relu6_numpy_trunk_dim32_out/Weight/<檔名>_bi.txt`

### 12.1 `memory/` ROM ↔ `Weight/*_bi.txt`（對拍與驗 ROM 用）

以下 **ROM 檔名** 相對於編譯時之 `memory/` 目錄（見 **§22**）；**golden** 相對於  
`python/output/golden/vit_care_relu6_numpy_trunk_dim32_out/Weight/`。括號內為 ROM 內元素筆數／區塊劃分說明（與展平後總深度一致；實際位寬仍以 Q8.8／`write_wbi` 為準）。

#### backbone（blocks 0~6）

- `rom_backbone_blocks_0_6_attn_proj_bias.v` **(7×32)** 對應（依 block 0→6 串接）：
  - `backbone_blocks_0_attn_proj_bias_bi.txt` … `backbone_blocks_6_attn_proj_bias_bi.txt`
- `rom_backbone_blocks_0_6_attn_proj_weight.v` **(7×1024)** 對應：
  - `backbone_blocks_0_attn_proj_weight_bi.txt` … `backbone_blocks_6_attn_proj_weight_bi.txt`
- `rom_backbone_blocks_0_6_attn_qkv_bias.v` **(7×96)** 對應：
  - `backbone_blocks_0_attn_qkv_bias_bi.txt` … `backbone_blocks_6_attn_qkv_bias_bi.txt`
- `rom_backbone_blocks_0_3_attn_qkv_weight.v` **(4×3072)** 對應：
  - `backbone_blocks_0_attn_qkv_weight_bi.txt` … `backbone_blocks_3_attn_qkv_weight_bi.txt`
- `rom_backbone_blocks_4_6_attn_qkv_weight.v` **(3×3072)** 對應：
  - `backbone_blocks_4_attn_qkv_weight_bi.txt`、`backbone_blocks_5_attn_qkv_weight_bi.txt`、`backbone_blocks_6_attn_qkv_weight_bi.txt`
- `rom_backbone_blocks_0_6_mlp_fc1_bias.v` **(7×128)** 對應：
  - `backbone_blocks_0_mlp_fc1_bias_bi.txt` … `backbone_blocks_6_mlp_fc1_bias_bi.txt`
- `rom_backbone_blocks_0_3_mlp_fc1_weight.v` **(4×4096)** 對應：
  - `backbone_blocks_0_mlp_fc1_weight_bi.txt` … `backbone_blocks_3_mlp_fc1_weight_bi.txt`
- `rom_backbone_blocks_4_6_mlp_fc1_weight.v` **(3×4096)** 對應：
  - `backbone_blocks_4_mlp_fc1_weight_bi.txt`、`backbone_blocks_5_mlp_fc1_weight_bi.txt`、`backbone_blocks_6_mlp_fc1_weight_bi.txt`
- `rom_backbone_blocks_0_6_mlp_fc2_bias.v` **(7×32)** 對應：
  - `backbone_blocks_0_mlp_fc2_bias_bi.txt` … `backbone_blocks_6_mlp_fc2_bias_bi.txt`
- `rom_backbone_blocks_0_3_mlp_fc2_weight.v` **(4×4096)** 對應：
  - `backbone_blocks_0_mlp_fc2_weight_bi.txt` … `backbone_blocks_3_mlp_fc2_weight_bi.txt`
- `rom_backbone_blocks_4_6_mlp_fc2_weight.v` **(3×4096)** 對應：
  - `backbone_blocks_4_mlp_fc2_weight_bi.txt`、`backbone_blocks_5_mlp_fc2_weight_bi.txt`、`backbone_blocks_6_mlp_fc2_weight_bi.txt`
- `rom_backbone_blocks_0_6_norm1_bias.v` **(7×32)** 對應：
  - `backbone_blocks_0_norm1_bias_bi.txt` … `backbone_blocks_6_norm1_bias_bi.txt`
- `rom_backbone_blocks_0_6_norm1_weight.v` **(7×32)** 對應：
  - `backbone_blocks_0_norm1_weight_bi.txt` … `backbone_blocks_6_norm1_weight_bi.txt`
- `rom_backbone_blocks_0_6_norm2_bias.v` **(7×32)** 對應：
  - `backbone_blocks_0_norm2_bias_bi.txt` … `backbone_blocks_6_norm2_bias_bi.txt`
- `rom_backbone_blocks_0_6_norm2_weight.v` **(7×32)** 對應：
  - `backbone_blocks_0_norm2_weight_bi.txt` … `backbone_blocks_6_norm2_weight_bi.txt`
- `rom_backbone_norm_bias.v` **(1×32)** 對應：`backbone_norm_bias_bi.txt`
- `rom_backbone_norm_weight.v` **(1×32)** 對應：`backbone_norm_weight_bi.txt`

#### box head（shared conv / tail）

- `rom_box_head_shared_conv1_2_folded_bias.v` **(96+48)** 對應（依序串接）：
  - `box_head_shared_conv1_folded_bias_bi.txt`（96）
  - `box_head_shared_conv2_folded_bias_bi.txt`（48）
- `rom_box_head_shared_conv1_folded_weight1.v` **(位址 0～16383)** 對應：`box_head_shared_conv1_folded_weight1_bi.txt`
- `rom_box_head_shared_conv1_folded_weight2.v` **(位址 16384～27647)** 對應：`box_head_shared_conv1_folded_weight2_bi.txt`
- `rom_box_head_shared_conv2_folded_weight1.v` **(位址 0～16383)** 對應：`box_head_shared_conv2_folded_weight1_bi.txt`
- `rom_box_head_shared_conv2_folded_weight2.v` **(位址 16384～32767)** 對應：`box_head_shared_conv2_folded_weight2_bi.txt`
- `rom_box_head_shared_conv2_folded_weight3.v` **(位址 32768～41471)** 對應：`box_head_shared_conv2_folded_weight3_bi.txt`
- `rom_box_head_tail_ctr_offset_size_bias.v` **(1+2+2 元素)** 對應（依序串接）：
  - `box_head_tail_ctr_bias_bi.txt`（1）
  - `box_head_tail_offset_bias_bi.txt`（2）
  - `box_head_tail_size_bias_bi.txt`（2）
- `rom_box_head_tail_ctr_offset_size_weight.v` **(48+96+96 元素)** 對應（依序串接）：
  - `box_head_tail_ctr_weight_bi.txt`（48）
  - `box_head_tail_offset_weight_bi.txt`（96）
  - `box_head_tail_size_weight_bi.txt`（96）

## 13. 常數與形狀鎖定（Python ↔ Verilog）

- numpy trunk 檔頭常數（如 `EMBED_DIM`、`NUM_HEADS`、`N_TOKENS`、`START_LAYER`、`FEAT_SZ`）變更時，**必須**同步檢查 `python/lib/models/verilog` 內對應 `parameter`／`localparam`，並重新產生 golden／ROM；不可只改一邊。此類變更通常**連動 ROM**，須依 **§20** 向使用者說明。
- 若 golden 根目錄名稱或 `output-dir` 慣例變更，應**更新本檔第 1、12 節（含 §12.1 ROM 對照）路徑**，或改在 repo 內維護**單一路徑 manifest / README**，避免 RTL 註解與實際檔案脫鉤。

## 14. Golden 檔名與 numpy 節點對照

- 區塊：`backbone_blocks_<block_idx>_...`（對應 `block_forward` / `attention_forward` 的 `pf`）。
- Head：`box_head_*`（對應 `head_shared_trunk`、`cal_bbox`）。
- Tracker：`tracker_*`（對應 `main` 內含 `manifest` 的後處理）。
- 註解除檔名外，可補 **對應函式**（例如 `// maps attention_forward`），方便從 Python 跳轉。

## 15. `_bi.txt` 格式與展平順序

- 每行一個元素；位寬與 Q 格式以專案既有約定為準（預設 **Q8.8、16-bit 二進位字串**，與 `write_bi` 一致）。
- 張量展平須與 numpy **C-order / row-major** 一致。若 RTL 因管線使用不同讀序，須**另產「RTL 順序」golden** 或在註解中**白紙黑字寫 reorder 規則**，不可默認與 `flatten()` 相同卻未說明。

## 16. 對拍順序（除錯策略）

- 依資料流**前向**對拍，順序對齊 numpy trunk `main`（post-embed → merged → blocks 0..START_LAYER → adaptive → selected block → backbone norm → head → bbox → tracker）。
- **第一個不一致的節點**應先釐清是規格、Python 量化還是 RTL，再往下修；禁止只在最末級比對卻不回頭查中間節點。

## 17. numpy trunk 內部已知語意陷阱（對齊時必查）

- `linear()` 對啟用／權重使用 **`np.round`** 量化；`conv2d()` 對輸入使用 **`np.round`**。RTL 與 binary check **須分模組**與各自 golden 對齊，不可假設同一捨入。
- `_recip_nr`、`_inv_sqrt_nr` 為 **float64 迭代**；RTL 多為 **定點 + LUT + 固定迭代次數**。對拍以**已書面化的定點規格**為準；若要求與 float 完全一致，應先改 Python 為整數語意再產 golden。

## 18. 分支與驗證範圍

- `main` 中 **讀取既有 adaptive golden** 與 **無 golden 時自行算 adaptive** 為兩條路；規格／TB 應註明 RTL **預設驗證哪一條**，另一條可列為軟體或第二版。
- `manifest` 是否存在決定是否產生 `tracker_*` golden；RTL 若僅驗 backbone+head，模組註解應標 **「tracker 僅在 manifest 路徑存在時有 golden」**。

## 19. 模組頭建議欄位

- 每個主要 `.v` 建議簡述：**對應 numpy 函式**、**輸入／輸出形狀**、**start/busy/done 語意**、以及與第 7、12 節一致的 **Golden / Golden-Weight 檔名**。

## 20. ROM 再產生（盡量避免）

- 已假設 **ROM／RAM 初值檔已在外部生成**；修改 Verilog 時**盡量不要**動到會迫使 **整批重產 ROM** 的內容（例如：權重展平順序、位址對照表、每筆位寬、ROM 深度、與 `gen_rom`／export 腳本綁定的編碼方式、LUT 表內容與其衍生規格）。
- 優先以 **不改 ROM 本體** 的方式除錯：FSM、握手、暫存器解碼、pipeline 對齊、`wgt_addr` 解讀錯誤、與 golden 對照的索引錯誤等。
- **若無法避免**（任何變更會導致需重新跑 ROM 生成流程、或更換已燒入的 image），Agent **必須在回覆中明確告知使用者**，並條列：
  - **為何**必須重產 ROM、
  - **受影響的 ROM／檔案**、
  - **建議重新執行的指令或腳本**（若專案內有固定流程可指名）。
- 同樣原則適用於 **內容與規格綁死的 LUT ROM**（例如 sigmoid／inv_sqrt／recip 種子表）：變更演算法或 Q 格式導致表內容變更時，視同需重產 ROM，須先告知使用者。
- 若依 **§5** 修改 numpy／golden 且該變更會連動 **ROM 初值內容**（例如權重重新匯出），亦須依本節向使用者說明並列出重產步驟。

## 21. Activation 階段對拍（numpy 輸出 ≡ Verilog 同階段輸出）

- **`run_backbone_numpy_shared_trunk.py` 產出至 `Activation/` 的每一份 activation**（經 `save_npy`→`write_bi` 等路徑對應之 `*_bi.txt`），代表該**運算階段**在約定定點語意下的**參考輸出**。
- **Verilog** 執行到**對應階段**時（同一運算邊界：例如指定 block 的 norm1 輸出、attention 後、`box_head` 某 map 等），該階段呈現在匯流排／暫存／串流上的 activation **必須與上述 golden 對答案**（逐元素比對：同一 flatten 順序、同一 Q 格式；允許誤差時須書面定義）。  
  **兩邊是同一套驗證關係**：numpy 檔為「參考」，RTL 為「待驗」；不可只做功能仿真而不對檔。
- 「階段」與 golden 檔名的對應以 **§7** 註解為準；格式與展平順序見 **§15**。
- 除錯時依 **§16** 自資料流前向縮小；禁止只看最終輸出而不對中間 activation。
- 若刻意讓 RTL 語意與現有 golden 分歧（例如改定點規格），須**先**更新 numpy／重跑 golden（並依 **§20** 評估是否重產 ROM），再對拍 RTL。

## 22. VCS 模擬流程（使用者環境）

- 本專案 RTL 模擬在 **Synopsys VCS** 下執行；與 **§10**（模擬不在本 repo）並存時，以使用者實際編譯指令為準。
- **步驟 1 — 編譯**：在含 RTL／TB 與 `memory/` ROM／SRAM 模型的目錄執行 **VCS**，將輸出 tee 至記錄檔（路徑／檔名依使用者習慣，例如 `runvcs.log`）。
- **步驟 2 — 執行**：編譯成功後執行 **`./simv`** 開始仿真。
- **參考指令結構**（與目前使用者流程一致；實際請在**編譯根目錄**執行，`memory/` 為 ROM／SRAM macro）：見下列 shell（可比對 repo 內檔案是否齊全後微調）。

```bash
vcs *.v \
memory2/Sram_tok1.v \
memory2/Sram_tok2.v \
memory2/Sram_q.v \
memory2/Sram_k.v \
memory2/Sram_v.v \
memory2/Sram_qkm.v \
memory2/Sram_c2w.v \
memory2/rom_backbone_blocks_0_3_attn_qkv_weight.v \
memory2/rom_backbone_blocks_0_3_mlp_fc1_weight.v \
memory2/rom_backbone_blocks_0_3_mlp_fc2_weight.v \
memory2/rom_backbone_blocks_0_6_attn_proj_bias.v \
memory2/rom_backbone_blocks_0_6_attn_proj_weight.v \
memory2/rom_backbone_blocks_0_6_attn_qkv_bias.v \
memory2/rom_backbone_blocks_0_6_mlp_fc1_bias.v \
memory2/rom_backbone_blocks_0_6_mlp_fc2_bias.v \
memory2/rom_backbone_blocks_0_6_norm1_bias.v \
memory2/rom_backbone_blocks_0_6_norm1_weight.v \
memory2/rom_backbone_blocks_0_6_norm2_bias.v \
memory2/rom_backbone_blocks_0_6_norm2_weight.v \
memory2/rom_backbone_blocks_4_6_attn_qkv_weight.v \
memory2/rom_backbone_blocks_4_6_mlp_fc1_weight.v \
memory2/rom_backbone_blocks_4_6_mlp_fc2_weight.v \
memory2/rom_backbone_norm_bias.v \
memory2/rom_backbone_norm_weight.v \
memory2/rom_box_head_shared_conv1_2_folded_bias.v \
memory2/rom_box_head_shared_conv1_folded_weight1.v \
memory2/rom_box_head_shared_conv1_folded_weight2.v \
memory2/rom_box_head_shared_conv2_folded_weight1.v \
memory2/rom_box_head_shared_conv2_folded_weight2.v \
memory2/rom_box_head_shared_conv2_folded_weight3.v \
memory2/rom_box_head_tail_ctr_offset_size_bias.v \
memory2/rom_box_head_tail_ctr_offset_size_weight.v \
+lint=TFIPC-L -debug_access+all -debug_region+cell +define+TSMC_CM_NO_WARNING \
| tee -i runvcs.log

./simv | tee -i simv.log
```

- **注意**：`TEST.v` 等 TB 若使用 **`$readmemb("./TXT_File/...")`**，須在**執行 `simv` 時的工作目錄**能解析該相對路徑（見 `TEST.v` 註解）；Agent 除錯時不假定可在 Cursor 沙盒內跑通 VCS，仍以使用者提供的 **`simv.log`／波形／`$display` 截圖**為準；若僅能 log 無波形，見 **§10.1**。

## 與其他 rule 的關係

- 大方向與 `python-to-rtl-plan.mdc` 一致時從之；本檔針對 **numpy trunk + 指定 golden 目錄 + verilog 目錄** 補充可操作細節。
- 撰寫／修改 `verilog_backbone2/*.v`、`verilog_head2/*.v`、`verilog/*.v` 或（僅在使用者指定時）`verilog_head/*.v` 時一併遵守 `verilog_rule.mdc`（含 **平行 `if` 拆分**、**跨 `always` 多驅動** 自查）；目錄分工見 **§1**（**head 預設以 `verilog_head2/` 為準**）。
- 若 `hw-friendly-design.mdc` 與本檔對 attention／定點敘述並存，以 **可對拍之 numpy trunk 行為** 為準；設計原則檔僅作輔助說明。

## 語言規則
- 僅使用 Verilog-2001 語法，禁止 SystemVerilog 特性（logic, always_ff, always_comb, interface, typedef, enum, struct, package, assertions）

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
