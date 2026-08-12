## python-to-rtl-plan

> - Python 到 RTL 轉換計劃過程中只實作 **test / inference** 流程，不處理任何 train 會用到的邏輯。


# Python 到 RTL 轉換計劃（CARE / test-only / post-embedding）

## 專案邊界

- Python 到 RTL 轉換計劃過程中只實作 **test / inference** 流程，不處理任何 train 會用到的邏輯。
- RTL 只處理 **一個 template + 一個 search** 的情況，不考慮 batch 與多 template / 多 search。
- RTL **不實作** `patch_embed` 與 `position embedding`；這兩部分保留在 Python / PyTorch 端。
- RTL 的輸入是 **已完成 patch embedding 與 position embedding 後的特徵 / token**。
- RTL 的輸入形式固定為 **分開的 template token 與 search token**，不可將兩者在 Python 端先合成單一 `320 token` 再當成唯一 RTL 輸入。
- RTL 的目標是從 post-embedding 特徵開始，跑完 **backbone + head + bbox 生成** 的 test 路徑。
- 若與 Python test 對齊到最終 bbox，需納入 `score_map / size_map / offset_map / cal_bbox` 等 test-time head 邏輯。

## 固定點格式規則

- 本專案 Python check、binary Python check、參數文字檔與 RTL 的**預設固定點格式一律使用 `Q8.8`**。
- `Q8.8` 定義：
  - signed fixed-point
  - 1 個 sign bit
  - 8 個整數位
  - 8 個小數位
  - 總位寬預設視為 16-bit signed
- 若某個 stage 暫存器、累加器、乘法輸出需要更寬位寬，必須另外明確標註，但最終對外 dump / 對拍格式仍優先回到 `Q8.8`。
- 沒有特別聲明時：
  - activation 使用 `Q8.8`
  - weight 使用 `Q8.8`
  - bias 使用 `Q8.8`
  - intermediate dump 使用 `Q8.8`
- 若未來要改成其他格式，例如 `Q4.12`、`Q16.8`，必須先同步更新：
  - Python check
  - binary Python check
  - `.txt` / binary `.txt` 轉檔規則
  - RTL module 介面與 testbench

## 固定研究主線

1. 使用 `vit_CARE_relu6.py` 訓練浮點模型。
2. 使用 `vit_CARE_relu6.py` 訓練出的 checkpoint，載入到 `vit_CARE_relu6_fixed.py` 做 test / 量化驗證。
3. 將該 checkpoint `.pth.tar` 轉成各個 parameter 的 `.npy`，給 Python check 使用。
4. 撰寫 Python check（讀 `.npy`），用 hardware-friendly 方式重寫 **backbone + head + bbox** 的 test 流程。
5. 將 `.npy` 轉成 `.txt` / binary text，給 RTL 讀取。
6. 撰寫 binary 版 Python check，完全模擬未來 Verilog 的 bit-width、截斷、捨入、除法近似與控制流程。
7. 將 binary Python check 對應轉成 Verilog RTL。

## 參考檔案與路徑

- 浮點訓練 backbone：`python/lib/models/sglatrack/vit_CARE_relu6.py`
- 定點測試 backbone：`python/lib/models/sglatrack/vit_CARE_relu6_fixed.py`
- 定點 test config：`python/experiments/sglatrack/vit_coco_uav123_care_relu6_fixed.yaml`
- checkpoint 匯出腳本：`python/tracking/export_checkpoint_npy.py`
- Python check 參考：`reference/rongxuan/05_PythonCheck/python_check.py`
- Verilog 前 binary Python check 參考：`reference/rongxuan/06_GetBinary/python_check_verilog.py`

## checkpoint 路徑規則

- `vit_CARE_relu6.py` 訓練出的 checkpoint 預設路徑為：
  - `python/output/checkpoints/train/sglatrack/vit_coco_uav123_care_relu6/sglatrack_ep0050.pth.tar`
- 之所以優先使用 `ep0050`，是因為 config 的 `TEST.EPOCH = 50`，test 預設即會載入第 50 epoch。
- 若之後改 config 名稱或 epoch，需同步更新：
  - 訓練 checkpoint 路徑
  - 匯出 NPY 的來源 checkpoint
  - Python check / binary check / RTL testbench 使用的 golden 版本

