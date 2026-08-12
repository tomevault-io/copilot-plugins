## sram-suggestion

> 將reg修改成SRAM


# RTL → SRAM 改造建議（backbone + head + 整合版 verilog2）

> **規格**：Q8.8（16-bit signed），全專案共用。
> **規則依據**：[.cursor/rules/verilog_rule.mdc](../../.cursor/rules/verilog_rule.mdc)（**§7.7 SRAM 時序**、**§8 Reg→SRAM 檢查清單**）、[.cursor/rules/numpy-trunk-to-verilog.mdc](../../.cursor/rules/numpy-trunk-to-verilog.mdc)
> **判斷門檻**（全文件通用）：
>
> | 等級 | depth | 處理建議 |
> |------|-------|----------|
> | **必改 SRAM** | depth ≥ 1024（≥ 16 Kb / 2 KB 起跳） | 使用 1R1W 或單埠 SRAM macro |
> | **建議 SRAM** | 512 ≤ depth < 1024 | 視製程 macro 表決定，通常 ≥ 256 word 即可省面積 |
> | **可留 reg** | depth < 256 | flip-flop register file 即可 |
>
> 容量計算：**bits = depth × width**，**KB = bits / 8 / 1024**。

---

## 工作流總覽

| 目錄 | 角色 | SRAM 改造作為 |
|------|------|--------------|
| [python/lib/models/verilog_backbone2/](../lib/models/verilog_backbone2/) | **分模組除錯**：backbone 獨立驗證 | Phase 1：先在此改 SRAM port，獨立通過對拍 |
| [python/lib/models/verilog_head2/](../lib/models/verilog_head2/) | **分模組除錯**：head 獨立驗證 | Phase 2：先在此改 SRAM port，獨立通過對拍 |
| [python/lib/models/verilog2/](../lib/models/verilog2/) | **整合版**：backbone + head 端到端 | Phase 3：同步上述改動，加入 sglatrack_top 共享 SRAM mux |

> **本文件結構**：第一、二部分為 verilog_backbone2 / verilog_head2 的獨立分析（debug 階段參考），**第三部分為最終整合方案（verilog2，採用 Unified Shared SRAM Pool）**。

---

## 編譯開關與 RTL 雙路徑（`USE_REG_BUF` / `USE_SRAM_BUF`）

改 activation buffer 時，**必須**在 VCS 與 RTL 內維持兩條互斥路徑，方便 regression 與除錯：

| `+define` | 行為 |
|-----------|------|
| **`USE_REG_BUF`** | 保留 legacy **大 `reg` 陣列**（flip-flop register file）；對拍 baseline |
| **`USE_SRAM_BUF`** | 拆掉對應大 `reg`，改接 **SHC-SPMBSRAM compiler 產出的 macro**（見 §3.9 指令） |

**規則（必守）**：

1. **二選一**：禁止同時 `+define+USE_REG_BUF` 與 `+define+USE_SRAM_BUF`。 [TEST_backbone.v](../lib/models/verilog_backbone2/TEST_backbone.v) 若兩者皆定義會 `$finish`；兩者皆未定義時 TB 預設 `` `define USE_REG_BUF ``。
2. **RTL 用 `` `ifdef USE_REG_BUF `` / `` `ifndef USE_REG_BUF ``**（或對稱的 `USE_SRAM_BUF`）包住 reg 陣列與 SRAM mux／latency 邏輯；**同一模組內不可混用未 ifdef 的 `reg [15:0] buf [0:N-1]` 與 macro**。
3. **改完一步就對拍一步**：先確認 `USE_REG_BUF` 仍 PASS，再切 `USE_SRAM_BUF`。
4. **編譯列表示例**（macro `.v` 路徑依工作站；**不要**依賴 `*_wrap.v`）：
   ```bash
   # Reg baseline
   vcs verilog_backbone2/*.v memory/*.v \
     +define+TSMC_CM_NO_WARNING +define+USE_REG_BUF | tee runvcs.log

   # SRAM（含 compiler 產物；路徑依工作站）
   vcs verilog_backbone2/*.v memory/*.v \
     /path/Sram_tok1.v /path/Sram_tok2.v \
     /path/Sram_x.v /path/Sram_q.v /path/Sram_k.v /path/Sram_v.v /path/Sram_qkm.v \
     +define+TSMC_CM_NO_WARNING +define+USE_SRAM_BUF | tee runvcs.log
   ./simv | tee simv.log
   ```

> **注意**：`Sram_tok1` 與 `Sram_x` 為**不同 compiler 模組檔**（深度皆可用 12288×16，但 **不可** 只編譯一種檔名取代全部 instance）。instance 顆數見 **§1.0**。

---

# 第一部分：Backbone（`verilog_backbone2/`）— 分模組除錯版

> **目錄**：[python/lib/models/verilog_backbone2/](../lib/models/verilog_backbone2/)
> **常數**：`EMBED_DIM=32`、`NUM_HEADS=4`、`HEAD_DIM=8`、`N_TOKENS=320`、`MLP_DIM=128`
> **注意**：此版為**分模組除錯目錄**，`backbone_top` 內 **7 次** 呼叫 `transformer_block`（每 block 一組 `u_attn` macro）。整合到 `verilog2` 後為 **單一 `transformer_block` instance reuse 7 次**（見第三部分）。

## 1.0 Phase 1 實作快照（`USE_SRAM_BUF`，2026-05）

> 下列為 **目前 RTL 已落地** 之配置；與 **第三部分 Unified Pool（Phase 3）**、**§5.3 跳過 S_LOAD_X（Phase 3 目標）** 不同者已標 **(目標)**。

### 1.0.1 各檔 inline macro 顆數（`+define+USE_SRAM_BUF`）

| 模組 | macro 型號 | instance 名 | 顆數 | 有效深度×寬 | 對應 legacy reg |
|------|------------|-------------|------|-------------|-----------------|
| [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v) | `Sram_tok2` | `u_sram_tok2` | **1** | 10240×16 | `tok_buf`（block 間 ping-pong） |
| [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v) | `Sram_tok1` | `u_sram_tok1` | **1** | 10240×16 | `out_buf`（backbone norm 輸出） |
| [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v) | `Sram_tok1` | `u_sram_x` | **1** | 10240×16 | `x_buf`（主幹＋residual 錨點） |
| [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v) | `Sram_tok1` | `u_sram_tmp` | **1** | 10240×16 | `tmp_buf`（norm1/2、attn、mlp 暫存） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_x` | `u_sram_x` | **1** | 10240×16 | `x_in_buf`（norm1 快照；**與 `Sram_q` 不可共用**，見 §1.0.3） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_q` | `u_sram_q` | **1** | 10240×16 | `q_buf`（head-major） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_k` | `u_sram_k` | **1** | 10240×16 | `k_buf` |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_v` | `u_sram_v` | **1** | 10240×16 | `v_buf` + **`ao_buf`（deviation：時間多工）** |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_qkm` | `u_sram_qkm` | **1** | 1280×16 | `qkm_buf` + `zr_buf`（時間多工） |
| [mlp.v](../lib/models/verilog_backbone2/mlp.v) | — | — | **0** | — | **無** 本地 10240 SRAM；FC1 讀 parent `u_sram_tmp` |

**單次 `backbone_top` 仿真編譯（macro `.v` 檔）**：`Sram_tok1`×2 檔案 + `Sram_tok2`×1 + `Sram_x`×1 + `Sram_q`×1 + `Sram_k`×1 + `Sram_v`×1 + `Sram_qkm`×1 = **7 種** compiler 產物。

**單次 `backbone_top` 仿真 instance 總數**（7 blocks）：`Sram_tok1`×(1+7×2)=**15**、`Sram_tok2`×1、`Sram_x`×7、`Sram_q/k/v/qkm`×7 各 = **15+1+7+28 = 51** 顆 macro instance（面積估算用；非 Phase 3 的 7 顆 pool）。

### 1.0.2 `mlp` 與 `transformer_block` 資料流（已實作）

- **`mlp`**：已刪 `x_in_buf` / `S_LOAD_X`；`S_FC1` 以 **`norm_rd_en` + `norm_rd_flat` + `norm_x`** 2-phase 讀 parent **`u_sram_tmp`**（norm2，token-major）。
- **`S_MLP_FEED`**：單拍 `mlp_start` → `S_MLP_WAIT`（不再 10240 拍串流餵 mlp）。
- **`care_attention`**：**仍保留** `S_LOAD_X` → `Sram_x`（parent `S_ATTN_WAIT` 會把 attn 輸出寫回 `u_sram_tmp`，覆蓋 norm1；故 QKV 前須快照 norm1）。
- **(目標 Phase 3)**：刪 `S_LOAD_X`、norm 直接寫共用 pool、`care_attention` 從 `S_QKV` 讀同址——見 §5.3.1。

### 1.0.3 為何 `Sram_x` 與 `Sram_q` 必須分開（Phase 1 已驗證）

同一 10240 深度、**不同 flatten** 時，flat index 會 **alias**：

| 語意 | 展平 |
|------|------|
| norm1 **x**（token-major） | `tok × EMBED_DIM + feat` |
| **q**（head-major） | `h × (N_TOKENS×HEAD_DIM) + tok × HEAD_DIM + d` |

例：`h=1, tok=0, d=0` → index **2560** = `tok=80, feat=0` 的 x。`S_QKV` 寫 q 與讀 x **重疊** 時，共用一顆 SRAM 會踩資料（典型症狀：QKV golden 約半數 FAIL）。通則见 [verilog_rule.mdc §7.7.7](../../.cursor/rules/verilog_rule.mdc)。

### 1.0.4 `u_sram_tmp` port mux 優先級（`transformer_block`，建議固定）

1. `tmp_wr_do`（`layer_norm` `out_beat_o` latch 寫入，norm1/2 capture）
2. `mlp_norm_rd_en`（mlp FC1 ADDR 讀 norm2）
3. `S_ATTN_FEED` + `feed_phase==0`（attn feed ADDR 讀）
4. `attn_yv` / `mlp_yv` 寫 capture
5. `S_RES1` / `S_RES2` residual 讀

---

## 1.1 各檔案 buffer 對照（`USE_REG_BUF` legacy ↔ `USE_SRAM_BUF` macro）

> 行號僅供參考，以 `` `ifdef USE_REG_BUF `` 區塊為準。

### A. [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v)

| Legacy reg | SRAM（已實作） | depth × width | 狀態 |
|------------|----------------|---------------|------|
| `tok_buf` | `u_sram_tok2` (`Sram_tok2`) | 10240 × 16 | **已改** |
| `out_buf` | `u_sram_tok1` (`Sram_tok1`) | 10240 × 16 | **已改**；norm capture 見 §5.7.1 |

### B. [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v)

| Legacy reg | SRAM（已實作） | depth × width | 狀態 |
|------------|----------------|---------------|------|
| `x_buf` | `u_sram_x` (`Sram_tok1`) | 10240 × 16 | **已改**（模組內 inline，非 top port） |
| `tmp_buf` | `u_sram_tmp` (`Sram_tok1`) | 10240 × 16 | **已改**；mlp／attn 共用 mux |

**(目標 Phase 3)**：改為 top 7-pool port pass-through，刪模組內雙 macro。

### C. [care_attention.v](../lib/models/verilog_backbone2/care_attention.v)

| Legacy reg | SRAM（已實作） | depth × width | 狀態 |
|------------|----------------|---------------|------|
| `x_in_buf` | `u_sram_x` (`Sram_x`) | 10240 × 16 | **已改**；`S_LOAD_X` 寫、`S_QKV` 2-phase 讀 |
| `q_buf` | `u_sram_q` | 10240 × 16 | **已改** |
| `k_buf` | `u_sram_k` | 10240 × 16 | **已改** |
| `v_buf` / `ao_buf` | `u_sram_v` | 10240 × 16 | **已改**（ao 與 v 時間多工，見模組頭 deviation） |
| `qkm_buf` / `zr_buf` | `u_sram_qkm` | 1280 × 16 | **已改** |

**可留 reg：** `km_buf` (32)、`kv_buf` (256)。macro **inline 於本模組**（Option A），非 `backbone_top` 下傳 port。

### D. [mlp.v](../lib/models/verilog_backbone2/mlp.v)

| Legacy reg | SRAM（已實作） | 狀態 |
|------------|----------------|------|
| ~~`x_in_buf`~~ | **無**；讀 parent `norm_rd_*` / `u_sram_tmp` | **已刪** |
| `fc1_buf` | reg `[128]` | **保留**（256 B，可留 reg） |

**(目標 Phase 3)**：若 top 共用 pool，改為 port 讀共用 SRAM，仍不恢復本地 10240 `x_in_buf`。

### E. 可全部留 reg 的檔案

- [linear.v](../lib/models/verilog_backbone2/linear.v)：`x_buf / wgt_buf` 32 entries each（64 B）
- [linear_wide.v](../lib/models/verilog_backbone2/linear_wide.v)：`x_buf / wgt_buf` 128 entries each（256 B；邊緣）
- [layer_norm.v](../lib/models/verilog_backbone2/layer_norm.v)：`feat_buf` 32 entries
- [residual.v](../lib/models/verilog_backbone2/residual.v)、[recip_nr.v](../lib/models/verilog_backbone2/recip_nr.v) 等：純 datapath，無陣列

### F. Testbench 不列入

- [TEST_backbone.v](../lib/models/verilog_backbone2/TEST_backbone.v) 的 `GOLD_BB / TEMPL_MEM / SRCH_MEM`：`$readmemb` 用，不進晶片。

---

# 第二部分：Head（`verilog_head2/`）— 分模組除錯版

> **目錄**：[python/lib/models/verilog_head2/](../lib/models/verilog_head2/)
> **常數**：`IN_CH=32`、`FEAT_H=FEAT_W=16`（`FEAT_SZ=256`）、`C_SH1=96`、`C_SH2=48`、`MAP2_LEN=512`（cal_bbox）、`K=3`、`PAD=1`、tail `IN_CH=48`
> **來源**：`head_top.v` 已自 [verilog2/head_top.v](../lib/models/verilog2/head_top.v) 同步（模組頭註明 *Development copy: edit here first*）；[TEST_head.v](../lib/models/verilog_head2/TEST_head.v) 僅串流 backbone token 進 `head_top`，**不再**在 TB 內承載 feature map。

## 2.1 各檔案需改成 SRAM 的 `reg` 訊號

### A. [head_top.v](../lib/models/verilog_head2/head_top.v) — **必改 SRAM（面積主因）**

| 訊號 (行號) | 宣告 | depth × width | 容量 | 等級 | 角色／golden |
|-------------|------|---------------|------|------|----------------|
| `x_buf` [head_top.v:75](../lib/models/verilog_head2/head_top.v#L75) | `[0:IN_LEN_HEAD-1]` | **8192 × 16** | **16 KB** | **必改** | conv1 輸入 NCHW（僅 search 8192）；fill 自 `backbone_after_norm_backbone_out_bi.txt`（skip 前 2048 template） |
| `sh1_buf` [head_top.v:76](../lib/models/verilog_head2/head_top.v#L76) | `[0:C1_LEN-1]` | **24576 × 16** | **48 KB** | **必改** | conv1 輸出／conv2 輸入；對 `box_head_shared_conv1_*` 中間 activation |
| `sh2_buf` [head_top.v:77](../lib/models/verilog_head2/head_top.v#L77) | `[0:C2_LEN-1]` | **12288 × 16** | **24 KB** | **必改** | conv2 輸出／tail 輸入；對 `box_head_shared_conv2_*` 中間 activation |

**localparam（與 RTL 一致）**：

| 符號 | 算式 | 值 |
|------|------|-----|
| `IN_LEN_HEAD` | `FEAT_SZ × IN_CH` | 8192 |
| `C1_LEN` | `C_SH1 × FEAT_SZ` | 24576 |
| `C2_LEN` | `C_SH2 × FEAT_SZ` | 12288 |

**可留 reg（head_top 內）**：

| 訊號 | 說明 |
|------|------|
| `bbox_reg [0:3]` [head_top.v:79](../lib/models/verilog_head2/head_top.v#L79) | 4×16-bit 最終 bbox 暫存 |
| `c1_x_i_q` / `c2_x_i_q` / `t_x_i_q` | 各 1 word 讀取 pipeline 暫存（改 SRAM 後保留，對齊 macro 1-cycle read） |
| `fill_cnt`、`c1_wr_idx`、`c2_wr_idx`、FSM `state` 等 | 控制／計數，非陣列 |

**讀寫時序（改 SRAM 時須寫進模組頭契約）**：

| 緩衝 | 寫入 | 讀取（餵子模組） |
|------|------|------------------|
| `x_buf` | `S_FILL`：`posedge clk`，`fill_search` 時 `x_buf[fill_dst] <= a_i` | `negedge clk`：`c1_x_i_q <= x_buf[c1_x_addr]`（conv1 `X_AW=14`） |
| `sh1_buf` | `negedge clk`：`c1_y_valid` 時依 `c1_wr_idx` 寫入 | `negedge clk`：`c2_x_i_q <= sh1_buf[c2_x_addr]`（conv2 `X_AW=15`） |
| `sh2_buf` | `negedge clk`：`c2_y_valid` 時依 `c2_wr_idx` 寫入 | `negedge clk`：`t_x_i_q <= sh2_buf[t_x_addr]`（tail `X_AW=15`） |

> 與 [verilog_rule.mdc §7.7](../../.cursor/rules/verilog_rule.mdc) 一致：若 macro 為 **`CLK(~clk)`**、controller 用 **`posedge clk` 送址**，則 **`Q` 在下一拍 `posedge` 有效**；目前 `negedge` 鎖 `*_i_q` 等效於在 macro 讀出穩定後再採樣，改 SRAM 時勿改成「同拍 addr + 同拍進 MAC」。

**PDK 拆顆（`sh1_buf`）**：24576 > SHC-SPMBSRAM Mux16 最大深度 16384 → **單顆 macro 無法涵蓋**，須拆成 **2 × 12288 × 16**（與第三部分 §3.8 相同）：

```verilog
wire        sh1_sel        = sh1_addr[13];   // 0 -> SRAM lo, 1 -> SRAM hi
wire [12:0] sh1_local_addr = sh1_addr[12:0];
```

**head-only 除錯階段建議 macro（不與 backbone 共用時）**：

| 用途 | 建議 compiler 指令 | 備註 |
|------|-------------------|------|
| `x_buf` | `Sram_head_x   8192 16 8 s` 或併入 12288 池 | 8192 可用 Mux8；若與整合版對齊可升到 12288 留 spare |
| `sh1_buf` lo/hi | `Sram_head_sh1a 12288 16 16 s` ×2 | 合計 48 KB |
| `sh2_buf` | `Sram_head_sh2 12288 16 16 s` | 24 KB，深度剛好 |

**head2 Phase 2 建議 macro 顆數（獨立除錯、不與 backbone 共用 pool）**：

| # | 用途 | compiler 指令 | 顆數 | depth×width | 備註 |
|---|------|---------------|------|-------------|------|
| 1 | `x_buf` | `Sram_head_x 8192 16 8 s`（或 12288 對齊 pool） | **1** | 8192×16 | conv1 輸入 |
| 2–3 | `sh1_buf` lo/hi | `Sram_head_sh1a 12288 16 16 s` ×2 | **2** | 12288×16×2 | **24576 拆 2 顆**（§2.1、`sh1_addr[13]` 選 bank） |
| 4 | `sh2_buf` | `Sram_head_sh2 12288 16 16 s` | **1** | 12288×16 | conv2 出／tail 入 |
| (opt) | conv2 `wgt_buf` | `Sram_c2w 1056 16 4 s` | **1** | 1056×16 | 見 §3.8（864→1056） |
| (opt) | `size_buf`+`off_buf` | `Sram_bbox 1024 16 4 s` 或併 pool | **1** | 1024×16 | `addr[9]` 區分 size/off |

**合計（feature map 必改）**：**4 顆**（1 + sh1×2 + sh2）；加 conv2 weight + bbox 則 **6 顆**。

Phase 2 先以 **4～6 顆獨立 SRAM** 通過 head 對拍；Phase 3 再併入第三部分 **7 顆** Unified Pool（見 §3.3）。

### B. [conv.v](../lib/models/verilog_head2/conv.v) — **conv2 建議 SRAM**

| 訊號 (行號) | 實例 | depth × width | 容量 | 等級 |
|-------------|------|---------------|------|------|
| `wgt_buf` [conv.v:132](../lib/models/verilog_head2/conv.v#L132) | conv1 (`ROM_PROFILE=1`, IN_CH=32) | 288 × 16 | 576 B | 邊緣／可留 reg |
| `wgt_buf` [conv.v:132](../lib/models/verilog_head2/conv.v#L132) | **conv2** (`ROM_PROFILE=2`, IN_CH=96) | **864 × 16** | **1.7 KB** | **建議 SRAM**（整合版升到 **1056×16**，見 §3.8） |

### C. [cal_bbox.v](../lib/models/verilog_head2/cal_bbox.v) — **建議 SRAM**

| 訊號 (行號) | depth × width | 容量 | 等級 |
|-------------|---------------|------|------|
| `size_buf` [cal_bbox.v:106](../lib/models/verilog_head2/cal_bbox.v#L106) | 512 × 16 | **1 KB** | 建議（可與 `off_buf` 合併 1 顆 1024-deep，`addr[9]` 區分） |
| `off_buf` [cal_bbox.v:107](../lib/models/verilog_head2/cal_bbox.v#L107) | 512 × 16 | **1 KB** | 建議 |

tail 串流寫入 `size_buf`／`off_buf`（`S_SIZE`／`S_OFF`），`S_CAL` 再讀出算 bbox；改 SRAM 時須在 FSM 上區分 **寫入拍** 與 **讀出拍**（見 §3.4 Head BBOX 列）。

### D. [tail.v](../lib/models/verilog_head2/tail.v) — 可留 reg

`wgt_buf [0:IN_CH-1]`（48 entries）= **96 B**，flip-flop OK。

### E. [sigmoid_lut.v](../lib/models/verilog_head2/sigmoid_lut.v) — 無需改

65-entry case-ROM，合成為組合邏輯，不佔 flip-flop。

### F. Testbench 不列入晶片

[TEST_head.v](../lib/models/verilog_head2/TEST_head.v) 的 `raw_in`／`bbox_gold`：`$readmemb` 用，不進晶片。

---

## 2.2 `verilog_head2` head 路徑容量小計（reg → SRAM 前）

| 模組 | 必改／建議陣列 | 合計（約） |
|------|----------------|------------|
| `head_top` | `x_buf` + `sh1_buf` + `sh2_buf` | **88 KB** |
| `conv` (conv2) | `wgt_buf` | 1.7 KB（建議） |
| `cal_bbox` | `size_buf` + `off_buf` | 2 KB（建議，可併 1 顆） |
| **head-only 若全改 SRAM** | — | **~ 92 KB**（不含 ROM） |

與 [verilog2](../lib/models/verilog2/) 第三部分 head 路徑表（§3.2）一致；差異在於 **head2 此階段不強制與 backbone 共用 pool**，macro 顆數可先多後併。

## 2.3 Phase 2 重構要點（`head_top` 已有 RTL buffer）

1. **拆掉** [head_top.v](../lib/models/verilog_head2/head_top.v) 內 `x_buf`／`sh1_buf`／`sh2_buf` 的 `reg [...]` 陣列，改 **3+1 組 SRAM port**（`sh1` 拆 lo/hi）。
2. **保留** `S_FILL`（token 串流仍須寫入 `x_buf`／對應 SRAM）；與整合版「刪 S_FILL、資料已在 pool」不同，**head-only TB 仍依賴 fill**。
3. `sh1` 位址 **bit[13]** 選 bank；`sh2` 單顆 12288 即可。
4. 同步改 [conv.v](../lib/models/verilog_head2/conv.v) conv2 `wgt_buf`、[cal_bbox.v](../lib/models/verilog_head2/cal_bbox.v) `size_buf`／`off_buf`（可參考 §3.7 依賴表）。
5. 用行為模型 `sram_sp` + `DUMP_HEAD_*`（若需要）對 `Activation/box_head_*_bi.txt` 逐層對拍；通過後再 copy `head_top.v` → [verilog2/head_top.v](../lib/models/verilog2/head_top.v)。

---

# 第三部分：整合版 `verilog2/` — Unified Shared SRAM Pool（**最終方案**）

> **目錄**：[python/lib/models/verilog2/](../lib/models/verilog2/)
> **設計取捨**：**捨棄 Power Gating 細粒度，換取最大 SRAM 共用**（見 §3.5）。

## 3.1 verilog2 重點結構發現

### (1) Backbone 是「**1 個 transformer_block instance 跑 7 次**」

從 [backbone_top.v:108](../lib/models/verilog2/backbone_top.v#L108)：

```verilog
transformer_block #(...) u_tb (...);    // ← 只有 1 個 instance
```

→ **不是** 7 個獨立 hardware copy，而是 1 個硬體 + `tok_buf` 回放 7 次。
→ 所以 `transformer_block / care_attention / mlp` 內的 reg 陣列**只算 1 份**，不像第一部分要 ×7。

### (2) Head 的 feature memory **已在 RTL 內**（verilog2 與 verilog_head2 同源）

[verilog2/head_top.v:76-78](../lib/models/verilog2/head_top.v#L76) 與 [verilog_head2/head_top.v:75-77](../lib/models/verilog_head2/head_top.v#L75) 皆為：

```verilog
reg [DATA_W-1:0] x_buf   [0:IN_LEN_HEAD-1];  // 8192  → 16 KB
reg [DATA_W-1:0] sh1_buf [0:C1_LEN-1];       // 24576 → 48 KB
reg [DATA_W-1:0] sh2_buf [0:C2_LEN-1];       // 12288 → 24 KB
```

→ 除錯目錄與整合版 **語意一致**；SRAM 改造以 **head2 先改、再 sync verilog2** 為準（見第二部分 §2.3）。

### (3) `sglatrack_top.v` 已有時間互斥 FSM

從 [sglatrack_top.v:84-99](../lib/models/verilog2/sglatrack_top.v#L84)：

```
S_IDLE → S_BACKBONE → S_HEAD → S_DONE
              ↑              ↑
        Head 完全 idle    Backbone 完全 idle
```

→ **共用 SRAM 的天然舞台**。

## 3.2 verilog2 全部 reg 陣列清單（depth ≥ 256）

### Backbone 路徑（1 份 hardware）

| 模組 (行號) | 訊號 | depth × width | 容量 |
|------------|------|---------------|------|
| [backbone_top.v:83](../lib/models/verilog2/backbone_top.v#L83) | `tok_buf` | 10240 × 16 | **20 KB** |
| [backbone_top.v:165](../lib/models/verilog2/backbone_top.v#L165) | `out_buf` | 10240 × 16 | **20 KB** |
| [transformer_block.v:63](../lib/models/verilog2/transformer_block.v#L63) | `x_buf` | 10240 × 16 | **20 KB** |
| [transformer_block.v:64](../lib/models/verilog2/transformer_block.v#L64) | `tmp_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:97](../lib/models/verilog2/care_attention.v#L97) | `x_in_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:98](../lib/models/verilog2/care_attention.v#L98) | `q_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:99](../lib/models/verilog2/care_attention.v#L99) | `k_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:100](../lib/models/verilog2/care_attention.v#L100) | `v_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:105](../lib/models/verilog2/care_attention.v#L105) | `ao_buf` | 10240 × 16 | **20 KB** |
| [care_attention.v:102](../lib/models/verilog2/care_attention.v#L102) | `qkm_buf` | 1280 × 16 | **2.5 KB** |
| [care_attention.v:103](../lib/models/verilog2/care_attention.v#L103) | `zr_buf` | 1280 × 16 | **2.5 KB** |
| [mlp.v](../lib/models/verilog2/mlp.v) | ~~`x_in_buf`~~ | — | **已於 backbone2 刪除**；norm2 讀 parent `tmp` |

> **重要**：legacy 清單中多個 10240×16 buf 為 **module 邊界重複語意**（`x_buf` / `x_in_buf` / 舊 `mlp.x_in`）。**Phase 1 已實作**：`mlp` 不再持有一份；`care_attention` 仍持 **`Sram_x` 快照**（因 `tmp` 會被 attn 覆寫）。**Phase 3 目標**：摺疊成 §3.3 的 **7 顆 Unified Pool**。

### Head 路徑（1 份 hardware）

| 模組 (行號) | 訊號 | depth × width | 容量 |
|------------|------|---------------|------|
| [head_top.v:73](../lib/models/verilog2/head_top.v#L73) | `x_buf` | 8192 × 16 | **16 KB** |
| [head_top.v:74](../lib/models/verilog2/head_top.v#L74) | `sh1_buf` | 24576 × 16 | **48 KB** |
| [head_top.v:75](../lib/models/verilog2/head_top.v#L75) | `sh2_buf` | 12288 × 16 | **24 KB** |
| [conv.v:132](../lib/models/verilog2/conv.v#L132) (u_conv2) | `wgt_buf` | 864 × 16 | **1.7 KB** |
| [cal_bbox.v:106](../lib/models/verilog2/cal_bbox.v#L106) | `size_buf` | 512 × 16 | **1 KB** |
| [cal_bbox.v:107](../lib/models/verilog2/cal_bbox.v#L107) | `off_buf` | 512 × 16 | **1 KB** |

**可留 reg：** conv1 `wgt_buf` (288)、tail `wgt_buf` (48)、km_buf (32)、kv_buf (256)、fc1_buf (128)、layer_norm `feat_buf` (32)。

## 3.3 Unified Shared SRAM Pool — **總共 7 顆**

關鍵洞察：**真正同時存在的 activation 只有少數幾個**。把 12 條 backbone reg 陣列摺疊成 5~6 顆實體 SRAM，再與 head 共用。

### 共用對應表

| # | Macro 名稱 | macro 規格 | **Backbone phase** 用途 | **Head phase** 用途 |
|---|-----------|-----------|-----------------------|---------------------|
| **1** | `Sram_tok1` | 12288 × 16 SP | `tok_buf` ping（block 間 activation，使用 10240）| `x_buf`（conv1 input，使用 8192，addr offset 2048 跳 template）|
| **2** | `Sram_tok2` | 12288 × 16 SP | `tok_buf` pong / `out_buf`（使用 10240）| `sh2_buf`（conv2 out / tail in，使用 12288 ✓）|
| **3** | `Sram_q` | 12288 × 16 SP | **`q_buf` only**（Phase 1 另需 **`Sram_x`** 存 norm1；`ao` 在 `Sram_v`）| `sh1_buf` 前半（使用 12288 ✓）|
| **4** | `Sram_k` | 12288 × 16 SP | `k_buf`（使用 10240）| `sh1_buf` 後半（使用 12288 ✓）|
| **5** | `Sram_v` | 12288 × 16 SP | `v_buf`（使用 10240）| （idle, spare）|
| **6** | `Sram_qkm` | 1280 × 16 SP | `qkm_buf` / `zr_buf` 共用 | `cal_bbox.size_buf + off_buf`（使用 1024）|
| **7** | `Sram_c2w` | **1056 × 16 SP**（由 864 升上） | （idle, OFF）| `conv2` weight prefetch（實際使用 864，剩 192 不寫不讀）|

> **macro 統一升到 12288 × 16**：backbone 只用 10240（浪費 16%），但 head 的 sh1/sh2 剛好整除，sh1 = 2 × 12288 完美對齊。
> **SRAM #7 由 864 升到 1056**：864 低於 SHC-SPMBSRAM compiler 最小深度 1056（Mux 4 起跳），compiler 無法生產。升到 1056 後可直接編譯（詳見 §3.8）。

### 7 顆 SRAM 總容量

| # | 規格 | Compiler 設定 | 容量 |
|---|------|--------------|------|
| 1~5 | 12288 × 16 SP | SHC-SPMBSRAM Mux 16 | 5 × 24 KB = **120 KB** |
| 6 | 1280 × 16 SP | SHC-SPMBSRAM Mux 4 | **2.5 KB** |
| 7 | **1056 × 16 SP** | SHC-SPMBSRAM Mux 4 | **2.06 KB** |
| **合計** | **7 顆** | | **~ 124.6 KB** |

> 較原規劃多 ~0.4 KB（SRAM #7 由 864→1056，多 192 entries × 16-bit = 0.375 KB）。代價極小，換取「全部 macro 都能由 PDK compiler 直接生產」。

## 3.4 Phase-by-phase 時序檢查（確認單埠 SRAM 不衝突）

### Backbone 1 個 block 內

| FSM state | SRAM 1 | SRAM 2 | SRAM 3 (Q/AO) | SRAM 4 (K) | SRAM 5 (V) | SRAM 6 (QKM) |
|-----------|--------|--------|---------------|------------|------------|--------------|
| (care) LOAD_X → `Sram_x` | W | – | – | – | – | – |
| QKV (linear) | R (`Sram_x`) | – | W (`Sram_q`) | W | W | – |
| K_MEAN | – | – | – | R | – | W |
| QK_MEAN | – | – | R | – | – | R+W |
| KV | – | – | – | R | R | – |
| ATTN | – | – | R (`Sram_q`) | – | W (`Sram_v` ao) | R |
| PROJ | – | W | R | – | – | – |

→ **每顆 SRAM 任何拍只有讀 OR 寫**，單埠 macro 完全夠用。
→ block 切換時 SRAM 1/2 角色互換（ping-pong）。

### Head 4 個 stage

| Stage | SRAM 1 | SRAM 2 | SRAM 3 (sh1A) | SRAM 4 (sh1B) | SRAM 5 | SRAM 6 (bbox) | SRAM 7 (c2wgt) |
|-------|--------|--------|---------------|---------------|--------|---------------|----------------|
| FILL | W (x_buf) | – | – | – | – | – | – |
| CONV1 | R | – | W | W | – | – | – |
| CONV2 | – | W (sh2) | R | R | – | – | R |
| TAIL | – | R | – | – | – | W | – |
| BBOX | – | – | – | – | – | R | – |

→ 同樣**單埠 macro 不衝突**。
→ SRAM 5 在 head phase 完全閒置（可做 spare 或乾脆關電）。

## 3.5 工程取捨：放棄 Power Gating 細粒度

### 取捨對照

| 項目 | 影響 |
|------|------|
| **SRAM 顆數** | 從 ~14 顆 → **7 顆**（減少 50%）|
| **總 SRAM 容量** | ~ 235 KB → **~ 124 KB**（減少 47%）|
| **die area** | SRAM 部分縮小 ~50% |
| **leakage 靜態功耗** | 仍可改善（總 bit cell 變少），但**無法 layer-by-layer 變化** |
| **Power State Table** | **不能再寫 layer-by-layer ON/OFF**，論文章節需重新框架（見 §3.6）|
| **Control 複雜度** | 每顆共用 SRAM 入口加 2-to-1 mux + mode FSM |
| **APR floorplan** | SRAM 必須擺在 backbone & head 都繞得到處（通常擺中央）|
| **驗證成本** | Backbone mode、Head mode 各對拍一次 |

### 為什麼這樣取捨合理

1. **UAV tracking 推論模式**：一次跑完一張影像 → 整顆 idle → 等下一張。**inference 中 power gating 收益有限**（單張影像跑時間短）。
2. **真正 idle 時間在 chip-level**（兩張影像間），整顆 die 可以一起 shutdown，這時 SRAM 共用不共用都 OFF。
3. **die area 與成本**遠比 inference 階段的 leakage 重要。

## 3.6 論文章節（Chapter 3）應對策略

### 衝突點

[thesis_rule2.md](../../.cursor/rules/thesis_rule2.md) §3 明文要求 Chapter 3 必須有 **Power State Table**：

> *探討 Power Gating 時，必須以表格呈現硬體在執行每一層時，各個 Power domain (pgen1 ~ pgenX) 的 On/Off 狀態。*

捨棄 power gating 後，這張表會變成「**幾乎全 ON**」，沒有看點。

### 推薦策略：保留 Logic Power Gating + 新增 Memory Sharing Table

| 項目 | 做法 |
|------|------|
| **SRAM** | 全部共用、全 ON（接受面積優化）|
| **Logic（datapath / FSM）** | 仍做 power gating：backbone 跑時 head logic OFF，反之亦然 |
| **Chapter 3 §3.X** | **Power State Table** 改寫「邏輯 domain」的 ON/OFF |
| **Chapter 3 §3.Y**（新增）| **Memory Sharing Table** 展示 7 顆 SRAM 在 backbone/head 雙重用途 |

#### Power State Table 改寫範例

| 階段 | pgen_bb_logic | pgen_head_logic | SRAM_pool（共用）|
|------|:---:|:---:|:---:|
| Backbone Block 0~6 | **ON** | OFF | Always-ON |
| Head Conv1~bbox | OFF | **ON** | Always-ON |

#### 論文敘述範本（英文）

> *"To maximize SRAM area efficiency, the proposed architecture employs a unified SRAM pool shared across backbone and head phases. Power gating is applied at the logic domain level only, as shown in Table 3.X. The shared SRAM domain remains powered throughout inference but achieves lower total leakage through reduced macro count (~7 vs ~14 in a non-shared design). Memory sharing schedule is detailed in Table 3.Y."*

## 3.7 sglatrack_top 重構草圖


內容包含：

- 7 顆 `sram_sp` 行為模型實例（APR 時換成 PDK macro）
- backbone-side / head-side SRAM port 線宣告（各 6 組 quintet）
- `running_mode` 驅動的 mux 路由（每顆 SRAM 一段 4 行）
- `backbone_top` / `head_top` 實例化（含新 SRAM port 介面契約）
- FSM 三段式（state register / next-state / output + `running_mode` 切換）

### 重構依賴清單（檔頭已標註）

| 模組 | 必改項 |
|------|--------|
| `backbone_top.v` | 刪 `tok_buf / out_buf` reg；新增 6 組 SRAM port；下傳 SRAM 3~6 ports |
| `transformer_block.v` | **(Phase 3)** 刪模組內 `u_sram_x/tmp`，改 top pool port |
| `care_attention.v` | **(Phase 1 已)** inline macro；**(Phase 3)** 改 port、刪 `S_LOAD_X` |
| `mlp.v` | **(Phase 1 已)** 刪 `x_in_buf`，`norm_rd_*` 讀 parent tmp |
| `head_top.v` | 刪 `x_buf / sh1_buf / sh2_buf`；新增 6 組 port；**刪除 S_FILL**（資料已在 SRAM 1）；sh1 split：`addr[13]==0 → SRAM 3、==1 → SRAM 4`|
| `cal_bbox.v` | 刪 `size_buf / off_buf`；改用 SRAM 6（addr[9] 區分 size/off）|
| `conv.v` | conv2 instance 的 `wgt_buf` 改用 SRAM 7 |

## 3.8 PDK Compiler 對齊檢查（SHC-SPMBSRAM）

依據 PDK Table 2.4「SHC-SPMBSRAM Compiler Range Information」：

| Mux Option | Word Depth 範圍 | Word Width 範圍 |
|:----------:|----------------|----------------|
| 4 | 1056~2048, 2112~4096 (step 32) | 8 ~ 144 |
| 8 | 2112~4096, 4224~8192 (step 64) | 4 ~ 72 |
| 16 | 4224~8192, 8448~16384 (step 128) | 4 ~ 39 |

### 7 顆 SRAM 逐一驗證

| # | 原規劃 | Compiler 驗證 | **修正後規格** | 備註 |
|---|--------|--------------|---------------|------|
| 1~5 | 12288 × 16 | Mux 16：12288 = 8448 + 30×128 ✓<br>Width 16 ∈ [4, 39] ✓ | **12288 × 16** ✓ | 不用拆 |
| 6 | 1280 × 16 | Mux 4：1280 = 1056 + 7×32 ✓<br>Width 16 ∈ [8, 144] ✓ | **1280 × 16** ✓ | 不用拆 |
| 7 | **864 × 16** | ❌ 864 < 1056（Mux 4 最小值）<br>**任何 Mux 都拒絕生產** | **1056 × 16**（升上）| 多浪費 192 entries (18%) |

### SRAM #7 為什麼必須升到 1056

864 同時 **小於三個 Mux 選項的最小深度門檻**（1056 / 2112 / 4224），落在「沒人接的死角」。

**物理原因**：SRAM macro 的 peripheral（row decoder、sense amp、write driver）有固定面積開銷。當深度太小時，peripheral / bit cell 比例失衡，compiler 廠商乾脆設下限避免浪費。

**選擇方案 A：升到 1056 × 16，Mux 4 編譯**
- 多 0.4 KB 容量代價（可忽略）
- macro 規格標準、可直接由 SHC-SPMBSRAM compiler 產出
- conv2 weight prefetch 實際只用前 864 entries（addr 0~863），addr 864~1055 不寫不讀（X 狀態不影響功能）

**RTL 改動最小**：[conv.v](../lib/models/verilog2/conv.v) 內 `wpre_feat` 計數器與 weight ROM 對應位址維持 0..863，**SRAM #7 的 address port 寬度從 10 bit 加大到 11 bit**（ceil(log2(1056)) = 11），但 controller 只用低 10 bit 即可。

### 24576 為什麼必須拆 2 顆

24576 > 16384（Mux 16 最大深度上限），**沒有任何 Mux option 在 16-bit width 下能提供 24576 深度的單 macro**。

**拆法**：24576 = 12288 + 12288，對應 SRAM 3（sh1 part A）+ SRAM 4（sh1 part B）。

**Head 端 RTL 解碼**：
```verilog
wire        sh1_sel        = sh1_addr[13];        // 0 -> SRAM 3, 1 -> SRAM 4
wire [12:0] sh1_local_addr = sh1_addr[12:0];
```

### Compiler 對齊後總結

| 指標 | 數值 |
|------|------|
| SRAM macro 顆數 | **7 顆**（全部可由 SHC-SPMBSRAM 產出） |
| 總容量 | **~ 124.6 KB** |
| 最大單顆 macro | 12288 × 16 = 24 KB（Mux 16）|
| 最小單顆 macro | 1056 × 16 = 2.06 KB（Mux 4）|
| 規格相容性 | ✅ 100% 符合 SHC-SPMBSRAM Table 2.4 |

## 3.9 SHC-SPMBSRAM 編譯指令清單

**指令格式**：`Sram_<name> <depth> <width> <CM_mux> s`
- `<CM_mux>`：column mux 選項（4 / 8 / 16），由 depth 決定
- `s`：segment option（Small / SGE）

### Unified Pool — 7 條指令（Phase 3 一鍵複製）

```
# ============================================================================
# Verilog2 Unified Shared SRAM Pool - 7 SHC-SPMBSRAM macros (Phase 3 target)
# Format: Sram_<name> <depth> <width> <CM_mux> s
# All single-port (segment=s), Q8.8 16-bit data path
# ============================================================================
Sram_tok1   12288 16 16 s
Sram_tok2   12288 16 16 s
Sram_x      12288 16 16 s
Sram_q      12288 16 16 s
Sram_k      12288 16 16 s
Sram_v      12288 16 16 s
Sram_qkm    1280  16 4  s
Sram_c2w    1056  16 4  s
```

### `verilog_backbone2` Phase 1 額外 compiler 產物

Phase 1 **除上表 7 顆 pool 用 macro 外**，`care_attention` 須另編 **`Sram_x`**（與 `Sram_tok1` **不同 .v 檔**）：

```
Sram_x      12288 16 16 s
```

見 §1.0.1（單次 sim：7 種 `.v` 檔、51 個 instance）。

### 指令對應角色（Backbone + Head 共用對照，Phase 3）

| 指令 | Backbone phase 用途 | Head phase 用途 |
|------|--------------------|----------------|
| `Sram_tok1` | `tok_buf` ping（用 10240）| conv1 `x_buf`（用 8192）|
| `Sram_tok2` | `tok_buf` pong / `out_buf`（用 10240）| `sh2_buf`（用 12288 ✓）|
| `Sram_q` | `q_buf`（用 10240；`ao` 在 `Sram_v`）| `sh1_buf` lo（用 12288 ✓）|
| `Sram_k` | `k_buf`（用 10240）| `sh1_buf` hi（用 12288 ✓）|
| `Sram_v` | `v_buf`（用 10240）| idle / spare |
| `Sram_qkm` | `qkm_buf` / `zr_buf` 共用（用 1280）| `cal_bbox.size_buf+off_buf`（用 1024）|
| `Sram_c2w` | idle | conv2 weight prefetch（用 864）|

### CM_mux 選擇邏輯

各 macro 為何用該 Mux：

| 指令 | depth | 為何選此 Mux | 範圍驗證 |
|------|-------|-------------|---------|
| `Sram_tok1/2/q/k/v` | 12288 | **Mux 16**：12288 > 8192（Mux 8 上限），只能 Mux 16 | depth ∈ [4224, 16384] ✓ |
| `Sram_qkm` | 1280 | **Mux 4**：1280 < 2112（Mux 8 下限），只能 Mux 4 | depth ∈ [1056, 4096] ✓ |
| `Sram_c2w` | 1056 | **Mux 4**：Mux 4 最小深度（864 升上）| depth ∈ [1056, 4096] ✓ |

> 註：Mux 16 的 width 上限為 39，剛好涵蓋 16 bit（Q8.8）；Mux 4/8 的 width 上限更寬（72/144），對 16-bit 都沒問題。

---

# 第四部分：SRAM port 設計通用注意事項

不論 backbone 或 head，每顆 SRAM macro 在 RTL 內須遵守 [verilog_rule.mdc §7](../../.cursor/rules/verilog_rule.mdc)：

1. **讀取契約三行註解** 寫在模組頭：
   - 位址何時有效（`posedge clk`）
   - 讀出資料何時有效（位址後第 1 拍）
   - macro `CLK` 是 `clk` 還是 `~clk`
2. **禁止同拍 write & read 同一位址**（除非 macro 規格明訂 write-through）。
3. **FSM 顯式區分** `ADDR` 與 `USE` 狀態，避免「送位址同拍就送進 MAC」的 off-by-one。
4. **Producer 邊界**：若資料來自 **`layer_norm`**（`y_sat_o` comb、`y_valid` reg 晚一拍），consumer **須 posedge latch 再寫 SRAM**，不可用 `y_valid` 組合寫入；見 [verilog_rule.mdc §7.7.3.1](../../.cursor/rules/verilog_rule.mdc)。
5. **`stream` / `stream_r` 預取慣例**（§7.2）：weight ROM 已驗證可行；token / feature SRAM 也應沿用此模式。
6. **golden 對拍**：每顆 SRAM 對應的 activation stream 須在模組頭標出 golden 檔，例如：
   ```
   // Golden: vit_care_relu6_numpy_trunk_dim32_out/Activation/backbone_after_norm_backbone_out_bi.txt
   ```

---

# 第五部分：實作硬規範（v2 — 改寫前必讀）

> 本節為**改寫 backbone2 / head2 RTL 之前的硬性規範**；違反這些規定的 patch 不會被收。Codex 複查也會依本節判斷。

## 5.1 七條硬規範

| # | 規範 | 動機 |
|---|------|------|
| **1** | **直接 inline 實例化 PDK macro**（依 §3.9 **compiler 原始指令** 產生的 `Sram_*.v`），**禁止另寫 `*_wrap.v` wrapper** | tie-off／`CLK(~clk)` 寫在 **使用模組內**（見 §5.2）；VCS 只編譯 `Sram_*.v` + RTL，不編譯 `*_wrap.v` |
| **2** | **禁止新增大型 2D reg**（例如 `reg [15:0] shadow [0:10239]`）作 shadow buffer | SRAM 化的本意就是消除大 reg；新增 shadow 等於白做。只允許 **scalar reg**（單個 16-bit 或 1-bit phase）做 1-cycle latency 對齊 |
| **3** | **最小化 RTL 改動**；非必要不重構 | 易讀、易複查、易回退 |
| **4** | 改完後 **Codex 會接手複查** | 命名與註解須一致；deviation 要在模組頭顯式說明 |
| **5** | **本機不能跑 VCS**；除錯只能靠 `$display` 截圖回傳 | 加大量 `` `ifdef `` -gated `$display`；列印必須是 ASCII 英文（[verilog_rule.mdc §10.1](../../.cursor/rules/verilog_rule.mdc)） |
| **6** | **Bit-accurate 對拍** `Activation/*_bi.txt` golden | 任何 deviation（例如 ao_buf 從 SRAM3 移到 SRAM5）須在模組頭與本文件 §3.3 註明 |
| **7** | macro port 完整列表 = **使用者 datasheet 截圖**（注意 `BWEBM` 不是 `BWEM`） | 與 [reference/rongxuan_verilog/CFVT.v](../../reference/rongxuan_verilog/CFVT.v) 範例的 `BWEM` 拼字略有差異，**以 macro 檔為準** |

## 5.2 SRAM macro 直接 inline 範本

**Phase 1（現況）**：macro 可 inline 於 **`backbone_top` / `transformer_block` / `care_attention`**（見 §1.0.1）；**Phase 3 (目標)**：收斂至 `sglatrack_top` 7-pool + port 下傳。

每顆 macro 必須完整列 PDK port（**禁止 `*_wrap.v`**），**禁止簡寫**：

```verilog
Sram_tok1 u_sram1 (
    .SLP   (1'b0),   .DSLP  (1'b0),   .SD    (1'b0),
    .PUDELAY(),
    .CLK   (~clk),   .CEB   (s1_ceb), .WEB   (s1_web),
    .BIST  (1'b0),   .CEBM  (),       .WEBM  (),
    .A     (s1_addr),.D     (s1_din),
    .BWEB  (16'b0),
    .AM    (),       .DM    (),       .BWEBM (16'b0),
    .RTSEL (2'b01),  .WTSEL (2'b00),
    .Q     (s1_q)
);
```

對應 [verilog_rule.mdc §7.7](../../.cursor/rules/verilog_rule.mdc) 的讀取契約：
- `posedge clk` (T) 送 `A`、`D`、`WEB`、`CEB=0`
- `posedge clk` (T+1) `Q` 有效（macro 內部 CLK 在 `negedge T` 採樣）
- **禁止同一 posedge 對單埠 macro 同時 read + write 同一 macro**

## 5.3 設計決策：Phase 1（現況）vs Phase 3（目標）

| 決策 | **Phase 1 已實作** | **Phase 3 (目標)** |
|------|-------------------|-------------------|
| **macro 位置** | **分散 inline**：`backbone_top`（tok1/tok2）+ `transformer_block`（x/tmp）+ `care_attention`（x,q,k,v,qkm） | **7 顆** 在 `sglatrack_top` + port 下傳 |
| **norm1 / norm2 暫存** | `transformer_block.u_sram_tmp`；mlp FC1 **port 讀** tmp（無本地 10240） | 共用 pool 同址，刪重複拷貝 |
| **attn norm1 輸入** | **`S_LOAD_X` → `Sram_x`**（與 `Sram_q` 分離） | **(目標)** 刪 feed + 刪 `S_LOAD_X`，`S_QKV` 直接讀 pool |
| **in-place RW** | **2-phase + scalar shadow reg** | 同左 |

### 5.3.1 為何 Phase 3 要「跳過 S_LOAD_X」**(目標，尚未全套實作於 care_attention)**

原 streaming 流程：
1. `transformer_block.S_NORM1` 寫入 `tmp_buf`
2. `S_ATTN_FEED` 從 `tmp_buf` 串流到 `u_attn.x_i`
3. `care_attention.S_LOAD_X` 再寫入 `x_in_buf` / `Sram_x`

若步驟 1–3 共用**同一顆**單埠 SRAM：**讀 feed 與寫 LOAD_X 同拍衝突**。

**Phase 1 作法**：保留步驟 2–3 語意，但 **norm1 快照用獨立 `Sram_x`**，**q 寫 `Sram_q`**（flatten 不同，不可合併，§1.0.3）。

**Phase 3 作法**：步驟 2–3 **皆刪**；norm1 只寫 pool 一次；`S_IDLE → S_QKV` 讀 pool；並約束 **`S_ATTN_WAIT` 寫 tmp 不得與 QKV 讀 x 重疊**（或改用第二顆 scratch）。

**mlp（Phase 1 已做）**：已刪 `S_LOAD_X`；`S_NORM2` 寫 `u_sram_tmp` 後，FC1 以 `norm_rd_en` 2-phase 讀取（見 §1.0.2）。

**(目標)** care_attention `S_IDLE` 範例：

```verilog
S_IDLE: next_state = start ?
`ifdef USE_REG_BUF
                       S_LOAD_X
`else
                       S_QKV          // Phase 3 only: data already in shared pool
`endif
                     : S_IDLE;
```

### 5.3.2 2-phase + scalar shadow reg 範本（以 S_SPLIT 為例）

```verilog
// 新增 scalar regs（合法，因為是單一 16-bit 不是大 2D reg）
reg signed [15:0] sp_q_r;     // 1-cycle shadow for s3 read
reg signed [15:0] sp_k_r;     // 1-cycle shadow for s4 read
reg               sp_phase;   // 0 = ADDR, 1 = USE

// FSM 內：
S_SPLIT: begin
    if (sp_phase == 1'b0) begin
        sp_phase <= 1'b1;      // 下一拍轉 USE
    end else begin
        sp_phase <= 1'b0;
        if (sp_ptr < HD_ELEMS-1) sp_ptr <= sp_ptr + 1;
    end
end

// SRAM mux always @(*) block：
S_SPLIT: begin
    if (sp_phase == 1'b0) begin
        // ADDR phase: drive read addr
        s3_ceb_o  = 1'b0; s3_web_o = 1'b1; s3_addr_o = sp_ptr;
        s4_ceb_o  = 1'b0; s4_web_o = 1'b1; s4_addr_o = sp_ptr;
    end else begin
        // USE phase: combinational compute and write back
        s3_ceb_o  = 1'b0; s3_web_o = 1'b0; s3_addr_o = sp_ptr;
        s3_din_o  = relu6_q88(rnd_shr8_q88($signed(s3_q_i) * S_Q88));
        s4_ceb_o  = 1'b0; s4_web_o = 1'b0; s4_addr_o = sp_ptr;
        s4_din_o  = relu6_q88(rnd_shr8_q88($signed(s4_q_i) * S_Q88));
    end
end
```

> 注意：**規範 #2 允許 `sp_q_r`、`sp_k_r` 這種單一 16-bit scalar reg**，但若要省可直接用 `s3_q_i` 組合進 `s3_din_o`（前提：macro 在 write 拍 Q 維持上拍 read data，多數 PDK 是這樣，但須以 datasheet 為準）。

## 5.4 Phase 1 單模組 macro 角色（`verilog_backbone2` 現況）

| Macro 檔 | 規格 | 主要角色 |
|----------|------|----------|
| `Sram_tok2` | 12288×16 | `backbone_top` inter-block token |
| `Sram_tok1` | 12288×16 | `backbone_top` norm 輸出；**或** `transformer_block` 的 `u_sram_x` / `u_sram_tmp` |
| **`Sram_x`** | 12288×16 | **`care_attention` norm1 快照**（`S_LOAD_X` 寫、`S_QKV` 讀）；**不可與 `Sram_q` 共用** |
| `Sram_q` | 12288×16 | **`care_attention` q only**（head-major；QKV capture / SPLIT / ATTN 讀） |
| `Sram_k` | 12288×16 | `care_attention.k_buf` |
| `Sram_v` | 12288×16 | `v_buf`（S_KV）+ **`ao_buf`（S_ATTN 寫、S_PROJ 讀）⚠ deviation** |
| `Sram_qkm` | 1280×16 | `qkm_buf` + `zr_buf`（時間多工） |

**⚠ `ao_buf` 在 `Sram_v`（非 `Sram_q`）**：`S_ATTN` 需同時 read q（`Sram_q`）與 write ao；單埠 `Sram_q` 無法兼任。模組頭須註記。

**⚠ `Sram_x` 與 `Sram_q` 不可合併**：flatten alias（§1.0.3），非單純單埠衝突。

## 5.4b Phase 3 Unified Pool 對應表（與 §3.3 一致，**(目標)**）

| Macro | 規格 | Backbone phase | Head phase |
|-------|------|----------------|------------|
| `Sram_tok1` | 12288×16 | `tok_buf` ping / `out_buf` | `x_buf`（8192） |
| `Sram_tok2` | 12288×16 | `tok_buf` pong | `sh2_buf` |
| `Sram_q` | 12288×16 | `q` / ~~ao~~ | **`sh1_buf` lo** |
| `Sram_k` | 12288×16 | `k` | **`sh1_buf` hi** |
| `Sram_v` | 12288×16 | `v` + `ao` | spare |
| `Sram_qkm` | 1280×16 | qkm/zr | bbox maps |
| `Sram_c2w` | 1056×16 | idle | conv2 weights |

> Phase 3 表內 **不再** 把 `x_in` 與 `q` 塞進同一顆 `Sram_q`；norm/x 與 head-major q 須 **分址規劃或分時**（見 §1.0.3）。

## 5.5 除錯流程（無本機 sim，僅 `$display`）

1. **使用者於另一工作站 compile**（範本見 [TEST_backbone.v](../lib/models/verilog_backbone2/TEST_backbone.v) 檔頭註解）
2. **使用者截圖 simv.log 回傳**
3. **逐 stage 對拍順序**（依 [numpy-trunk-to-verilog.mdc §16](../../.cursor/rules/numpy-trunk-to-verilog.mdc)）：
   - 若 `[BB_FAIL]` 出現：看 `first_bad_idx`，加 `+define+DUMP_FSM_TRACE` 看哪個 state 出錯
   - 若 block 邊界異常：加 `+define+DUMP_BLOCK_BOUND` 看 ping-pong sel 與 `tok_wr_ptr`
   - 若懷疑 SRAM 時序：加 `+define+DUMP_SRAM_S1`（或其他 sN）印 `CEB/WEB/A/D/Q`
4. **Regression baseline**：
   ```bash
   vcs ... +define+USE_REG_BUF ...     # 跑 reg 版本確認原版仍 work
   ```
   - 若 reg 版本通過、SRAM 版本失敗 → SRAM 改寫 bug
   - 若 reg 版本也失敗 → 是其他改動的 bug（非 SRAM 引入）

## 5.6 ROM 慣例對比（提醒，不要混淆）

| 元件 | CLK 接法 | 寫入時機 | 讀出時機 | 慣例參考 |
|------|----------|----------|----------|----------|
| 既有 ROM（如 `rom_backbone_blocks_0_6_norm1_weight`） | `~clk` | N/A（唯讀）| 位址 `posedge T` 送，`Q` 於 `posedge T+1` 有效 | [backbone_top.v:304](../lib/models/verilog_backbone2/backbone_top.v#L304) |
| 新增 SRAM（如 `Sram_tok1`） | `~clk` | `posedge T` 送 `A/D/WEB=0/CEB=0`，macro 於 `negedge T` 採樣寫入 | 同 ROM | 本節 §5.2 範本 |

**統一 `~clk` 慣例**：避免 SRAM 與 ROM 不同邊沿造成 timing skew。Compile 後若發現對拍偏差正好 1 拍，多半是邊沿寫反。

## 5.7 改 SRAM 前必查：`reg` 同拍讀寫與時序（`verilog_backbone2`）

> 在把大 `reg` 換成 **單埠（1P）macro** 之前，先對照下表與 [verilog_rule.mdc §7.7](../../.cursor/rules/verilog_rule.mdc)。**reg 允許** Verilog NBA 的「同址讀舊值、寫新值」；**1P SRAM 不允許**同拍、同 macro、同址（或需拆 **ADDR／USE** 兩拍）。

### A. 同址、同拍 in-place（換 SRAM **必**拆 FSM）

| 模組 | 狀態／行為 | 訊號 | 處理 |
|------|------------|------|------|
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `S_SPLIT` | `q_buf[sp_ptr]`、`k_buf[sp_ptr]` 同拍讀舊寫新 | **ADDR 拍讀 → 運算 → USE 拍寫**（§5.3.2） |
| [layer_norm.v](../lib/models/verilog_backbone2/layer_norm.v) | `S_CENTER` | `feat_buf[addr]` combo 讀 + NBA 寫 `centered` | 同上；深度 32 可暫留 reg |

### B. 同陣列、同拍、不同 index（1P 仍只能一種操作）

| 模組 | 行為 | 注意 |
|------|------|------|
| [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v) | `S_RES1`/`S_RES2`：`x_buf[res_rp]` 讀 + `x_buf[res_wp]` 寫 | `res_wp` 落後 `res_rp`（`u_res` 1-cycle）；改 SRAM 須確認無同拍對 **同一顆 1P** 又讀又寫 |
| [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v) | replay：`tok_buf` 讀（餵 `u_tb`）+ `tb_y_valid` 寫 | 讀寫指標通常錯開；若同拍衝突 **寫優先** 或 **stall 讀**，並補 **1-cycle read latency**（`tb_xv_replay_r`） |

### C. 分狀態讀寫（較易接 SRAM，仍要注意 read latency）

| 緩衝 | 寫入時機 | 讀取時機 |
|------|----------|----------|
| `backbone_top` `out_buf` / **`Sram_tok1`** | 見 **§5.7.1**（**非** `bn_y_valid` 組合寫） | `S_OUT` 串流（SRAM 須 **ADDR/USE 兩拍**，見 §7.7.2） |
| `mlp` norm2（parent `u_sram_tmp`） | `S_NORM2` latch 寫 tmp | `S_FC1`：`mlp_norm_rd_en` ADDR → 下一拍 `norm_x`（**已刪** 本地 `x_in_buf`） |
| `care_attention` `Sram_x` | `S_LOAD_X` | `S_QKV` 2-phase 讀（與 `Sram_q` 寫 q 並行不同 macro） |
| `care_attention` `qkm_buf` / `zr_buf` | `S_QK_MEAN` / `S_Z_RECIP` | `S_Z_RECIP` / `S_ATTN`（不同拍） |

### 5.7.1 `backbone_top`：norm → `Sram_tok1`（`out_buf`）capture 契約（**必守**）

> 依據：`layer_norm.v` **`out_beat_o`**／port 註解；[verilog_rule.mdc §7.7.3.1](../../.cursor/rules/verilog_rule.mdc)。  
> Golden：`backbone_after_norm_backbone_out_bi.txt`（flatten：`flat = tok×EMBED_DIM + feat`，row-major）。

**寫入（norm 輸出 → buffer）**

| 拍 | 條件 | 動作 |
|----|------|------|
| **`posedge T`** | `state==S_BACKBONE_NORM` && **`bn_out_beat`**（=`u_bn.out_beat_o`） | 鎖 **`s1_wr_flat_lat <= bn_cap_flat`**、**`s1_wr_din_lat <= bn_y_sat`** |
| **`posedge T+1`** | **`s1_wr_do`** 為 1 | **`USE_REG_BUF`**：`out_buf[lat]<=din`；**`USE_SRAM_BUF`**：comb 驅動 **`bt_s1_*`** 寫 macro |

- **禁止** `if (bn_y_valid)` 組合寫 S1／`out_buf`（**off-by-one**，漏 **`flat=0`**）。
- **`USE_REG_BUF` 與 `USE_SRAM_BUF` 共用同一組 latch**（`s1_wr_flat_lat` / `s1_wr_din_lat` / `s1_wr_do`），避免兩路徑 handshake 不一致。

**讀取（`S_OUT` → 模組 `y_o`）** — 與寫入契約 **分開**；寫對了仍可能因 read latency 失敗：

| 拍 | 動作 |
|----|------|
| **`out_rd_phase==0`** | 送 **`A=out_rd_addr`**，`WEB=1`（read） |
| **`out_rd_phase==1`** | **`y_valid=1`**，消耗 **`s1_q`**（macro `CLK(~clk)` → **T+1** 有效） |

**與 S2 block 輸出寫入的對照**（勿混用 trigger）：

| 路徑 | 寫入 trigger | 原因 |
|------|--------------|------|
| **`tb_y_valid` → `Sram_tok2`** | 組合 `if (tb_y_valid)` + `tok_wr_ptr` | stream：`valid`／`data`／ptr **同拍對齊**，ptr **NBA** 遞增 |
| **`layer_norm` → `Sram_tok1`** | **`out_beat_o` posedge latch + `s1_wr_do`** | **`y_valid` reg 晚一拍**，不可用 `bn_y_valid` 組合寫 |

**Testbench 對拍注意**

- norm 寫入檢查須比 **`GOLD_BB[s1_wr_flat_lat]`**（或 **`GOLD_BB[bn_cap_flat]`** 在 **capture 拍**），**勿** 僅比 **`GOLD_BB[bn_feat_addr]`**（off-by-one 時仍可能誤顯示 PASS）。
- **`USE_REG_BUF` PASS** 不能證明 capture 正確；須確認 latch 契約與 **`flat=0`** 有寫入 log。

### D. 改 SRAM 時通用時序檢查（與第四部分呼應）

1. **Read latency**：`posedge T` 送 `A`、`CEB=0`、`WEB=1` → **`posedge T+1` 才使用 `Q`**（macro `CLK(~clk)` 時與 ROM 相同）。勿把「送址拍」的 `Q` 直接送 MAC。
2. **禁止**：1P macro 同拍 `WEB=0`（寫）又依賴同拍 `Q` 做運算（除非 datasheet 明訂 write-through）。
3. **FSM**：串流餵料改為 **`x_valid` 比讀取位址晚 1 拍**（或 2-phase `out_rd_phase`），並在模組頭寫清契約三行。
4. **Producer latch**：`layer_norm` → SRAM 須 **§5.7.1 / §7.7.3.1**；勿與 **`tb_y_valid` stream 寫** 混用同一 `if (valid)` 模式。
5. **允許的額外暫存**：僅 **scalar** `reg`（如 `tb_xv_replay_r`、`out_rd_phase`、`s1_wr_do`、`s1_wr_flat_lat`），**禁止**用 `reg [15:0] shadow [0:10239]` 取代 SRAM（§5.1 #2）。
6. **對拍**：仍以 `Activation/*_bi.txt` 為準；SRAM 版用 `+define+USE_SRAM_BUF`，reg 版用 `+define+USE_REG_BUF`（見上文「編譯開關」）。TB 比 **`golden[flat_index]`**，勿僅比 **`golden[feat_addr]`**（見 §5.7.1）。

---

# 第六部分：後續工作順序

依使用者工作流（**先 verilog_backbone2、verilog_head2，再同步 verilog2**）：

## Phase 1：verilog_backbone2 改 SRAM（分模組除錯）

| 步驟 | 內容 | 狀態 |
|------|------|------|
| 1 | [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v)：`tok_buf`/`out_buf` → `Sram_tok2`/`Sram_tok1` | **已完成** |
| 2 | [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v)：`x_buf`/`tmp_buf` → 模組內 `u_sram_x`/`u_sram_tmp` | **已完成** |
| 3 | [care_attention.v](../lib/models/verilog_backbone2/care_attention.v)：`q/k/v/ao/qkm/zr` → inline `Sram_q/k/v/qkm` + **`Sram_x`** | **已完成** |
| 4 | [mlp.v](../lib/models/verilog_backbone2/mlp.v)：刪 `x_in_buf`；FC1 讀 parent `norm_rd_*` | **已完成** |
| 5 | `+define+USE_SRAM_BUF` + §1.0 所列 macro 檔對拍 `Activation/*_bi.txt` | **進行中／使用者 VCS** |
| 6 | 保留 `+define+USE_REG_BUF` regression | **必守** |
| **(Phase 3)** | top 7-pool port、刪 `care_attention.S_LOAD_X`、收斂 macro | **未做** |

編譯須含 **`Sram_x.v`**（與 `Sram_tok1.v` 不同檔）。缺 macro 時可暫用 [sglatrack_top_shared_sram_draft.v](sglatrack_top_shared_sram_draft.v) 行為模型，**不可替代 PDK 驗證**。

## Phase 2：verilog_head2 改 SRAM port（分模組除錯）

1. [head_top.v](../lib/models/verilog_head2/head_top.v)：**拆掉** `x_buf / sh1_buf / sh2_buf` reg 陣列，改 **3+1 組 SRAM port**（`sh1` 拆 2×12288；見第二部分 §2.1 A、§2.3）
2. [conv.v](../lib/models/verilog_head2/conv.v) conv2 `wgt_buf` 改 SRAM port（或先用 1056-deep 行為模型對齊 §3.8）
3. [cal_bbox.v](../lib/models/verilog_head2/cal_bbox.v) `size_buf / off_buf` 合併為 1 顆 SRAM（`addr[9]` 區分 size/off）
4. [TEST_head.v](../lib/models/verilog_head2/TEST_head.v) 維持 token 串流 + bbox golden；中間 feature 對拍改讀 `Activation/box_head_*_bi.txt` 對 SRAM 輸出
5. 通過後將 `head_top.v` 同步至 [verilog2/head_top.v](../lib/models/verilog2/head_top.v)

## Phase 3：同步到 verilog2 + 加 sglatrack_top mux

1. 把 Phase 1、2 改好的 RTL 同步到 [verilog2/](../lib/models/verilog2/)
2. 用 [sglatrack_top_shared_sram_draft.v](sglatrack_top_shared_sram_draft.v) 草圖替換現有 [sglatrack_top.v](../lib/models/verilog2/sglatrack_top.v)
3. 重新整合 7 顆 SRAM + mux + `running_mode` FSM
4. 對端到端 TEST.v 通過對拍

## Phase 4：APR 前置作業

1. 把行為模型 `sram_sp` 替換成 PDK SRAM compiler 產出的真實 macro
2. 確認所有 SRAM 與 ROM 的 `CLK` 邊沿一致性（本專案 weight ROM 多為 `CLK(~clk)`）
3. 規劃 logic-only power gating domain（呼應論文 Power State Table）
4. 撰寫 Memory Sharing Table（呼應論文 §3.Y 新增章節）

---

> 若需要進一步輸出 **backbone_top.v / head_top.v 的 SRAM port 重構 patch、Memory Sharing Table Markdown、論文 §3 章節範本**，後續再請指示。

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