## patch embedding / position embedding 規則

- `patch_embed` 與 `pos_embed` 不進 RTL。
- Python 端必須先完成：
  - patch embedding
  - 加上 positional embedding
  - 保留 template token 與 search token 分開的輸出
- embedding 前段的 NPY dump 檔名固定為：
  - `template_after_patch_embed_out.npy`
  - `search_after_patch_embed_out.npy`
  - `template_pos_embed.npy`
  - `search_pos_embed.npy`
  - `template_after_pos_add_out.npy`
  - `search_after_pos_add_out.npy`
- 上述檔名定義如下：
  - `template_after_patch_embed_out.npy`：`template = patch_embed(template)` 之後的輸出
  - `search_after_patch_embed_out.npy`：`search = patch_embed(search)` 之後的輸出
  - `template_pos_embed.npy`：`self.pos_embed_z`
  - `search_pos_embed.npy`：`self.pos_embed_x`
  - `template_after_pos_add_out.npy`：`patch_embed(template) + pos_embed_z`
  - `search_after_pos_add_out.npy`：`patch_embed(search) + pos_embed_x`
- 硬體驗證的 golden input 應該是 **post-embedding tensor**，不是原始 RGB image。
- Python 端可做 `patch_embed(template) + pos_embed_z` 與 `patch_embed(search) + pos_embed_x`，但 **`combine_tokens` 必須保留在 RTL 端**。
- RTL 輸入固定定義為：
  - `template_post_embed`：shape `(1, 64, 768)`
  - `search_post_embed`：shape `(1, 256, 768)`
- 上述兩個 tensor 才是 RTL 測試與對拍的第一級 golden input；`320 token merged tensor` 只能作為 Python / RTL 中間節點 dump，不可取代 RTL 正式輸入介面。
- 若匯出 parameter 只為 RTL 使用，可排除：
  - `patch_embed.*`
  - `pos_embed*`
  - `cls_token`
  - 任何只屬於 embedding 前段的權重
- 若匯出 parameter 也要保留完整 Python 對照，可額外保存完整 checkpoint 對應 NPY，但 RTL 實際使用時只讀 post-embedding 之後需要的參數。

## test-only 流程規則

- 僅分析與實作 test path。
- 不閱讀、不修改、不 RTL 化以下 train-only 項目：
  - dataloader
  - data augmentation
  - loss function
  - optimizer
  - backward
  - scheduler
  - AMP
  - candidate elimination warm-up / train-only control
- Python / RTL 對齊時，優先以 `forward_test` 與 tracker `track()` 實際推論路徑為準。

## Golden 版本規則

- 本專案 golden 驗證採用 **軟體先產生 template/search post-embedding token，RTL 只驗證第二幀 model path** 的策略。
- 第一幀在 Python / PyTorch 端的用途是：
  - 讀入 `frame1`
  - 使用 `init_bbox` 建立 template crop
  - 完成 preprocess、`patch_embed`、`pos_embed_z`
  - 產生 `template_post_embed`
- 第二幀在 Python / PyTorch 端的用途是：
  - 讀入 `frame2`
  - 依第一幀初始化後的 state 建立 search crop
  - 完成 preprocess、`patch_embed`、`pos_embed_x`
  - 產生 `search_post_embed`
- RTL 的第一版不負責：
  - 第一幀初始化
  - image crop
  - preprocess / normalize
  - patch embedding
  - position embedding
- RTL 的第一版只負責從：
  - `template_post_embed`
  - `search_post_embed`
  開始，完成 `combine_tokens -> backbone -> head -> bbox`。
- 若日後要做 system-level RTL，才再把第一幀初始化、第二幀 search crop 與 tracker 後處理往前後延伸；第一版 golden 規則不預設納入。
- golden 版本至少要保存：
  - 第一幀的 `template_post_embed`
  - 第二幀的 `search_post_embed`
  - 第二幀的 `merged_tokens`
  - 第二幀的 block-level outputs
  - 第二幀的 `backbone_out`
  - 第二幀的 `score_map`
  - 第二幀的 `size_map`
  - 第二幀的 `offset_map`
  - 第二幀的 `pred_boxes`
  - 若有納入 tracker 後處理，再保存 `response_after_window`、`bbox_after_cal_bbox`、`final_bbox`
- 除非特別說明，本專案提到的「golden」皆預設指：
  - `frame1` 產生 template
  - `frame2` 產生 search
  - RTL 對拍第二幀推論結果

## RTL 實作範圍

- 必做：
  - post-embedding 後的 backbone blocks
  - attention
  - MLP
  - normalization
  - head
  - bbox 生成相關 test 路徑
- 視最終 test 對齊需求決定是否納入：
  - Hann window
  - `cal_bbox`
  - mean box reduction
  - `map_box_back`
  - `clip_box`
- 每次討論「是否完成 RTL 化」時，必須先明確說明目前邊界停在哪一層。

## Step 1：浮點訓練規則

- 訓練模型固定為 `vit_CARE_relu6.py`。
- 浮點訓練的目的不是直接上板，而是取得：
  - 可接受精度的 checkpoint
  - 後續 fixed-point test 與 RTL golden 的來源權重
- 不在這個階段修改 train pipeline 去配合 RTL；train 只負責產生 checkpoint。

## Step 2：固定使用浮點 checkpoint 驗 fixed 模型

- `vit_CARE_relu6_fixed.py` 不重新訓練，先以 `vit_CARE_relu6.py` 的 checkpoint 做 test。
- 這一步的目的是驗證：
  - state_dict 相容
  - `Q8.8 fixed-point` 截斷後精度是否仍可接受
  - 後續 Python check / RTL 是否值得繼續
- 若 fixed test 已經無法接受，優先回頭調整 Python fixed-point 策略，不要直接進 RTL。

## Step 3：checkpoint 轉 NPY 規則

- 使用 `python/tracking/export_checkpoint_npy.py` 匯出 `.npy`。
- checkpoint 來源固定優先使用：
  - `python/output/checkpoints/train/sglatrack/vit_coco_uav123_care_relu6/sglatrack_ep0050.pth.tar`
- 匯出目的分兩種：
  - 完整保存用：匯出完整 state_dict
  - RTL 用：只匯出 post-embedding 之後真正需要的參數
- 每次匯出都要保留：
  - 原始 checkpoint 路徑
  - config 名稱
  - 匯出時間
  - manifest / key 對照表

## Step 4：Python check（讀 NPY）規則

- Python check 必須以 `.npy` 為唯一權重來源，不可暗中直接讀 PyTorch model 權重。
- Python check 的輸入不是 raw image，而是 post-embedding tensor。
- Python check 的正式輸入介面必須與 RTL 對齊為兩路：
  - `template_post_embed`
  - `search_post_embed`
- Python check 不可只接受單一 `merged_tokens` 作為唯一正式輸入；若有 `merged_tokens`，僅能作為 debug / dump 中間節點。
- Python check 必須用 hardware-friendly 方式重寫：
  - backbone
  - head
  - bbox 生成流程
- Python check 允許先保留浮點，但程式結構必須朝 RTL 容易轉換的方向設計：
  - 明確 stage / module 邊界
  - 固定張量尺寸
  - 避免 Python 黑盒 API 一步做完太多運算

## Step 5：NPY 轉 TXT / Binary 規則

- FPGA 端記憶體只讀 binary，因此 `.npy` 內容需轉成 binary text。
- `.txt` 應明確定義：
  - bit-width（預設 16-bit）
  - signed / unsigned
  - 小數位數（fraction bits，預設 8）
  - tensor flatten 順序
  - channel / token / row-major 的展平方式
- 同一份 parameter 的 `.npy`、decimal `.txt`、binary `.txt` 必須能互相對照。
- 任何 tensor flatten 規則一旦決定，不可在 Python check 與 RTL 間各自定義不同版本。
- 若未特別說明，`.txt` 與 binary `.txt` 一律視為 `Q8.8`。

## Step 6：Binary Python check 規則

- binary Python check 必須盡量做到 bit-accurate 模擬 Verilog。
- 必須明確模擬：
  - `Q8.8` 固定位寬
  - rounding / truncation
  - saturation / clamp
  - signed overflow 行為
  - pipeline / stage 間暫存
- 若某些內部乘加節點暫時使用比 `Q8.8` 更寬的位寬，必須明確記錄：
  - internal format
  - 回寫 `Q8.8` 的截斷位置
  - rounding / saturation 規則
- 對於除法：
  - 先保留真實除法版本作 golden
  - 再提供 shift / reciprocal approximation 候選版本
  - 每個近似都要記錄風險：精度損失、溢位風險、硬體成本變化
- 若要把除法改成 shift，必須先產出風險說明，再由使用者決定是否接受。

## Step 7：Python check 轉 Verilog 規則

- Verilog RTL 必須以 binary Python check 為唯一 golden model。
- RTL 模組切分應與 Python check 對齊，避免兩邊架構不同造成除錯困難。
- 每個 RTL module 都要能對應回：
  - Python 函式 / stage
  - 輸入輸出 tensor
  - 參數檔案
- 第一版 RTL 先求功能正確，再做面積 / cycle / throughput 最佳化。

## Dump 規則（Python / RTL 對拍）

- 每一層的重要節點輸出都必須能 `dump` 成固定格式，供 Python / binary Python / RTL 三方逐層比對。
- `dump` 的目的不是展示結果，而是建立可重現的 golden intermediate outputs。
- 同一個節點在不同流程中必須使用相同：
  - 檔名
  - tensor shape 定義
  - flatten 順序
  - 數值格式

### 必須可 dump 的節點

- embedding 前段：
  - `template_after_patch_embed_out`
  - `search_after_patch_embed_out`
  - `template_pos_embed`
  - `search_pos_embed`
  - `template_after_pos_add_out`
  - `search_after_pos_add_out`
- `template_post_embed_input`
- `search_post_embed_input`
- `merged_tokens`
- 每個 backbone block 的：
  - `norm1_out`
  - `attn_out`
  - `residual1_out`
  - `norm2_out`
  - `mlp_out`
  - `block_out`
- backbone 最後輸出：
  - `backbone_out`
- head 相關輸出：
  - `head_input`
  - `score_map`
  - `size_map`
  - `offset_map`
  - `pred_boxes`
- 若有做完整 test 對齊，還要能 dump：
  - `response_after_window`
  - `bbox_after_cal_bbox`
  - `final_bbox`

### 檔名規則

- 檔名命名風格必須參照 `reference/rongxuan/03_GetParameter/NPY/linearParam` 與 `layerParam`：
  - 使用 module / stage 路徑攤平成單層底線命名
  - 不使用 `b00` 這類縮寫
  - 風格應接近：
    - `stage2_0_transformer_0_attn_qkv_weight.npy`
    - `stage2_0_transformer_0_mlp_fc1_weight.npy`
- dump 檔名必須包含：
  - 模組路徑名稱
  - `after_<function_name>`
  - 節點 / 訊號名稱
  - 數值格式副檔名
- 建議格式：
  - `{module_path_flat}_after_{function_name}_{signal_name}.{ext}`
- `module_path_flat` 規則：
  - 由 Python 中的 module path 攤平成底線命名
  - `.` 一律改成 `_`
  - 例如：
    - `backbone.blocks.0.attn` -> `backbone_blocks_0_attn`
    - `backbone.blocks.0.mlp` -> `backbone_blocks_0_mlp`
    - `box_head` -> `box_head`
- `function_name` 規則：
  - 必須填寫「此輸出是在哪個 function / 運算步驟之後產生」
  - 例如：
    - `norm1`
    - `attn`
    - `residual_add1`
    - `norm2`
    - `mlp`
    - `block`
    - `forward_head`
    - `cal_bbox`
    - `output_window`
- `signal_name` 規則：
  - 用來描述該 function 後保存的具體訊號
  - 例如：
    - `out`
    - `qkv`
    - `score_map`
    - `size_map`
    - `offset_map`
    - `pred_boxes`
    - `bbox`
- 範例：
  - `template_after_patch_embed_out.npy`
  - `search_after_patch_embed_out.npy`
  - `template_pos_embed.npy`
  - `search_pos_embed.npy`
  - `template_after_pos_add_out.npy`
  - `search_after_pos_add_out.npy`
  - `backbone_blocks_0_after_norm1_out.npy`
  - `backbone_blocks_0_attn_after_qkv_qkv.npy`
  - `backbone_blocks_0_after_attn_attn_out.txt`
  - `backbone_blocks_0_after_residual_add1_out_bin.txt`
  - `backbone_blocks_0_after_norm2_out.npy`
  - `backbone_blocks_0_mlp_after_mlp_out.npy`
  - `box_head_after_forward_head_score_map.npy`
  - `box_head_after_forward_head_size_map.npy`
  - `box_head_after_forward_head_offset_map.npy`
  - `box_head_after_cal_bbox_bbox.txt`
  - `tracker_after_output_window_response.npy`
  - `tracker_after_final_bbox_bbox.txt`
- 若同一 module 內有多個 function 可能產生同名輸出，必須保留 `after_<function_name>`，不可省略。
- 所有 dump 腳本、binary check、RTL testbench 都必須使用完全相同的命名規範，不可各自簡化或改名。

### 輸出格式規則

- 每個 dump 節點至少支援：
  - `.npy`：保留原始 tensor 形狀與數值
  - `.txt`：decimal 純文字
  - `_bin.txt`：binary 純文字
- `.txt` 與 `_bin.txt` 必須為一行一個元素，避免 Python 與 RTL 解析方式不同。
- 所有 dump 檔都必須可由同一份 manifest 或 naming rule 反查：
  - 原始節點名稱
  - shape
  - dtype / Q format
- 若未特別註明，所有 dump 節點的文字格式都預設為 `Q8.8`。
- `_bin.txt` 的每一行都必須是 `Q8.8` 的 16-bit 二補數表示。

### Flatten 規則

- 每一類 tensor 都必須固定 flatten 順序，不可在不同腳本中各自定義。
- 預設採用 row-major / C-order。
- 對常見張量需明確規定：
  - `B, N, C`
  - `B, H, W, C`
  - `B, C, H, W`
  - `B, num_heads, N, d`
- 若某節點在 RTL 中會先 reorder，Python dump 必須另外提供：
  - 原始順序版本
  - RTL 實際輸入順序版本

### Dump 開關規則

- Python check、binary Python check、RTL testbench 都應提供 dump 開關。
- 開關至少分為：
  - `dump_off`
  - `dump_key_nodes_only`
  - `dump_all_nodes`
- 預設使用 `dump_key_nodes_only`，避免資料量過大。

### 對拍規則

- Python 浮點版本先作為第一層 golden。
- binary Python 版本需與浮點版本在可接受誤差內對齊。
- RTL 版本需優先與 binary Python dump 做對拍，而不是直接跳回浮點結果。
- 對拍順序固定為：
  1. `template_post_embed_input`
  2. `search_post_embed_input`
  3. `merged_tokens`
  4. block-level nodes
  5. `backbone_out`
  6. head outputs
  7. `final_bbox`

### 失敗排查規則

- 若最終 bbox 不一致，禁止直接只看 final output。
- 必須從最前面的 dump 節點開始逐層縮小誤差來源。
- 若某一層第一次出現不一致，需記錄：
  - 節點名稱
  - 最大誤差
  - 平均誤差
  - 是否可能由 rounding / saturation / reorder 造成

## 實作優先順序

1. 先固定 checkpoint 與 config
2. 先固定第一幀 / 第二幀的 golden 產生方式
3. 先固定分開的 template/search post-embedding tensor 輸入格式
4. 先完成 NPY 匯出與 manifest
5. 先完成浮點 Python check
6. 再完成 binary Python check
7. 最後才寫 RTL

## 可補充的 rules（建議後續加入）

- 明確列出「RTL 需要的參數」白名單與「不需要的參數」黑名單
- 明確定義 template/search 的固定 shape 與 token 數
- 明確定義每一層輸出的 dump 格式，方便 Python / RTL 對拍
- 為每個重要 stage 設置 golden checkpoints：
  - post-embedding input
  - block output
  - norm output
  - head output
  - bbox output
- 為每個除法 / normalization 建立兩套版本：
  - accuracy-first
  - hardware-first
- 規定所有 Python check 與 RTL testbench 都要輸出同名中間檔，方便逐層比對

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
