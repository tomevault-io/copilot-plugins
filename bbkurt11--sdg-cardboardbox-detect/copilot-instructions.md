## sdg-cardboardbox-detect

> <!-- CLAUDE.md — Claude Code project instructions + research journal -->

<!-- CLAUDE.md — Claude Code project instructions + research journal -->
<!--
  This file serves dual purposes:
  1. Project instructions for my Claude Code AI assistant
  2. A complete research journal documenting experimental decisions and findings

  Workflow: I designed the research architecture, experiments, and made all
  technical decisions. Claude Code assisted with code implementation and
  formatting. The 7 core conclusions and all ablation study designs are mine.
-->

# CardboardBox_detect — 專案說明文件

## 專案最終目標

使用筆電視訊鏡頭，即時執行兩階段 YOLO 推論：
1. **YOLO1** 偵測畫面中的紙箱，框出 The Box（class 0）
2. **Crop Program** 依照 YOLO1 輸出的 BBox 裁切出紙箱 ROI
3. **YOLO2** 在 ROI 內偵測瑕疵：Stain（class 0）、Puncture（class 1）

用戶拿著紙箱出現在鏡頭前，系統以肉眼可感受的 Realtime 同時框出 The Box / Stain / Puncture。

**核心 Feature：使用 Synthetic Data Generation（合成資料）來解決真實標記資料不足的問題。**

**方法論：Zero-shot Sim-to-Real Transfer** — 訓練集 100% 使用合成資料，透過 Domain Randomization
（隨機相機、光照、地面材質、瑕疵位置/大小/貼圖）使模型直接泛化至真實場景。

---

## 兩階段 YOLO 模型規格

| 模型 | 架構 | 輸入尺寸 | 偵測目標 | Class ID |
|------|------|----------|----------|----------|
| YOLO1 | YOLOv8n | 640×640（全幀） | 紙箱 (The Box) | 0 |
| YOLO2 | YOLOv8n | 640×640（紙箱 ROI） | Stain、Puncture | Stain=0, Puncture=1 |

> **YOLO2 的訓練輸入是 YOLO1 的輸出 crop，推論流程必須與訓練完全鏡像。**

---

## 目錄結構（實際）

```
CardboardBox_detect/
├── 01_Assets/                              # 素材庫
│   ├── Ground/                             # 地面 PBR 材質 (29 種，各含 Color/Normal/Roughness .jpg)
│   ├── HDRIs/                              # 環境光照 (.exr，20 個)
│   ├── decals/
│   │   ├── defects/                        # 瑕疵 Decal 貼圖
│   │   │   ├── Stain/                      # 污漬 (17 張 .png，帶 Alpha，2026-04-18 刪除 3 張並重新編號)
│   │   │   ├── Puncture/                   # 破孔 (7 張)
│   │   │   ├── Ink/                        # 墨水 (12 張)
│   │   │   ├── Scratch/                    # 刮痕 (16 張)
│   │   │   └── Surface_Damage/             # 表面損傷 (9 張)
│   │   └── interference/                   # 干擾貼圖
│   │       ├── Barcode/
│   │       ├── Colored_Labels/
│   │       └── Shipping Label/
│   └── real_refs/
│       └── 0408/                           # 真實參考照片 (Val / Test 專用，不入訓練)
│           ├── Roboflow_Ready/             # 134 張原始 Roboflow 匯出（含 hash 檔名，備用）
│           ├── Cardboard_project.yolov8 (1)/  # 最新 Roboflow 下載（含 labels，來源）
│           ├── labels/                     # 處理後 YOLO labels（class remap 版）
│           ├── labels_origin/              # Roboflow 原始 labels 備份（勿修改）
│           ├── clean_box/                  # 62 張  ← images/ + labels/ 已整理
│           ├── stain/                      # 24 張  ← images/ + labels/ 已整理
│           ├── puncture/                   # 20 張  ← images/ + labels/ 已整理
│           ├── both/                       # 8 張   ← images/ + labels/ 已整理
│           └── empty_background/           # 22 張  ← 備用（Hybrid Synthetic 實驗）
│
├── 02_Scripts/                             # 流水線腳本 (依序執行)
│   ├── 00_scene_health_check.py            # Blender 場景健檢，輸出 scene_spec.md / .json
│   ├── 01_orchestrator.py                  # 劇本生成器：輸出 JSON 到 06_Raw_Output/configs/
│   ├── 02_blender_render.py                # Blender 內部腳本：三 Pass 算圖 + Mask 輸出
│   ├── 03_render_operator.py                      # 外部守衛：驗證 configs → 啟動 Blender → 自動重啟
│   ├── 04_annotator_yolo1.py               # Box Mask → YOLO1 全圖 BBox labels + QA 視覺化（無增強）
│   ├── 05_cropper_yolo2.py                 # 裁切紙箱 ROI → 增強 → YOLO2 labels + QA 視覺化
│   ├── 06_build_dataset.py                 # (已部分取代) val/test 實拍圖現由內嵌腳本直接處理
│   ├── 07_augment_test.py                  # 影像增強視覺測試工具（比較原圖/YOLO1/YOLO2 效果）
│   ├── 08_train.py                         # 統一訓練腳本：--target yolo1|yolo2（RTX 4050 最佳化）
│   ├── 09_predict_yolo1.py                 # YOLO1 單段推論：跑 04_Dataset_YOLO1/images/test/ → predict_yolo1/
│   ├── 10_predict_yolo2.py                 # YOLO2 單段推論：跑 05_Dataset_YOLO2/images/test/（已 crop）→ predict_yolo2/
│   ├── 11_realtime_demo.py                 # Webcam 雙段即時：YOLO1→ROI crop→YOLO2（S 截圖到 predict_snapshots/）
│   ├── 12_qa_real_dataset.py               # 實拍 val/test 標注視覺化（人眼 QA）
│   ├── scene_spec.md                       # 00_scene_health_check 輸出的場景規格文件
│   ├── scene_spec.json                     # 同上，JSON 格式
│   │
│   └── [參考用，勿修改]
│       ├── object_orchestrator_v7.py       # 舊版劇本生成器（路徑/格式已不適用，邏輯參考）
│       ├── hsd_pipeline_mvp_v15.py         # 上一代 Blender 渲染腳本（02_blender_render.py 的原型）
│       └── auto_annotator_v6_0407.py       # 上一代 OpenCV 標註腳本（mask 處理邏輯參考）
│
├── 03_Blender_Project/
│   └── AICV_MotherFile_Cardboard_detect.blend   # Blender 主場景（不入版控）
│
├── 04_Dataset_YOLO1/                       # YOLO1 訓練資料集
│   ├── images/train/                       # 合成全圖 640×640
│   ├── images/val/                         # 真實 + 合成 clean
│   ├── images/test/                        # 真實照片
│   ├── labels/train/                       # class 0：The Box
│   ├── labels/val/
│   └── labels/test/
│
├── 05_Dataset_YOLO2/                       # YOLO2 訓練資料集（紙箱 ROI）
│   ├── images/train/                       # 合成 ROI（裁切自 YOLO1 Box BBox + 5% padding）
│   ├── images/val/
│   ├── images/test/
│   ├── labels/train/                       # class 0：Stain，class 1：Puncture
│   ├── labels/val/
│   └── labels/test/
│
├── 06_Raw_Output/                          # Blender 原始輸出（不入版控）
│   ├── configs/                            # 場景劇本 JSON（01_orchestrator.py 輸出）
│   ├── images/                             # 全圖 PNG（640×640，RGBA）
│   ├── masks/                              # Segmentation Mask PNG
│   ├── annotation/                         # (預留)
│   ├── images_ready/                       # 增強前的乾淨圖備份（供回溯比較）
│   │   ├── yolo1/                          # 04_annotator 輸出：原始全圖
│   │   └── yolo2/                          # 05_cropper 輸出：原始乾淨 ROI
│   ├── images_enhancement_test/            # 07_augment_test.py 輸出的視覺比較圖
│   └── qa_visualization/
│       ├── yolo1/                          # 04_annotator 輸出：合成訓練圖，綠框標紙箱
│       ├── yolo2/                          # 05_cropper 輸出：合成 ROI，藍框 Stain / 黃框 Puncture
│       ├── yolo1_real/val/ + test/         # 12_qa_real_dataset.py：實拍 YOLO1 val/test 標注 QA
│       └── yolo2_real/val/ + test/         # 12_qa_real_dataset.py：實拍 YOLO2 val/test 標注 QA
│
└── 07_Models/                              # 訓練完成的模型權重（待建立）
    ├── yolo1/
    └── yolo2/
```

---

## Dataset 規劃（正式版 5000 張）

### 渲染分配

| Category | 數量 | YOLO1 用途 | YOLO2 用途 |
|----------|------|-----------|-----------|
| `clean` | 2000 張 | 紙箱正樣本 | 負樣本（空 label）|
| `stain_only` | 1000 張 | 紙箱正樣本 | Stain 偵測 |
| `puncture_only` | 1000 張 | 紙箱正樣本 | Puncture 偵測 |
| `both` | 1000 張 | 紙箱正樣本 | Stain + Puncture 多類別 |
| **Total** | **5000 張** | | |

> **設計原則：一次 Blender 全圖渲染 → 程式碼裁切 ROI**
> 不在 Blender 內對準 ROI 渲染，理由：訓練時的 ROI 裁切邏輯應與推論時的 YOLO1 → crop 完全鏡像，
> 避免訓練/推論的 ROI 分布不一致產生額外的 domain gap。

---

### YOLO1 Dataset 分割（目標：最大化紙箱偵測泛化能力）

| Split | 來源 | 數量 | 說明 |
|-------|------|------|------|
| **train** | 合成圖（全類別）| **3000 張** | 2000 clean + 1000 含 decal，強化各角度/光照/地面變化 |
| **val** | 真實照片 | **40 張** | 20 張真實 clean + 20 張真實 defect（全為 class 0，讓模型學會帶瑕疵也要框紙箱）|
| **test** | 真實照片（全部）| **114 張** | clean_box(62) + stain(24) + puncture(20) + both(8)，全部放入 |

- Labels 格式：`0 cx cy w h`（class 0 = The Box，Stain/Puncture annotation 丟棄）
- Val/Test 真實照片來自 `01_Assets/real_refs/0408/`

---

### YOLO2 Dataset 分割（目標：最大化瑕疵偵測精度）

| Split | 來源 | 數量 | 說明 |
|-------|------|------|------|
| **train** | 合成 ROI | **3000 張** | 1000 stain + 1000 puncture + 1000 both，類別平衡 |
| **val** | 真實照片 ROI | **40 張** | 20 張真實 clean（空 label）+ 20 張真實 defect（class 0=Stain, class 1=Puncture）|
| **test** | 真實照片 ROI（全部）| **114 張** | 全部有紙箱的圖片，ROI crop 後放入 |

- Labels 格式：`0 cx cy w h`（Stain）/ `1 cx cy w h`（Puncture）
- Clean 圖 label 檔案為空（負樣本）
- ROI 裁切含 **5% padding** 防止瑕疵被切邊
- val/test 不套用 Sim-to-Real 增強（誠實反映真實場景表現）

---

## 腳本說明（當前實作）

> **參考檔案（勿修改）：**
> - `object_orchestrator_v7.py` — 舊版劇本生成器，邏輯參考用
> - `hsd_pipeline_mvp_v15.py` — 舊版 Blender 腳本，`02_blender_render.py` 的原型
> - `auto_annotator_v6_0407.py` — 舊版 OpenCV 標註，mask 處理邏輯參考

---

### 01_orchestrator.py

生成 Blender 場景劇本 JSON，每個 JSON 描述一幀的完整隨機設定。

```python
TOTAL_IMAGES = 500             # TEST=500 / PRODUCTION=5000
DISTRIBUTION = {               # 500 張測試          / 5000 張正式
    "clean":          250,     # 250                 / 2000
    "stain_only":     100,     # 100                 / 1000
    "puncture_only":  100,     # 100                 / 1000
    "both":            50,     #  50                 / 1000
}
OUTPUT_DIR = os.path.join(PROJECT_ROOT, "06_Raw_Output", "configs")
```

JSON 欄位結構（須與 `02_blender_render.py` 完全對齊）：
```json
{
  "frame_id": "train_0000",
  "category": "stain_only",
  "box":         { "location_xy": [...], "rotation_z": ... },
  "camera":      { "radius": ..., "azimuth": ..., "elevation": ..., "target_jitter": [...] },
  "environment": { "hdri_rotation_z": ..., "ground": { "pbr_folder": ..., "uv_scale": ..., "uv_rotation_z": ... } },
  "objects":     [ { "type": "defect", "name": "Stain_Decal", "rule": "A", ... } ]
}
```

---

### 02_blender_render.py

Blender 內部腳本，由 `03_render_operator.py` 透過 `-b --python` 啟動。每幀三個 Pass：

| Pass | 目的 | samples | Denoising | 輸出 |
|------|------|---------|-----------|------|
| PASS 1 | 主圖（彩色） | 64 | ✅ 開 | `06_Raw_Output/images/{frame_id}.png` |
| PASS 2 | 紙箱 Mask（白色 Emission） | 8 | ❌ 關 | `masks/{frame_id}_mask_0_The_Box.png` |
| PASS 3 | 瑕疵 Mask（逐一 Alpha-Preserved Emission） | 8 | ❌ 關 | `masks/{frame_id}_mask_{i}_{class}.png` |

重要設計：
- 解析度：**640×640**（直接對齊 YOLO 輸入，省去後處理 resize）
- 斷點續傳：已存在的 `{frame_id}.png` 自動跳過
- Spawn & Destroy：複製母體 `Cardboard_Box`，部署 Decal，渲染，清除
- Rule A（Stain）：Empty Pivot 投影式 Shrinkwrap
- Rule B（Puncture）：面定位式，`uv_to_local_coords` + `apply_face_rotation`
- PASS 3 只渲染有 active_defects 的幀（clean 幀跳過）

---

### 03_render_operator.py

單一入口，`python 03_render_operator.py` 啟動整個流水線。

流程：
1. `configs_are_valid()` 驗證 JSON 格式（含 box/camera.radius/environment.ground/frame_id 為字串）
2. 若無效 → 自動清除舊 configs → 執行 `01_orchestrator.py` 重新生成
3. 確認 Blender 執行檔與 `.blend` 存在
4. 渲染迴圈：啟動 Blender → 計算新增張數 → 未達 TOTAL_TARGET 則重啟

```python
TOTAL_TARGET    = 5900  # TEST=500 / PRODUCTION=5900
TIMEOUT_SECONDS = 300   # 單次 Blender 執行上限（秒）
TIMING_LOG      = "06_Raw_Output/render_timing.json"  # 計時記錄檔
```

**計時機制（2026-04-19 加入）：**

- `init_timing(start_count)` — render operator 啟動時呼叫，將起始時間與張數寫入 `render_timing.json`；若檔案已存在（斷點續跑）則直接載入，保留原始起點。
- `finalize_timing(record, final_count)` — 達到 `TOTAL_TARGET` 時呼叫，寫入結束時間並印出摘要：

```
==================================================
  渲染計時摘要（production_v1_5000）
==================================================
  開始：YYYY-MM-DD HH:MM:SS  結束：YYYY-MM-DD HH:MM:SS
  張數：0 → 5900（共 5900 張）
  耗時：HH:MM:SS
  速度：xxx 張/小時（x.x 秒/張）
  記錄：D:\CardboardBox_detect\06_Raw_Output\render_timing.json
==================================================
```

> **新實驗時**：刪除 `render_timing.json`，render operator 下次啟動時自動建立新記錄。
> **production_v1_5000 手動起點**：11:30 AM（2026-04-19），已預先寫入 `render_timing.json`。

---

### 04_annotator_yolo1.py

讀取 `06_Raw_Output/masks/{frame_id}_mask_0_The_Box.png`，用 OpenCV `findContours` 找最大輪廓，輸出 YOLO1 全圖 BBox。

**YOLO1 不套用影像增強**（全圖偵測紙箱，合成圖與真實圖差距不大，增強無益）。

資料流：
```
06_Raw_Output/images/        → 06_Raw_Output/images_ready/yolo1/   (乾淨備份)
                             → 04_Dataset_YOLO1/images/train/       (直接複製，無增強)
```

輸出：
- `06_Raw_Output/images_ready/yolo1/{frame_id}.png`（備份原圖）
- `04_Dataset_YOLO1/images/train/{frame_id}.png`（訓練圖，與原圖相同）
- `04_Dataset_YOLO1/labels/train/{frame_id}.txt`（`0 cx cy w h`）
- `06_Raw_Output/qa_visualization/yolo1/qa_{frame_id}.png`（綠框，人眼 QA）

---

### 05_cropper_yolo2.py

從全圖裁切紙箱 ROI，套用 Sim-to-Real 增強後輸出 YOLO2 訓練資料。

**關鍵設計（防止標框錯誤）：**

| 問題 | 解法 |
|------|------|
| 跨面 Stain 只框到最大輪廓 | `findNonZero` + `boundingRect(all_coords)` 框住所有非零像素 |
| Stain mask 強度弱被截掉 | Stain threshold=8 / Puncture threshold=25（各別調整）|
| 跨面邊緣碎片沒連起來 | `MORPH_CLOSE` 5×5 kernel |
| 假陽性噪點 | 面積 < 60px 或邊長 < 6px 丟棄 |
| Clean 圖出現在 QA 資料夾 | 無標框圖直接跳過 QA 存檔 |

```python
PADDING_RATIO = 0.05   # 目前 500 張 smoke test 階段「已註解停用」，程式直接 bx/by/bw/bh 裁切
CLASS_LIMIT   = 50     # TEST=50 / PRODUCTION=1000
```

> **2026-04-17 更新**：500 張 smoke test 時目視檢查 Box BBox 非常精準，padding 暫時停用（原 `roi_x1/y1/x2/y2` 計算已在程式中以註解保留）。等 5000 張正式跑、依實拍效果決定是否重新啟用。

資料流：
```
06_Raw_Output/images/  →  [crop ROI]
                       →  06_Raw_Output/images_ready/yolo2/    (乾淨 ROI 備份，未增強)
                       →  [augment_yolo2()]
                       →  05_Dataset_YOLO2/images/train/       (增強後進訓練)
```

輸出：
- `06_Raw_Output/images_ready/yolo2/{frame_id}.png`（備份原始 ROI）
- `05_Dataset_YOLO2/images/train/{frame_id}.png`（增強後訓練圖）
- `05_Dataset_YOLO2/labels/train/{frame_id}.txt`（有瑕疵才建檔）
- `06_Raw_Output/qa_visualization/yolo2/qa_{frame_id}.png`（增強後標框，藍=Stain，黃=Puncture）

---

### Sim-to-Real 影像增強策略

> **⚠️ 注意：目前增強參數為初始校準版本，效果尚未完整驗證。**
> **待 YOLO1 / YOLO2 fine-tune 後，依據 val/test 數據結果（mAP、混淆矩陣）回頭微調增強強度。**

**增強對象：僅 YOLO2 train 的合成 ROI**

| 對象 | 策略 | 理由 |
|------|------|------|
| YOLO1 train | 無增強 | 全圖偵測紙箱，合成/真實差距可接受 |
| YOLO2 train | 見下表 | ROI 為 YOLO1 crop 放大，真實畫質明顯降級 |
| Val / Test | 絕對不增強 | 為真實照片，需誠實反映部署效果 |

**YOLO2 增強參數（對齊 Acer Nitro AN515 720p CMOS webcam）：**

```python
# 推論裝置：Acer Nitro AN515，720p 基礎 CMOS，無 AWB，H.264 串流壓縮
def augment_yolo2(img):
    # 冷色偏移（固定色溫無 AWB，室內偏冷藍）
    R -= 6,  B += 10
    # 對比壓縮（webcam 動態範圍比 Blender 渲染窄）
    img = img * 0.90 + 13
    # 輕微柔焦（720p crop 放大後解析度降級）
    GaussianBlur ksize=3
    # JPEG 壓縮（H.264 串流色塊）
    JPEG quality=82   # 2026-04-18 調整：72→82，減少暗部假紋理干擾 Stain 特徵學習
    # 輕量噪點（基礎 CMOS 感光元件）
    Gaussian noise std=4
```

**07_augment_test.py**：視覺測試工具，從 `06_Raw_Output/images/` 取樣 6 張，
輸出「原圖 | YOLO1 | YOLO2」三欄比較圖到 `06_Raw_Output/images_enhancement_test/`。

---

### 08_train.py

統一訓練腳本，`--target yolo1|yolo2` 切換。

```bash
python 08_train.py --target yolo1          # YOLO1 訓練（預設 yolov8n, batch=32, epochs=300）
python 08_train.py --target yolo2          # YOLO2 訓練
python 08_train.py --target yolo1 --model yolov8s.pt   # 升級架構
python 08_train.py --target yolo1 --resume              # 斷點續訓
```

- 輸出到 `07_Models/yolo1/` 或 `07_Models/yolo2/`（`best.pt` / `last.pt`）
- `exist_ok=True`：同名 run 不報錯，直接覆蓋
- 自動開啟 `plots=True`（confusion matrix、PR curve、results.csv）

---

### 09_predict_yolo1.py

YOLO1 單段推論，固定跑 `04_Dataset_YOLO1/images/test/`（全幀實拍圖）。
用於驗證 YOLO1 模型對實拍紙箱的偵測能力，不執行 YOLO2。

```bash
python 09_predict_yolo1.py                 # 預設 last.pt + conf 0.35
python 09_predict_yolo1.py --conf 0.25     # 降低信心門檻
python 09_predict_yolo1.py --weights ../07_Models/yolo1/weights/best.pt
```

Overlay：綠框 = The Box (class 0)，粗框 `BOX_THICKNESS=4`，字體 `FONT_SCALE=0.9 / FONT_THICKNESS=2`（2026-04-17 加粗加大，提升 demo 可讀性）。
輸出到 `06_Raw_Output/predict_yolo1/pred_<stem>.jpg`。

> **⚠️ 預設權重改為 `last.pt`** — 500 張 smoke test 證實 `best.pt` 被 Ultralytics fitness 機制鎖死在 epoch 1（詳見下方「500 張 Smoke Test 研究小結」）。

---

### 10_predict_yolo2.py

YOLO2 單段推論，固定跑 `05_Dataset_YOLO2/images/test/`（**已是 ROI crop**）。
用於驗證 YOLO2 模型對瑕疵的偵測能力，不執行 YOLO1、不做 crop。

```bash
python 10_predict_yolo2.py                 # 預設 best.pt + conf 0.30
python 10_predict_yolo2.py --conf 0.20     # 降低信心門檻
python 10_predict_yolo2.py --weights ../07_Models/yolo2/weights/last.pt
```

Overlay：藍框 = Stain (0) / 黃框 = Puncture (1)
輸出到 `06_Raw_Output/predict_yolo2/pred_<stem>.jpg`。

---

### 11_realtime_demo.py

Webcam 雙段即時推論，鏡像 05_cropper_yolo2.py 的 5% padding ROI crop 邏輯。
這是部署場景的最終 demo 腳本。

```bash
python 11_realtime_demo.py                 # Webcam 0 即時（Q/ESC 退出，S 截圖）
python 11_realtime_demo.py --source 1      # 指定 Webcam index
python 11_realtime_demo.py --no-yolo2      # 只看 YOLO1 紙箱偵測
python 11_realtime_demo.py --save-all      # 每幀都存檔
```

Overlay：綠 = The Box / 藍 = Stain / 黃 = Puncture
S 截圖存到 `06_Raw_Output/predict_snapshots/snap_<timestamp>.jpg`。

---

### 12_qa_real_dataset.py

為 `04_Dataset_YOLO1` 和 `05_Dataset_YOLO2` 的 val/test 實拍圖生成標注視覺化。

```bash
python 12_qa_real_dataset.py               # 全部（YOLO1+2, val+test）
python 12_qa_real_dataset.py --yolo 2      # 只看 YOLO2
python 12_qa_real_dataset.py --split val   # 只看 val
```

輸出到 `06_Raw_Output/qa_visualization/yolo1_real/` 和 `yolo2_real/`。

---

### 06_build_dataset.py（已部分取代）

原規劃為 train/val/test 整合腳本，現已由各處理腳本直接寫入對應資料夾。
保留備用，如需重新整合可參考其邏輯。

---

## Blender 場景規格（由 scene_spec.md 確認）

| 物件 | 名稱 | 說明 |
|------|------|------|
| 紙箱 | `Cardboard_Box` | MESH，0.303×0.265×0.210 m |
| 相機 | `Camera` | PERSP，85mm |
| 地面 | `Ground` | 20×20 m，Dynamic PBR（29 種隨機切換）|
| 污漬 | `Stain_Decal` | Defects Collection，Rule A（投影式 Shrinkwrap）|
| 破孔 | `Puncture_Decal` | Defects Collection，Rule B（面定位式 Shrinkwrap）|
| 膠帶 | `Tape_1`, `Tape_2` | Cardboard_Box 子物件 |
| 算圖 | Cycles | **640×640**，RGBA，PNG |

---

## 真實參考資料（Val / Test 專用）

位於 `01_Assets/real_refs/0408/`：

| 類別 | 數量 | 用途 |
|------|------|------|
| clean_box | 62 張 | YOLO1 val(20) + test(all) / YOLO2 val(20) + test(all) |
| stain | 24 張 | YOLO1/2 val(20 中取部分) + test(all) |
| puncture | 20 張 | YOLO1/2 val(20 中取部分) + test(all) |
| both | 8 張 | YOLO1/2 val(20 中取部分) + test(all) |
| empty_background | 22 張 | 備用（Hybrid Synthetic / 負樣本強化實驗用）|

> 真實資料**全部用於 Val / Test，不混入 Train。**

---

## 500 張 Smoke Test 研究小結（2026-04-17）

### 訓練結果

- 指令：`python 08_train.py --target yolo1`（預設 yolov8n.pt, batch=32, epochs=300, patience=50）
- 實際跑：**51 epoch 觸發早停**，train loss 仍在穩定下降（box 0.51→0.31，cls 1.96→0.26）
- Test 集推論（114 張實拍）：
  - `best.pt`：**0 / 114** 檢出，max-conf 中位數 0.061（最高 0.176）— 模型沒 fire
  - `last.pt`：**109 / 114** 檢出，max-conf 中位數 0.927、平均 0.852 — 效果優異

### 根本原因：`best.pt` 被 epoch 1 鎖死

Ultralytics 以 `fitness = 0.1×mAP50 + 0.9×mAP50-95` 選 best.pt。results.csv 顯示：

| epoch | mAP50 | mAP50-95 | fitness |
|-------|-------|----------|---------|
| **1** | 0.937 | **0.885** | **0.890** ← 全程最高 |
| 17 | 0.942 | 0.845 | 0.855 |
| 49 | 0.920 | 0.852 | 0.859 |

沒有任何後續 epoch 贏過 epoch 1，所以 `best.pt` 實質上是「幾乎沒訓練過的 COCO pretrained」，對真實紙箱完全不 fire。

### 為什麼 epoch 1 會異常高？

三因素**同時湊巧**：

1. **COCO pretrained bias**：yolov8n.pt 對「棕色方形物體」本來就有強響應（COCO 的 suitcase / backpack 類別）
2. **val ⊂ test（40 張 100% 重疊）**：val 不是獨立驗證，只是 test 最容易的 40 張子集；COCO generic object prior 就能框出
3. **LR warmup**：epoch 1 lr=0.0003 很低，模型還停在 COCO sweet spot；後續 lr→0.0019 才開始真正擬合合成 domain

### 為什麼 51 epoch 就早停？

Patience=50 的意思是「連續 50 epoch 沒出現新 best fitness 就停」。因 best fitness 在 epoch 1 封頂，**epoch 1+50 = 51 自動觸發早停**。模型根本沒機會跑完 300 epoch，train loss 顯示還有進步空間 — **不是「適應很快」，是被 epoch 1 的幽靈分數綁架**。

### 其他觀察

- **Val 曲線劇烈震盪**（mAP50 在 0.32 ~ 0.94 間跳動）：40 張 val 太小 + 真實 domain 敏感度高，導致訓練監控雜訊極大
- **YOLO1 無增強**：這次還不是主因（last.pt 已能工作），但正式 5000 張時仍建議加 mosaic/HSV 以擴展泛化
- **Test 109/114 的 5 張失敗**：值得逐張檢查（特定姿態？極端光照？）

**4/18 11:24 am新增:要回blender重新調整一下stain的shader, 因為實拍跟合成數據有特徵上的差別，相信調整後stain的表現會大提升；兩個階段的yolo的config可以調高一點。

負樣本增強我個人認為大成功! 可以再研究結論特別提及。且最後可以拿所有實驗的predicted test images進行疊圖。

### 給正式 5000 張訓練的校正清單

1. **Val 與 Test 必須解耦**：從 test 114 張抽 20-30 張獨立為 val，移出 test
2. **Val 擴大至 60-80 張** 以降低指標雜訊
3. **部署權重一律用 `last.pt`**；或改用 `save_period=10` 每 10 epoch 存 checkpoint 人工挑
4. **關掉或調低 patience**（`--patience 0` 或 `20`）避免被 epoch 1 封印
5. **考慮 `freeze=10` + `warmup_epochs=3`** 壓制 epoch 1 假性高分
6. **YOLO1 加回 Ultralytics 預設增強**（mosaic/HSV 等），合成 → 真實 domain 差距變大時有用

### 已套用的程式碼變更

| 檔案 | 變更 | 理由 |
|------|------|------|
| `09_predict_yolo1.py` | `DEFAULT_WEIGHTS` best.pt → last.pt | best.pt 不可用 |
| `09_predict_yolo1.py` | 粗框 4px、字體 0.9/2，加 `The Box` 前綴 | demo 標示太細太淡 |
| `05_cropper_yolo2.py` | 5% padding 註解停用（保留備用） | 目視 BBox 精準，padding 暫不需要 |
| `05_cropper_yolo2.py` | MORPH_CLOSE kernel Stain 5×5→7×7 / Puncture 保持 5×5 | 7×7 連接跨面 Stain 碎片；Puncture 保持邊緣精準 |
| `05_cropper_yolo2.py` | JPEG quality 72→82 | 減少暗部假紋理干擾 Stain 特徵學習 |
| `01_orchestrator.py` | Puncture scale 下限 0.20→0.25 | 過小 BBox（<25px）推論 conf 極低，無訓練意義 |
| `01_orchestrator.py` | 新增 `environment.bg_strength: 0.8~1.5` | 避免過暗 HDRI 造成全黑場景 |
| `02_blender_render.py` | `apply_dynamic_decal` Puncture 載入時強制 Non-Color | 預設 sRGB 讓深色破口被 gamma curve 扭曲 |
| `02_blender_render.py` | PASS1 前套用 `bg_strength` 倍率；frame 結束還原至真實原值 | 防止多幀間 bg_strength 累乘 |
| `01_orchestrator.py` | 移除 `TOTAL_IMAGES`，改用 `SAFETY_MARGIN=1.30`；defect 類別生成 `ceil(num × 1.30)` 份 config | 補償 mask 面積過濾造成的 YOLO2 數量缺口 |
| `04_annotator_yolo1.py` | 新增 `EXPERIMENT_NAME` 常數；train 輸出路徑改為版本化（`04_Dataset_YOLO1/{EXP}/`）；自動生成 `data.yaml` | 與 `08_train.py` 路徑一致，消除每次新實驗手動 mv 的必要 |
| `05_cropper_yolo2.py` | 新增 `EXPERIMENT_NAME` 常數；train 輸出路徑改為版本化（`05_Dataset_YOLO2/{EXP}/`）；自動生成 `data.yaml`；新增 `configs_seen` / `reclassified` 追蹤與結束摘要表 | 路徑一致性 + 數量控制診斷 |

---

## YOLO2 500 張 Smoke Test 研究小結（2026-04-18）

### 訓練結果

- 指令：`python 08_train.py --target yolo2`（yolov8n.pt, batch=32, epochs=300, patience=50）
- Train loss：**持續下降**（box ~2.5→1.0，cls ~5.5→0.85，dfl ~2.2→0.9）— 模型確實在學習
- Val metrics：**mAP50 ≈ 0.15~0.25（劇烈震盪），mAP50-95 ≈ 0.05~0.10**
- **Results CSV 消失**：`exist_ok=True` 覆蓋 run 時 Ultralytics 不重建 results.csv（`07_Models/yolo2/` 無 CSV）

### 混淆矩陣（Confusion Matrix，val 集）

| True＼Predicted | Stain | Puncture | background |
|-----------------|-------|----------|------------|
| Stain           | 0%    | —        | **100%** ← Stain 完全漏偵測 |
| Puncture        | —     | **74%**  | 26% |
| background      | 35%   | **65%**  | 35% |

- **Stain recall = 0%**：val 集所有 Stain GT 被判為 background
- **Puncture recall = 74%**：表現尚可
- **Background FP 極高**：65% 的 background 被誤判為 Puncture（模型過度 fire）

### 預測圖目視評估（test 集）

| 現象 | 評估 |
|------|------|
| Stain 框碎片化（同一塊污漬出現 3-4 個重疊框）| ⚠️ 位置偏差大，IoU < threshold，val 集計為 FP |
| Puncture 單框，信心值 0.71~0.81 | ✅ 視覺上合理 |
| 部分 Stain/Puncture 標在紙箱邊緣外（背景地面）| ❌ Background FP 問題明確 |
| Low-conf Stain (0.33, 0.35) 出現在非污漬區域 | ❌ 假陽性 |

### 根本原因分析

1. **訓練資料嚴重不足**：196 張（目標 3000 張），其中 stain=100、puncture=50、both=50
   → 模型對 Stain 特徵學習不完整，碎片化而非 bbox 精準
2. **Background FP 來源**：ROI 內包含紙箱邊緣、膠帶、印刷文字等非瑕疵紋理
   → 模型誤把某些紋理認定為瑕疵，尤其 Puncture（深色小圓）與紙紋相似
3. **負樣本不足**：YOLO2 train 的 50 張 clean（空 label）佔比過低，模型沒有足夠的「純紙箱背景不要 fire」學習訊號
4. **Sim-to-Real gap**：增強參數（冷色偏移、模糊、JPEG 壓縮）可能改變了 Stain 色調使 val 集 IoU 失配
5. **Val 集太小（40 張）+ Stain 樣本少**：mAP 震盪大，單張偏差就影響整體指標

### 已確認的設計決策

- **`05_cropper_yolo2.py` padding 保持停用**：YOLO1 BBox 已非常精準（目視確認），啟用 padding 只會引入更多背景紋理，加劇 Background FP 問題；正式 5000 張後視需要再評估

### ⚠️ 關於 best.pt vs last.pt（YOLO2 與 YOLO1 情況不同）

YOLO1 的 `best.pt` 被 epoch 1 鎖死是**極端特例**，三因素同時湊巧（COCO pretrain 強先驗 + val⊂test + LR warmup）。YOLO2 的 Stain/Puncture 在 COCO 無先驗，epoch 1 不會複製這個問題。

YOLO2 的 val 指標問題是**雜訊太大導致 patience 提早觸發**，解法是：
- `--patience 20`（縮短，讓 best.pt 在 loss 仍下降時更新）
- `save_period=10`（每 10 epoch 存一次 checkpoint，人工挑選）
- 擴大 val 集至 60~80 張（降低指標震盪）

### 待驗證實驗：加入 empty_background 負樣本（可在 smoke test 階段先試）

**目的**：解決 Background FP 問題，不必等 5000 張正式訓練。

**做法**：
- 將 `01_Assets/real_refs/0408/empty_background/` 22 張加入 `05_Dataset_YOLO2/images/train/`（空 label 負樣本）
- 重新跑 `python 08_train.py --target yolo2`，對比混淆矩陣的 background FP 欄是否下降

**預期效果**：讓模型見到純背景紋理（無紙箱、無瑕疵），降低把地面 / 牆面誤判為瑕疵的傾向。

### 給正式 5000 張訓練的校正清單（YOLO2）

1. **資料量是主因**：3000 張 ROI（1000 stain + 1000 puncture + 1000 both）預期可顯著改善 Stain recall
2. **負樣本強化**：`empty_background/` 22 張效果先在 smoke test 驗證；正式版考慮提高 clean 比例至 500 張
3. **提高 Stain mask threshold 可信度**：確認 `05_cropper_yolo2.py` threshold=8 不會引入噪點標框
4. **Stain 碎片化改善**：`MORPH_CLOSE` kernel 可加大（目前 5×5），減少多框覆蓋同一瑕疵
5. **Val/Test 解耦 + Val 擴大至 60~80 張**（與 YOLO1 校正清單相同）
6. **`--patience 20` + `save_period=10`** 防止 val 雜訊造成訓練提早終止
7. **Augmentation 強度**：待資料量足後，比對 val mAP 決定是否調低冷色偏移/模糊強度

---

## 實驗版本管理（2026-04-18 重組）

模型與 Dataset 均按實驗命名，每次新實驗只需改三個常數（`08_train.py`）：

```python
EXPERIMENT_NAME  = "smoke_test_v2_HN"   # 模型輸出到 07_Models/{EXPERIMENT_NAME}/
YOLO1_DATA_EXP   = "smoke_test_v2_HN"   # YOLO1 data → 04_Dataset_YOLO1/{YOLO1_DATA_EXP}/
YOLO2_DATA_EXP   = "smoke_test_v2_HN"   # YOLO2 data → 05_Dataset_YOLO2/{YOLO2_DATA_EXP}/
```

`09_predict_yolo1.py`、`10_predict_yolo2.py`、`11_realtime_demo.py` 各自頂部也有 `EXPERIMENT_NAME` 需同步更新。

### 實驗版本一覽

| 實驗 | 模型 | YOLO1 Dataset | YOLO2 Dataset | 說明 |
|------|------|--------------|--------------|------|
| `smoke_test_v1_500SDG` | `07_Models/smoke_test_v1_500SDG/` | `04_Dataset_YOLO1/smoke_test_v1_500SDG/` | `05_Dataset_YOLO2/smoke_test_v1_500SDG/` | 500 合成，val/test 5% padding |
| `smoke_test_v2_HN` | `07_Models/smoke_test_v2_HN/` | `04_Dataset_YOLO1/smoke_test_v2_HN/` | `05_Dataset_YOLO2/smoke_test_v2_HN/` | YOLO1 +22 HN 空背景；YOLO2 tight crop（無 padding）|
| `production_v1_5000` | `07_Models/production_v1_5000/` | `04_Dataset_YOLO1/production_v1_5000/` | `05_Dataset_YOLO2/production_v1_5000/` | **正式版**：5900 張算圖，純合成（無 HN），val/test 解耦，--patience 20 |
| `production_v1_noaug` | `07_Models/production_v1_noaug/yolo2/` | （沿用 v1_5000）| `05_Dataset_YOLO2/production_v1_noaug/` | **YOLO2 Ablation A**：關掉 cropper 的 Sim-to-Real 增強，Ultralytics 增強保留（2026-04-21）|
| `production_v1_noultaug` | `07_Models/production_v1_noultaug/yolo2/` | （沿用 v1_5000）| （沿用 v1_5000）| **YOLO2 Ablation B**：保留 cropper 增強，用 `--no-yolo-aug` 關掉 Ultralytics 內建增強（2026-04-21）|
| `production_v1_hybrid` | `07_Models/production_v1_hybrid/yolo2/` | （沿用 v1_5000）| `05_Dataset_YOLO2/production_v1_hybrid/` | **Hybrid v1（失敗）**：noaug 4000 + 12 真實正例（stain×5 + puncture×5 + both×2），無負例 → Clean FP 76%（2026-04-22）|
| `production_v1_hybrid_v2` | `07_Models/production_v1_hybrid_v2/yolo2/` | （沿用 v1_5000）| `05_Dataset_YOLO2/production_v1_hybrid_v2/` | **Hybrid v2 🏆**：noaug 4000 + 44 真實（12 stain + 10 puncture + 2 both + **20 clean negative anchor**），`--no-yolo-aug`。best@ep92，mAP50=0.89，Clean FP=0%（2026-04-22）|
| `production_v1_realonly` | `07_Models/production_v1_realonly/yolo2/` | （沿用 v1_5000）| `05_Dataset_YOLO2/production_v1_realonly/` | **純實拍對照組（不公平，作廢）**：**0 合成** + 44 真實，`--no-yolo-aug`。best@ep9、29 ep 早停，val mAP50=**0.111**，test **0/40** — 關掉 aug 對 44 張過於苛刻（2026-04-22）|
| `production_v1_realonly_fair` | `07_Models/production_v1_realonly_fair/yolo2/` | （沿用 v1_5000）| `05_Dataset_YOLO2/production_v1_realonly/`（沿用）| **純實拍對照組（公平版，採用）**：**0 合成** + 44 真實，Ultralytics 預設 aug 全開。best@ep113、163 ep 早停，val mAP50=**0.69**（Punct 0.835 / Stain 0.546），test **10/14 defect + 6/26 Clean FP** — 證實合成資料提升 mAP50 +0.20、Clean FP 歸零（2026-04-22）|
| `production_v2_HN` 🏆 | `07_Models/production_v2_HN/yolo1/` | `04_Dataset_YOLO1/production_v2_HN/` | （沿用 v1_5000）| **YOLO1 + HN 5000 規模對照（完成 P001 三段 ablation）**：6427 合成 + 22 真實 HN 空背景，`--patience 20`。best@**ep10**（fitness 0.995, mAP50=**0.998**），33 ep 早停，test **best.pt 79/84 = 94%**（first time best > last）— 證實 HN 在 5000+ 規模仍能排除 P001 epoch-early 鎖死（2026-04-22）|

### smoke_test_v2_HN 實驗目標

**YOLO1：**
1. 驗證 22 張 HN 空背景是否提升紙箱識別能力（降低 FP、提升 precision）
2. 檢查 `09_predict_yolo1.py` 新的粗框/大字 overlay 視覺效果
3. 回測 epoch=1 鎖死 best.pt 極端情形是否再次出現（注意：本次仍用預設 `patience=50`，三因素中 val⊂test 與 COCO pretrain 未排除 → 若仍鎖死屬預期，等正式 5000 張再改 patience 與 val/test 解耦）

**YOLO2：**
1. 取消 val/test padding 後是否更會抓瑕疵（減少 ROI 邊緣背景干擾）
2. **不加 HN 空背景**：YOLO2 推論輸入永遠是 YOLO1 crop 後的紙箱 ROI，不會見到純背景場景 → HN 空背景對 YOLO2 無訓練意義

### smoke_test_v2_HN Dataset 規格

| Dataset | Split | 圖片 | Labels | 與 v1 差異 |
|---------|-------|------|--------|-----------|
| YOLO1 | train | **522** | 522 | **+22 HN empty_background**（空 label，全部進 train）|
| YOLO1 | val | 40 | 40 | 同 v1 |
| YOLO1 | test | 114 | 114 | 同 v1 |
| YOLO2 | train | 196 | 146 | 同 v1（移除原先放錯的 22 HN）|
| YOLO2 | val | 40 | 40 | **tight crop（無 padding）** |
| YOLO2 | test | 114 | 114 | **tight crop（無 padding）** |

> YOLO1 train 522 = 500 合成全圖 + 22 Hard Negative（真實空背景，空 label）
> HN 比例 = 22/522 ≈ 4.2%，落在 Ultralytics 建議的 0~10% background 範圍內

### 為什麼 HN 全部進 train、不分 val？

根據 Ultralytics [Tips for Best Training Results](https://docs.ultralytics.com/yolov5/tutorials/tips_for_best_training_results/)：
- **Train**：建議加入 0~10% background images（降 FP、提升 precision）
- **Val**：官方未要求，且 val 加空圖會讓預測出任何框都計 FP，反而干擾 best.pt 挑選
- 本 smoke test 僅 22 張 HN，量少不適合分割；正式 5000 張實驗時可蒐集更多空背景（例如 200+ 張）再分 train/val/test

---

### production_v1_5000 實驗目標（2026-04-19 啟動）

**設計原則：純合成基準（不加 HN），用於與後續 production_v2 對比 HN 效益**

> HN 負樣本增強刻意留到 `production_v2` 才加入，方便直接比對加 HN 前後對 YOLO1/2 指標的改善幅度。

**YOLO1：**
1. 10× 資料量（5900 張 vs 500 張）是否讓 best.pt epoch 穩定、mAP50 > 0.97
2. 無 HN 條件下 epoch-1 鎖死是否再現（COCO pretrain + val⊂test 仍存在）
3. val/test **解耦後**指標是否更穩定（正式訓練前必須完成）

**YOLO2：**
1. Stain recall 從 0% 改善至可接受水準（新 shader + 3000 張 ROI）
2. Puncture background FP 從 64% 降至 < 30%（更多負樣本比例）
3. `--patience 20` 防止 val 雜訊觸發早停

### production_v1_5000 Dataset 規格（✅ **全部就緒，可直接訓練**）

| Dataset | Split | 張數 | 來源 | 備註 |
|---------|-------|------|------|------|
| YOLO1 | train | **6427** | 合成全圖（新 Stain shader）| 無 HN，純合成；6450 張算圖後 22 張 Box mask 失敗、1 張無輪廓（yield 99.6%）|
| YOLO1 | val | **30** | 實拍（解耦後）| both=2, clean_box=16, puncture=5, stain=7，val ⊄ test |
| YOLO1 | test | **84** | 實拍（解耦後）| both=6, clean_box=46, puncture=15, stain=17 |
| YOLO2 | train | **4000** | 合成 ROI（1000×4）| CLASS_LIMIT=1000/類，tight crop；clean 1000 空 label + stain/puncture/both 各 1000 |
| YOLO2 | val | **30** | 實拍 ROI（解耦後）| 與 YOLO1 val 同一批圖，tight crop |
| YOLO2 | test | **84** | 實拍 ROI（解耦後）| 與 YOLO1 test 同一批圖，tight crop |

**YOLO2 train 類別實例分布（從 label 檔案解析）：**

| 類別 | 圖片數（含該類 label）| 備註 |
|------|---------------------|------|
| Stain (class 0)    | 1378 張 | 1000 stain_only + 378 both（Puncture mask 被過濾掉）|
| Puncture (class 1) | 1139 張 | 1000 puncture_only + 139 both（Stain mask 被過濾掉）|
| Both 兩類同現      | 483 張  | both category 中 Stain + Puncture 都通過面積過濾 |
| Clean（空 label）  | 1000 張 | 純負樣本 |

> Stain instance 比 Puncture 多（1378 vs 1139），因 Puncture mask 容易被 `dw*dh < 60` 過濾。訓練時若出現 class imbalance 可調 loss 權重，但差距不大（~21%）可先跑看看。

### 補算歷史（2026-04-21）

首次跑 `05_cropper_yolo2.py` 時 puncture_only 缺口 286 張（yield 714/1300 = 54.9%）。透過在 orchestrator 加入 `FRAME_START_OFFSET` 機制補跑 550 張 puncture_only（train_5900~train_6449），渲染完成後重跑 cropper 填滿缺口。最終 5900 + 550 = 6450 張原始算圖，YOLO2 四類全部達標 1000/1000/1000/1000。

> 補跑完 orchestrator / render operator 的常數已還原為正式版設定（`DISTRIBUTION` 5000 分配、`SAFETY_MARGIN=1.30`、`FRAME_START_OFFSET=0`、`TOTAL_TARGET=5900`）。

> val/test 解耦已於 2026-04-19 完成（val ⊄ test 已驗證，重疊=0）。data.yaml 已建立。

---

## smoke_test_v2_HN 訓練結果小結（2026-04-18）

### YOLO1 — 核心突破：Epoch-1 鎖死問題排除 ✅

| 指標 | v1（smoke_test_v1_500SDG）| v2（smoke_test_v2_HN）| 變化 |
|------|--------------------------|----------------------|------|
| best.pt 對應 epoch | **1**（COCO pretrain 鎖死）| **30**（真實訓練結果）| ✅ 鎖死排除 |
| best.pt fitness | 0.890（假高分）| 0.867（真實訓練分）| — |
| best.pt mAP50 | 0.937 | **0.968** | ✅ +0.031 |
| best.pt mAP50-95 | 0.885 | 0.855 | 略降（訓練 epoch 不同）|
| 訓練停止 epoch | 51（epoch 1+50 patience）| **80**（epoch 30+50 patience）| ✅ +29 epoch |

**根本原因**：22 張 HN 空背景稀釋了 COCO pretrain 在 epoch 1 的假性高分（fitness 0.890→0.855），使 epoch 30 的真實訓練分（0.867）得以超越 epoch 1，best.pt 不再被鎖死。

**Confusion Matrix（val 集）：**
- The_Box recall = **0.92**（7% FN 可接受）
- background → The_Box FP = **0%**（HN 有效，空背景全部正確忽略）

### YOLO2 — 進步有限但方向正確

| 指標 | v1 | v2 | 變化 |
|------|----|----|------|
| Stain recall | 0% | **0%** | 不變（資料量問題）|
| Puncture recall | 74% | **79%** | ✅ +5% |
| background→Stain FP | **35%** | **0%** | ✅ tight crop 完全消除 |
| background→Puncture FP | 65% | **64%** | 幾乎不變 |

**Stain recall = 0% 的真正機制（由 predict 圖揭露）：**
模型**有 fire Stain**（conf 0.84 / 0.61 / 0.80），但同一塊污漬被切成 3-4 個碎片框，每個框獨立 IoU < 0.5，全部計為 FP → confusion matrix 顯示 0% 並非「看不見」，而是「框得太碎無法過 IoU 門檻」。根本原因是合成 Stain 特徵與真實污漬差距（shader 不符），加上資料量不足。

**⭐ 負樣本實驗結論（用戶評估：大成功）：**
HN 空背景對 YOLO1 的 epoch-1 鎖死問題產生決定性改善；background→Stain FP 從 35% 降至 0% 是 tight crop 的直接貢獻。正式 5000 張訓練報告中將特別提及此實驗成果。

### 已確認待改善項目（下一步行動依據）

| 問題 | 根本原因 | 解法 |
|------|---------|------|
| Stain recall = 0% | 合成 shader 與真實污漬特徵差距；資料量 196 張（目標 3000）| Blender 調整 Stain shader → 5000 張正式算圖 |
| Puncture background FP = 64% | 紙板紋理與破孔外觀相似；資料量不足 | 5000 張正式算圖（更多負樣本比例）|
| YOLO2 低信心 FP 雜訊 | conf 閾值偏低（0.30）| 部署時提高 `CONF_YOLO2` 0.30 → 0.45 |
| YOLO1 val 曲線震盪 | val 只有 40 張，真實 domain 敏感 | 正式訓練前 val/test 解耦，val 擴至 60-80 張 |

---

## production_v1_5000 訓練結果小結（2026-04-21）

### 訓練概況

| 項目 | YOLO1 | YOLO2 |
|------|-------|-------|
| 資料量 | 6427 train / 30 val / 84 test | 4000 train / 30 val / 84 test |
| 實際訓練 epoch | **22**（early stop）| **21**（early stop）|
| best.pt 對應 epoch | **epoch 2**（fitness 0.951）| **epoch 1**（fitness 0.166）|
| 停止機制 | patience=20，best@2 + 20 = 22 觸發 | patience=20，best@1 + 20 = 21 觸發 |
| Train loss 趨勢 | box 0.48→0.24（穩定下降）| box 2.20→1.62（穩定下降）|
| Val loss 趨勢 | 小幅震盪 | **劇烈震盪**（Sim-to-Real gap 顯著）|

### ⚠️ 核心發現：P001 epoch-1 鎖死**在無 HN 情境下再次發生**（YOLO1 + YOLO2 雙雙中招）

| 實驗 | HN 空背景 | YOLO1 best epoch | YOLO2 best epoch |
|------|-----------|------------------|-----------------|
| smoke_test_v1_500SDG | ❌ 無 | **1**（鎖死）| 未紀錄 |
| smoke_test_v2_HN | ✅ 22 張 | **30**（鎖死排除）| 未紀錄 |
| production_v1_5000 | ❌ 無（刻意）| **2**（鎖死再現）| **1**（YOLO2 首次被鎖）|

**這等於三組 ablation 完美對照**：有 HN → 鎖死排除；無 HN → 不論 500 / 5000 張規模都會鎖死。**HN 是 P001 的決定性解法**，資料量增加無效。

> CLAUDE.md 先前推測「YOLO2 的 Stain/Puncture 在 COCO 無先驗，epoch 1 不會複製此問題」**已被實測打臉**。P001 根本原因是 val 震盪 + patience 邏輯 + fitness 公式（0.1×mAP50 + 0.9×mAP50-95），**與 COCO bias 無關，任何模型都可能中招**。→ 新增 P012 問題紀錄。

---

### YOLO1 分析

#### Confusion Matrix (val 30 張)

```
                 True The_Box    True background
Pred The_Box         30              9   ← 9 張把背景誤框為紙箱
Pred background       0              0
```

- **Recall = 30/30 = 100%** ✅
- **FP = 9**（Precision 30/39 = 77%）
- 與 smoke_test_v2_HN（recall=92%, FP=0%）相比，**recall 微升、FP 顯著增加**

#### best.pt vs last.pt（目視評估）

| 權重 | test 84 張典型表現 |
|------|-------------------|
| best.pt (epoch 2) | 偏 COCO pretrain，檢出多但框鬆 |
| **last.pt (epoch 22)** | **多數框精準（conf 0.97）**，少數仍偏鬆（延伸到窗戶/地磚）|

**推薦部署：last.pt**（與 smoke_test_v1 結論一致）。

#### 失敗 case 樣本

- `pred_Real_sample_puncture_001`：紙箱僅佔畫面一小塊，綠框卻涵蓋整張圖（紗窗+地磚）— 嚴重誤框
- `pred_Real_sample_clean_box_011`：室內+窗外場景，框延伸到窗戶區域
- 規律：**背景複雜度高時**框會鬆，此類圖貢獻了大部分 9 FP

---

### YOLO2 分析

#### Confusion Matrix (val 30 張)

```
                True Stain   True Puncture   True background
Pred Stain           0            0              0
Pred Puncture        0            4              0
Pred background     10           12              0
```

| 類別 | Recall | vs smoke_test_v2_HN |
|------|--------|---------------------|
| Stain | **0 / 10 = 0%** | 持平 0%（shader 改了未救到 val）|
| Puncture | **4 / 16 = 25%** | **嚴重倒退**（smoke=79%）|
| Background FP | **0** | ✅ 維持 0（tight crop 的功勞）|

#### Test 84 張目視結果

| 權重 | 檢出張數 | 評語 |
|------|---------|------|
| best.pt (epoch 1) | **8 / 84** | 幾乎無作用（接近 COCO pretrain 隨機輸出）|
| **last.pt (epoch 21)** | **顯著增加**（用戶目測）| 對明顯瑕疵工作，對細微瑕疵碎片化 |

#### 兩極化行為（last.pt 樣本）

| 樣本 | 表現 |
|------|------|
| `both_001` | ✅ 驚人：Stain 0.55 / 0.32 / 0.41 + **Puncture 0.76 完美對準破口** |
| `stain_001` | ❌ 碎片化：6 個低信心 Stain 框（0.31~0.35）疊在文字/插圖上，未對準真實水漬 |

**規律**：特徵鮮明（深色新鮮水漬 + 圓形破口）時可 fire；淡色/乾漬會退化成「看到任何紋理就框」。

#### Puncture 倒退原因（smoke v2 79% → v1 25%）

1. **Best.pt 切換**：smoke v2 的 best=epoch 未追溯；v1 best=epoch 1（純 COCO）實測更差
2. **雙重增強疊加**：`args.yaml` 顯示 YOLO 預設 `mosaic=1.0`、`hsv_s=0.7`、`erasing=0.4` 等仍全開，**疊在 cropper 的 Sim-to-Real 增強之上**，Puncture 特徵可能被過度擾亂
3. **過擬合加深**：4000 張合成 ROI 讓模型對「合成 Puncture」學習更深 → 對真實 domain 泛化下降（train loss vs val loss gap 擴大）

---

### 實驗評價：**6.5 / 10 — 基線確立，天花板未探到**

| 維度 | 評分 | 評語 |
|------|------|------|
| Pipeline 工程 | **9/10** | 6450 張算圖 + FRAME_START_OFFSET 補跑機制 + 版本化路徑，極穩定 |
| 資料品質 | 7/10 | Shader 已大幅改善，但 Stain 真實/合成 gap 仍存在 |
| 訓練方法論 | 5/10 | **沒先加 HN 就跑 5000 張**是最大失誤 |
| YOLO1 實用性 | 8/10 | last.pt 可用於部署 |
| YOLO2 實用性 | 4/10 | 僅對明顯瑕疵 fire |
| 研究/論文價值 | **9/10** | 三組 ablation（500+無HN / 500+有HN / 5000+無HN）完美對照，HN 的關鍵性被徹底證實 |

### 最大收穫

> **「即使資料量擴大 10 倍（500→5000），沒加 HN 仍然會再現 P001 epoch-1 鎖死」** — 這句就是 production_v2_HN 對比實驗存在的完整學術合理性。現在手上有三組 YOLO1 對照組，是 ablation study 的完美素材。

### 最大失誤

> 當初預期「production_v1 純合成」可作為 baseline，實際上因 P001 鎖死，**baseline 的 best.pt 幾乎等於 COCO pretrain**，對比性大打折扣。未來設計類似實驗時應**無論如何都加入 HN**，再另設「無 HN 對照組」。

---

### 待驗證：Webcam 即時 Demo（Path A，推薦立即執行）

兩模型 last.pt 雖未完美但已具 demo 能力。建議：
1. 將 `11_realtime_demo.py` 的權重路徑切為 last.pt
2. 提高 conf 閾值過濾低信心雜訊：`CONF_YOLO1 = 0.50`、`CONF_YOLO2 = 0.45`
3. 拿紙箱錄 2 分鐘，不論結果好壞都是 demo / 論文素材

### 後續實驗優先序

| Path | 時程 | 預期收益 |
|------|------|---------|
| **A. Webcam demo** | 1 小時 | 驗證 pipeline 整條跑通，取得 demo 影片 |
| **B. YOLO2 重訓（關 mosaic/erasing、降 hsv）** | 半天 | Stain recall 從 0% 提升至 20-40% |
| **C. production_v2_HN 對比實驗** | 1 天 | 量化 HN 對 5000 張規模的效益（論文核心數據）|
| D. Blender Stain shader 再調 | 數天 | 僅 A+B+C 無法改善才考慮 |

---

## YOLO2 雙增強 Ablation 實驗小結（2026-04-21）

### 實驗動機

production_v1_5000 的 YOLO2 表現差（Puncture recall=25%、Stain=0%），`07_Models/production_v1_5000/yolo2/args.yaml` 顯示 Ultralytics 預設增強全開（`mosaic=1.0`、`erasing=0.4`、`hsv_s=0.7`、`auto_augment=randaugment`），疊在 `05_cropper_yolo2.py` 的 Sim-to-Real 增強（冷色+對比壓縮+blur+JPEG+noise）之上 → **雙重增強可能過度破壞 Puncture 細節**（P013 假說）。設計兩組對照實驗驗證：

| 實驗 | Cropper aug | Ultralytics aug | Dataset |
|------|-------------|-----------------|---------|
| `production_v1_5000`（baseline）| ✅ | ✅ | v1_5000 |
| `production_v1_noaug` | ❌ | ✅ | **新建 `05_Dataset_YOLO2/production_v1_noaug/`**（同樣 4000 ROI，未套用 `augment_yolo2()`）|
| `production_v1_noultaug` | ✅ | ❌ | 沿用 v1_5000 |

### 程式碼變更

| 檔案 | 變更 | 用途 |
|------|------|------|
| `05_cropper_yolo2.py` | 新增 `APPLY_AUGMENTATION` 常數 | `False` 時直接寫入原始 ROI 不套用 `augment_yolo2()` |
| `08_train.py` | 新增 `--no-yolo-aug` 旗標 | 對 YOLO2 覆寫 `mosaic=0.0 / erasing=0.0 / hsv_s=0.3 / hsv_v=0.2 / auto_augment=""` |
| `_predict_compare_yolo2.py`（新）| 一次性腳本 | 跑三組推論 + 並排對比圖到 `06_Raw_Output/predict_yolo2/_compare/` |

### 訓練結果（三組皆 `--patience 20`，best.pt）

| 實驗 | Best epoch | Val Stain mAP50 | Val Puncture mAP50 | Val Puncture R | Test 檢出 (best.pt, 84 張) |
|------|-----------|-----------------|---------------------|----------------|---------------------------|
| v1_5000 | **1**（鎖死）| ~0.008 | ~0.10 | 25% | 8/84 |
| **noaug** | **1**（鎖死）| 0.009 | **0.76** | **68.8%** | 4/84 ⚠️ |
| **noultaug** | **2**（鎖死）| **0.043** | 0.385 | 43.8% | **40/84** 🔥 |

### 核心發現

1. **Cropper 的 Sim-to-Real 增強對 Puncture 傷害最大**：拿掉後 val Puncture mAP50 從 ~0.10 → **0.76**（7.6×），recall 從 25% → **68.8%**
2. **Ultralytics 增強也有負面影響但較小**：noultaug 的 Puncture mAP50=0.385（比 baseline 好 3.8×）
3. **Stain 三組皆近 0%** — 證實 Stain 問題**與任何 aug 無關**，純粹是 Blender shader 跟真實污漬的 domain gap，需回頭調 shader（Path D 優先度應拉高）
4. **P001 epoch-1 鎖死三組都中招**（1/1/2）— 再次確認 HN 是唯一驗證過的解法，單純改變 aug 不會排除鎖死
5. **Val / Test 嚴重不一致**：noaug 在 val 最高（0.76 Puncture）但 test 檢出最少（4/84），noultaug val 中等但 test 最多（40/84）。val 只有 30 張太小，單張偏差就扭曲整體 → 證實 P004 未解，production_v2 需再擴大 val

### 訓練結果（四組皆 `--patience 20`，best.pt）

| 實驗 | Best epoch | Stain mAP50 | Puncture mAP50 | Puncture R | Test 檢出 (best.pt, 84 張) |
|------|-----------|-------------|----------------|------------|---------------------------|
| v1_5000 | **1**（鎖死）| ~0.008 | ~0.10 | 25% | 8/84 |
| **noaug** | **1**（鎖死）| 0.009 | **0.76** | **68.8%** | 4/84 ⚠️ |
| **noultaug** | **2**（鎖死）| **0.043** | 0.385 | 43.8% | **40/84** |
| **hybrid** | **4** ✅ | **0.061** | 0.676 | 66.5% | 72/84（❌ 假象，FP 爆炸）|
| **hybrid_v2** | **92** ✅✅ | **0.89（total mAP50）** | — | **—** | 13/40（test 縮至 40） — **Clean FP = 0/26 = 0% 🏆** |

### 核心發現

1. **Cropper 的 Sim-to-Real 增強對 Puncture 傷害最大**：拿掉後 val Puncture mAP50 從 ~0.10 → **0.76**（7.6×），recall 從 25% → **68.8%**
2. **Ultralytics 增強也有負面影響但較小**：noultaug 的 Puncture mAP50=0.385（比 baseline 好 3.8×）
3. **Stain 三組皆近 0%** — 證實 Stain 問題**與任何 aug 無關**，純粹是 Blender shader 跟真實污漬的 domain gap，需回頭調 shader（Path D 優先度應拉高）
4. **P001 epoch-1 鎖死三組都中招**（1/1/2）— 再次確認 HN 是唯一驗證過的解法，單純改變 aug 不會排除鎖死
5. **Val / Test 嚴重不一致**：noaug 在 val 最高（0.76 Puncture）但 test 檢出最少（4/84），noultaug val 中等但 test 最多（40/84）。val 只有 30 張太小，單張偏差就扭曲整體 → 證實 P004 未解，production_v2 需再擴大 val

### Hybrid 實驗補充與失敗分析（2026-04-22）

在 noaug 4000 張合成 ROI 加入 **12 張真實圖**（stain×5 + puncture×5 + both×2，從 test pool 抽出，val 不動）並搭配 `--no-yolo-aug` 訓練：

**表面數字（誤導性）：**
- Best epoch = 4（真實圖打破了 epoch-1 鎖死）
- Test 檢出 72/84（四組最高）
- Stain mAP50 0.009 → 0.061（6×）

**實際問題（FP 爆炸）：**

深入分析後發現 72/84 幾乎全是 FP：
- Clean 圖誤框：**35/46（76%）** → 製造 22 個 Stain FP + 93 個 Puncture FP
- 瑕疵命中：25/26（比 noultaug 的 26/26 僅多 1 張）

對比 noultaug（目前最佳）：
| | Clean FP | 瑕疵命中 |
|--|----------|---------|
| noultaug | 7/46（15%）| 22/26 |
| hybrid | **35/46（76%）** | 25/26 |

**hybrid 在真實部署中不可用**（clean 紙箱上會持續誤報）。

---

### Hybrid 失敗根本原因分析

**核心機制：加了正例，但沒加真實負例**

```
加入 5 張真實 Puncture → 模型學到「真實紙板的小暗圓 = Puncture」
加入 5 張真實 Stain    → 模型學到「真實紙板的褐色斑塊 = Stain」

未加真實 Clean ROI    → 沒有「真實紙板正常紋理 ≠ 瑕疵」的 anchor

結果：任何真實紙板上的印刷字、摺痕、紙紋都被觸發
```

**與成功 Hybrid 案例的差距（如參考影片所示）：**

| 維度 | 成功案例 | 我們的實驗 |
|------|---------|-----------|
| 真實圖佔訓練比例 | 通常 10~30% | 12/4012 = **0.3%**（統計意義不足）|
| 真實 Negative 樣本 | 一定有，且數量足夠 | **完全沒有**（只加了正例）|
| 正負例比例 | 平衡 | 嚴重失衡（正：12，負：0）|
| Synthetic 品質 | 接近真實 domain | Stain shader 已知有 domain gap |

**結論：「少量真實 Anchor」只在同時提供真實 Negative 的情況下才有效；光加正例反而讓模型 threshold 崩潰。**

---

### 修復方向

**Hybrid v2（正確做法）：**
- noaug 4000 合成 + 12 真實瑕疵（現有）+ **20~30 張真實 clean_box ROI（新增 negative anchor）**
- 真實 clean ROI 從現有 `05_Dataset_YOLO2/production_v1_5000/images/test/` 的 `Real_sample_clean_box_*.jpg` 取
- 空 label 負樣本，教模型「真實紙板紋理 ≠ 瑕疵」

**ComfyUI 改善 Stain Shader（中長期）：**
- 目標：生成具 Alpha 通道、有機形狀、顏色偏暖的真實感 Stain 貼圖取代現有 Blender decal
- 工具：ComfyUI + IPAdapter（以真實污漬照片為參考）+ ControlNet 保持幾何約束
- 輸出：20~30 張新 Stain PNG → 匯入 Blender 取代舊 decal → 重算 1000 張 stain → 重訓
- 預估工時：2~3 天（包含 ComfyUI workflow 調參、Blender 整合）
- 注意：即使 shader 改善，仍需搭配真實 negative anchor 才能避免 FP 問題

### Hybrid v2 實驗成果（2026-04-22，L1 診斷被完全驗證）

**Dataset：**
- Train 4044 = 4000 noaug 合成 ROI + 44 真實圖（12 stain + 10 puncture + 2 both + **20 clean_box negative anchor**）
- Real 佔比 1.1%（對齊 Nvidia 50 張 benchmark 的同數量級）
- Val 30 不動；Test 從 72 → 40（5 stain + 5 puncture + 4 both + 26 clean）

**訓練指標：**
- best.pt @ **epoch 92**（109 epoch 早停）— **P001 epoch-1 鎖死首次被徹底排除（非靠 HN，靠真實 anchor）**
- mAP50 = **0.89**，mAP50-95 = **0.91**（歷史新高，22× 於 noultaug baseline）

**Test 表現（`10_predict_yolo2.py --conf 0.40`）：**

| 類別 | Test | 檢出 | Recall |
|------|-----|------|--------|
| Stain | 5 | 4 | 80% |
| Puncture | 5 | 5 | 100% |
| Both | 4 | 4 | 100% |
| **Clean (FP 檢查)** | **26** | **0** | **0% FP** 🎯 |

**跨實驗對比（FP 修復核心證據）：**

| 實驗 | Clean FP 率 | Defect Hit |
|------|------------|-----------|
| noultaug（原最佳）| 15%（7/46）| 73% |
| hybrid v1（失敗）| **76%（35/46）**| 96% |
| **hybrid_v2 🏆** | **0%（0/26）** | **93%（13/14）** |

**驗證結論：**
1. ✅ L1 診斷（缺真實 clean negative anchor）完全正確，FP 76% → 0%
2. ✅ 44 張真實圖（1.1%）足以打破 P001（之前必須靠 HN 空背景）
3. ✅ 真實 anchor **比** HN 空背景更強：後者讓 best 從 ep1 → ep30，前者讓 best 從 ep1 → ep92

### 下一步部署建議（2026-04-22 更新）

- **目前最佳 YOLO2 權重：`07_Models/production_v1_hybrid_v2/yolo2/weights/best.pt`** ✨（Clean FP 0%, Defect Hit 93%）
- 舊建議 noultaug 作廢（hybrid_v2 全面勝出）
- Webcam demo 建議組合：YOLO1 `production_v1_5000/last.pt` + **YOLO2 `production_v1_hybrid_v2/best.pt`**，conf=0.50/0.40

### 實驗產出檔案位置

- 四組 predict 結果：`06_Raw_Output/predict_yolo2/{production_v1_5000, production_v1_noaug, production_v1_noultaug, production_v1_hybrid}/`
- **四組並排對比圖（baseline | noaug | noultaug | hybrid）**：`06_Raw_Output/predict_yolo2/_compare/cmp_*.jpg`（72 張）

---

## ⭐ 核心結論實驗：Synthetic Data Generation 價值量化（realonly ablation, 2026-04-22）

### 實驗動機

整個專案的論述核心是「用合成資料解決真實標記資料稀缺」。為了避免被質疑「光靠那 44 張真實圖可能就夠了」，設計 **realonly 對照組**：Train=0 合成 + 44 真實（與 hybrid_v2 完全相同的 real 子集：12 stain + 10 puncture + 2 both + 20 clean_negative），Val/Test 與 hybrid_v2 完全一致。

### 兩次跑法：不公平版 → 公平版（重要方法論修正）

**1. realonly（不公平，`--no-yolo-aug`）— 首次跑：結論作廢**

為了對齊 hybrid_v2 的 aug 條件使用 `--no-yolo-aug`，結果 val mAP50=0.111、test 0/40 — 但這**不公平**：44 張訓練資料極度依賴 mosaic/hsv/erasing 等 Ultralytics aug 才能學起來，關掉等同自殘。結論是「訓練設定不公，非合成必要性」。

**2. realonly_fair（公平版）— 採用此結果**

指令：`python 08_train.py --target yolo2 --patience 50`（移除 `--no-yolo-aug`，讓 Ultralytics 預設 aug 全開）

| 指標 | realonly_fair | hybrid_v2 🏆 | 差距 |
|------|---------------|-------------|------|
| Train 規模 | 44 真實 | 4000 合成 + 44 真實 | — |
| Best epoch | **113**（163 ep 早停）| 92（109 ep 早停）| 都排除 P001 鎖死 |
| Val mAP50 | **0.69** | **0.89** | +0.20 |
| Val Puncture mAP50 | **0.835** | — | Puncture 幾乎追平 |
| Val Stain mAP50 | 0.546 | — | — |
| Val Precision | 0.806 | — | 不再亂框 |
| Test Stain Hit | 2/5 (40%) | **4/5 (80%)** | +40pp |
| Test Puncture Hit | **5/5 (100%)** | 5/5 (100%) | 0 |
| Test Both Hit | 3/4 (75%) | **4/4 (100%)** | +25pp |
| **Test Defect Hit 總計** | **10/14 = 71%** | **13/14 = 93%** | **+22pp** |
| **Test Clean FP** | **6/26 = 23%** | **0/26 = 0%** | **−23pp** |

### 完整五組對照總表（專案故事線核心證據）

| 實驗 | Train 組成 | Aug | Val mAP50 | Test Defect Hit | Test Clean FP | 結論 |
|------|-----------|-----|-----------|-----------------|---------------|------|
| **realonly_fair** | 0 合成 + 44 真實 | 全 ult aug | **0.69** | 10/14 = 71% | **23%** | ⚠️ Puncture 可做但 Stain 弱 + FP 高 |
| noaug | 4000 合成 + 0 真實 | 僅 ult aug | 0.38（Punct 0.76）| 4/84 | — | 純合成 val 過擬合 |
| noultaug | 4000 合成 + 0 真實 | 僅 cropper aug | ~0.21 | 40/84 | 中 | 純合成最佳部署 |
| hybrid v1 | 4000 合成 + 12 真實（無 anchor）| 無 ult aug | ~0.06 | 25/26 | **76%** ❌ | 加正例無負例→threshold 崩潰 |
| **hybrid_v2** 🏆 | 4000 合成 + 44 真實（含 20 anchor）| 無 ult aug | **0.89** | **13/14 = 93%** | **0/26 = 0%** | ✅ Nvidia 方法論完整復現 |

### 誠實結論（修正敘事）

**舊敘事（過度戲劇化，已作廢）**：「沒合成資料 → 0/40，合成是必要條件」

**新敘事（採用此版）**：
> 44 張真實圖 + 強 augmentation **可達到原型水準**（mAP50=0.69、Puncture 100% hit）；但 Stain recall 僅 40%、Clean FP 高達 23%，不可直接部署。
> 加入 4000 合成資料（hybrid_v2）後：**Stain hit 翻倍（40%→80%）、Defect 總命中 +22pp、Clean FP 歸零（23%→0%）、mAP50 提升 0.20**。
>
> **合成資料的價值不是「從零到一」，而是「把 50 張級別的原型模型推向部署級精度」** — 這反而更貼近 Nvidia SDG 的真實論述（他們也承認 25/50 張純真實失敗才走合成路線）。

### Nvidia 方法論對照

| 維度 | Nvidia 官方 | 本專案 |
|------|------------|--------|
| 合成資料 | 1500 張 | 4000 張 |
| 真實資料 | 50 張 | 44 張 |
| 真實佔比 | 3.3% | 1.1% |
| 方法 | 兩階段 fine-tune | Single-stage hybrid |
| 結論 | 純 50 真實失敗，合成預訓練救命 | 純 44 真實有限可用（Puncture 100% / Stain 40%），合成 + negative anchor 達部署級 |

### 對論文 / README / YouTube 的意義

- ✅ **SDG 價值量化有硬數據**：+22pp defect hit、−23pp FP、+0.20 mAP50、Stain hit 翻倍
- ✅ **敘事更誠實更經得起挑戰**：有經驗的讀者不會被「0/40 必要性」這種過度戲劇化的數字嚇退
- ✅ **完整 5 組 YOLO2 ablation**：realonly_fair / noaug / noultaug / hybrid v1 / hybrid_v2 覆蓋所有變因
- ✅ **Nvidia 方法論獨立復現**：配比、結論、失敗模式都對得上，可直接引用比較

### 部署建議（不變）

~~最佳組合：`YOLO1=production_v1_5000/last.pt` + `YOLO2=production_v1_hybrid_v2/best.pt`~~（過時，2026-04-22 升級）

**最終最佳組合（2026-04-22）**：`YOLO1=production_v2_HN/best.pt` + `YOLO2=production_v1_hybrid_v2/best.pt`，conf=0.50/0.40

---

## ⭐ YOLO1 production_v2_HN 結果小結（2026-04-22，研究全部收官）

### 實驗動機

production_v1_5000 的 YOLO1 雖然 last.pt 可用（109/114 推論），但 best.pt 被 epoch 2 鎖死（P012）。需驗證「**HN 在 5000+ 規模是否仍能排除 P001**」以完成 P001 的 2×2 ablation：
- 500 張 / 無 HN：smoke_test_v1（鎖死 ep1）
- 500 張 / +HN：smoke_test_v2_HN（排除 ep30）
- 5000 張 / 無 HN：production_v1_5000（鎖死 ep2）
- **5000 張 / +HN：production_v2_HN ← 本次補完**

### Dataset 規格

| Split | 來源 | 張數 |
|-------|------|------|
| Train | 6427 合成 + **22 真實 HN 空背景**（空 label）| **6449** |
| Val | 與 v1_5000 相同 | 30 |
| Test | 與 v1_5000 相同 | 84 |

訓練指令：`python 08_train.py --target yolo1 --patience 20`

### 訓練結果（**P001 完整排除**）

| 指標 | v1_5000（無 HN）| **v2_HN（+ 22 HN）** |
|------|----------------|---------------------|
| Best epoch | **2** 🔒（鎖死）| **10** ✅（排除）|
| 訓練停止 | 22 ep | 33 ep |
| Best fitness | ~0.95 | **0.995** |
| Val mAP50 (best) | — | **0.998** |
| Val mAP50-95 (best) | — | **0.948** |
| Test (best.pt) | 偏 COCO pretrain，多檢出但框鬆 | **79/84 = 94%** ✨ |
| Test (last.pt) | 109/114（best 鎖死，部署優先 last）| 75/84 = 89% |

**首次出現 best.pt > last.pt** — 三組 YOLO1 ablation 的最終封印。

### 三段 P001 ablation 完整對齊

| 規模 | 無 HN | +HN | HN 解 P001 |
|------|------|-----|-----------|
| 500 張 | best=ep1 🔒 | best=ep30 ✅ | ✅ 確認 |
| **5000+ 張** | best=ep2 🔒 | **best=ep10 ✅** | ✅ 確認（規模無關）|

**結論**：P001 的根本解法是「擾動 val 分布」(HN / negative anchor)，不是擴大資料量。資料量從 500→6427（13×）無法排除鎖死，22 張 HN（0.34%）卻能排除。**HN 是規模無關的解**。

### 部署影響

- **舊**：`YOLO1 = production_v1_5000/last.pt`（best 鎖死，被迫用 last）
- **新**：`YOLO1 = production_v2_HN/best.pt`（94% test recall，best > last）
- 待更新：`11_realtime_demo.py` 的 `YOLO1_EXP` 從 `production_v1_5000` → `production_v2_HN`

### 對論文 / README 的意義

- ✅ **P001 章節有了 2×2 完整 ablation**，無漏洞
- ✅ **HN 解 P001 規模無關**這個結論非常乾淨，可獨立成 README 三大發現之一
- ✅ **整個專案研究實驗 100% 收官**，主線三條結論（P001 / 雙層 aug / negative anchor）全部交叉驗證

---

## 🗓️ 迭代歷程與參數歸因（Iteration Timeline）

> 完整時間線：每次實驗改了什麼、為什麼、產生什麼後果。供報告寫作 / 面試引用。

### 時間線總表

| 日期 | Commit | 實驗 / 里程碑 | 主要改動 | 觸發動機 | 結果 |
|------|--------|--------------|---------|---------|------|
| 04-14 | 4a5fe41 | Initial commit | Pipeline 50 張 prototype | 概念驗證 | 流程跑通 |
| 04-14 | 04bc18d | refactor 01-05 | 50 張轉 500 張 smoke test 結構 | 規模化前準備 | 路徑/輸出格式定型 |
| 04-15 | 58f5ecf | **Sim-to-Real 增強導入** | `05_cropper_yolo2.py` 加入 augment_yolo2()（冷色 / 對比 / blur / JPEG / noise）| 對齊 webcam 真實 domain | 後成為雙層 aug 隱患（P013）|
| 04-15 | 4af9651 | **Real dataset pipeline** | val/test 從 real_refs 寫入；`08_train.py` 統一訓練腳本 | 準備驗證真實表現 | val 集 40 張規模偏小（P004）|
| 04-18 | 289be42 | YOLO1 500 張 smoke test 完成 | 拆分 09/10 predict scripts | 驗證 YOLO1 對真實紙箱效果 | **P001 epoch-1 鎖死首次發現**（best 0/114, last 109/114）|
| 04-18 | **453d3e2** | **🔧 Stain shader + P1-P4 大整修** | 6 個關鍵參數變動（見下表）| 100 張 QA 視覺檢查通過 | 為 production_v1_5000 鋪路 |
| 04-19 | fac1a2c | **🔧 P009 數量控制 + 路徑統一** | `SAFETY_MARGIN=1.30` + `EXPERIMENT_NAME` 常數化 | mask 過濾損耗導致缺口 | 各類別精準達標 1000 張 |
| 04-19 | a7b2c16 | 切換到 production_v1_5000 | 全腳本同步 EXPERIMENT_NAME | 正式 5000 張啟動 | 5900 張算圖開始 |
| 04-19 | cc2831b | val/test 解耦 + 算圖計時 | 從 test 抽 30 為 val（val ⊄ test）| 解 P001 三因素之一 | val=30, test=84 |
| 04-21 | f38c951 | **YOLO2 雙 aug ablation** | `08_train.py` 加 `--no-yolo-aug` flag | 假設 P013（雙層 aug 害死 Puncture）| ✅ 驗證：noaug Punct mAP50 0.10→0.76 |
| 04-22 | 2a76c92 | **Hybrid v1（失敗案例）** | noaug 4000 + 12 真實正例（無 anchor）| 引入 real domain | ❌ Clean FP=76% threshold 崩潰 |
| 04-22 | 2330393 | **Hybrid v2（最終突破）+ realonly** | hybrid v1 + **20 真實 clean negative anchor** + realonly_fair 對照 | hybrid v1 失敗反推根因 | ✅ mAP50=0.89, Clean FP=0% |
| 04-26 | e82b292 | **YOLO1 production_v2_HN** | 6427 合成 + 22 HN 空背景 | 補完 P001 2×2 ablation | ✅ best@ep10, test 79/84=94% |
| 04-26 | 740a125 | Apple-to-apple 對比腳本 | 5 組 best.pt × 共同 40 張 test | 解 L2 test 集大小不一致 | 5-panel 並排對比圖（README hero shot）|

---

### 重大改動詳記（含具體參數眉角）

#### 🔧 Iter A：Stain Shader + P1-P4 大整修（04-18, 453d3e2）

**動機**：500 張 smoke test 後 YOLO2 Stain recall=0%，Blender Stain shader 與真實污漬差距太大；同時 100 張 visual QA 暴露多個小毛病。

**6 個並行修復**：

| 檔案 | 改動 | 原因 |
|------|------|------|
| Blender 場景（手動）| Stain shader 重調為有機水漬形態 | 純合成 Stain 與真實外觀差距大 |
| `05_cropper_yolo2.py` | `MORPH_CLOSE` Stain `5×5 → 7×7` | 連接跨面 Stain 碎片，避免單一污漬被切多框 |
| `05_cropper_yolo2.py` | JPEG quality `72 → 82` | **P008**：72 在暗部產生假紋理可能讓模型誤學 |
| `01_orchestrator.py` | Puncture scale 下限 `0.20 → 0.25` | **P003**：過小 BBox（<25px）推論 conf 極低，無訓練意義 |
| `01_orchestrator.py` | 新增 `bg_strength: 0.8~1.5` 隨機 | 防止過暗 HDRI 造成全黑場景 |
| `02_blender_render.py` | Puncture decal 強制 `Non-Color` colorspace | **P006**：預設 sRGB 讓深色破口被 gamma curve 扭曲 |
| `02_blender_render.py` | 修 `bg_strength` 多幀累乘 bug | **P007**：未還原原始值導致逐幀亮度漂移 |

**驗收**：100 張 visual QA 通過，Stain BBox 精準、跨面碎片連起來。

**為什麼一次改 6 個是 OK 的**：這些都是「防禦性修復」（修 bug 不是調效果），單獨變動無法評估，整體驗收即可。**做效能比較的改動才該獨立 ablation**。

---

#### 🔧 Iter B：P009 數量控制 + 路徑統一（04-19, fac1a2c）

**動機**：100 張 QA 試跑時各類別實際入庫數量低於目標（stain 16/20、puncture 14/20）— 因為 mask 面積過濾把部分幀篩掉。

**修復**：

| 檔案 | 改動 |
|------|------|
| `01_orchestrator.py` | 新增 `SAFETY_MARGIN = 1.30`；defect 類別生成 `ceil(num × 1.30)` 份 config，多渲染 30% 備用 |
| `04_annotator_yolo1.py` | 新增 `EXPERIMENT_NAME` 常數，輸出版本化路徑 `04_Dataset_YOLO1/{EXP}/` |
| `05_cropper_yolo2.py` | 同上版本化 + 結束摘要表（缺口 > 0 觸發 ⚠️）+ `configs_seen` / `reclassified` 計數 |
| 自動產生 `data.yaml` | 路徑與 `08_train.py` 一致 |

**眉角**：`SAFETY_MARGIN` 不是隨便取 1.30 — 是觀察 yield 率 ~77% 後留 buffer 的數字。Puncture 後續實測 yield 僅 54.9%（P011），需用 `FRAME_START_OFFSET` 增量補跑。

---

#### 🔧 Iter C：YOLO2 雙 aug Ablation（04-21, f38c951）

**動機**：production_v1_5000 跑出來 YOLO2 Puncture recall 從 smoke v2 的 79% 倒退到 25%（P013）。檢查 `args.yaml` 發現 Ultralytics 預設 aug 全開（mosaic=1.0, hsv_s=0.7, erasing=0.4），疊在 cropper 的 Sim-to-Real 增強之上。

**修復**：`08_train.py` 加入 `--no-yolo-aug` flag，對 YOLO2 覆寫：
```python
mosaic        = 0.0   # 關 4 圖拼貼（ROI 小圖不適合）
erasing       = 0.0   # 關隨機遮蔽
hsv_s         = 0.3   # 降彩度擾動（cropper 已做冷色偏移）
hsv_v         = 0.2   # 降明度擾動
auto_augment  = ""    # 關 randaugment
```

**設計兩組 ablation**：
- `noaug`：建立新 dataset（cropper aug 關），Ultralytics aug 保留
- `noultaug`：沿用 v1_5000 dataset（cropper aug 保留），Ultralytics aug 關（用 `--no-yolo-aug`）

**結果**：noaug Puncture mAP50=0.76（7.6×）；noultaug test 40/84（純合成最佳）— **單獨任一層移除都顯著改善**。

---

#### 🔧 Iter D：Hybrid v1 失敗反推到 v2（04-22, 2a76c92 → 2330393）

**v1 假設**：Nvidia 方法論「合成 + 少量真實 fine-tune」，加 12 張真實正例（5 stain + 5 punct + 2 both）到 noaug 4000 train。

**v1 結果（誤導的成功）**：
- Test 72/84 hits（看似最高）
- 但深入分析：Clean 圖誤框 35/46 = **76%**，瑕疵命中只比 noultaug 多 1 張
- **100% 假象，模型實際是「亂槍打鳥」**

**v1 → v2 根因分析**：
> 加入 5 張真實 Puncture → 模型學到「真實紙板的小暗圓 = Puncture」
> 加入 5 張真實 Stain → 模型學到「真實紙板的褐色斑塊 = Stain」
> **但沒加真實 Clean ROI** → 沒有「真實紙板正常紋理 ≠ 瑕疵」的 anchor
> → 任何真實紙板上的印刷字、摺痕、紙紋都被觸發

**v2 修復**：在 v1 基礎上多加：
- 12 → 24 真實正例（從 test 多挖 7 stain + 5 punct）
- **+20 張真實 clean_box ROI（空 label，negative anchor）**
- 真實佔比 0.3% → 1.1%（對齊 Nvidia 50 張級別）

**v2 結果**：mAP50=0.89, Clean FP=0%, Defect Hit 93%。**單一變數加 20 張 anchor，FP 從 76%→0%**。

---

#### 🔧 Iter E：realonly 不公平 → realonly_fair（04-22, 2330393）

**realonly v1（不公平）**：為對齊 hybrid_v2 的 aug 條件用 `--no-yolo-aug`，結果 val mAP50=0.111, test 0/40。

**用戶質疑**：「Nvidia 50 張能成功，44 張不應該 0/40」 → 我承認對 44 張關 aug 等同自殘。

**realonly_fair（公平版）**：移除 `--no-yolo-aug`，讓 Ultralytics 預設 mosaic / hsv / erasing / randaugment 全開。

**結果**：val mAP50=0.69（Punct 0.835 / Stain 0.546），test 10/14 defect / 6/26 Clean FP — **44 張真實 + 強 aug 達原型水準，但 Stain 弱 + FP 偏高**。

**敘事修正**：
- 舊：「沒合成 → 0/40，合成必要」（過度戲劇化）
- 新：「合成把 50 張原型推到部署級」（誠實且更符合 Nvidia 原述）

**這次反覆是專案最重要的方法論成熟時刻**。

---

#### 🔧 Iter F：YOLO1 production_v2_HN 收官（04-26, e82b292）

**動機**：production_v1_5000 的 YOLO1 best@ep2 鎖死，待補完 P001 的 2×2（500/5000 × 無HN/+HN）的最後一格。

**改動**：複製 v1_5000 dataset 結構，加 22 張 `01_Assets/real_refs/0408/empty_background/` 進 train（空 label）。

**結果**：best@ep10（fitness 0.995, mAP50=0.998）、test 79/84=94%、首次 best > last。

**關鍵 takeaway**：**規模 13× 解不了 P001，HN 0.34% 卻能解** — 訓練分布形狀比體積更重要。

---

### 跨實驗參數摘要表（各眉角的最終設定）

| 元件 | 最終值 | 演進歷程 |
|------|--------|---------|
| YOLO1 patience | 20 | smoke v1 用 50 → 被 ep1 鎖死綁架到 ep51；v1_5000 起改 20 |
| YOLO2 patience | 20-50 | v1_5000=20、hybrid 也 20、realonly_fair=50（44 張需更長練習）|
| Cropper Sim-to-Real aug | 啟用（noultaug 模式）| noaug 對比後確定 cropper aug 對 Puncture 仍有價值 |
| Ultralytics aug | YOLO1 開、YOLO2 視情況 | YOLO2 開時打壞細微特徵（P013），合成資料訓練建議 `--no-yolo-aug` |
| ROI padding | **0%（tight crop）**| smoke v1 用 5%；smoke v2 改 tight 後 background→Stain FP 從 35%→0% |
| Stain mask threshold | 8 | 過低噪點被框，過高跨面 stain 被切；threshold=8 + MORPH 7×7 平衡點 |
| Puncture mask threshold | 25 | Puncture 邊緣銳利，threshold 較高仍可保完整 |
| MORPH_CLOSE Stain kernel | 7×7 | 5×5 連不起跨面碎片；7×7 為平衡點 |
| MORPH_CLOSE Puncture kernel | 5×5 | Puncture 形狀單純不需大 kernel |
| Min bbox 過濾 | 面積 < 60 或邊長 < 6 | 過低噪點入庫、過高小 Puncture 被丟 |
| JPEG quality（Sim-to-Real）| 82 | 從 72 提高，避免暗部假紋理（P008）|
| 冷色偏移 | R-6, B+10 | 對齊 Acer Nitro AN515 720p webcam（無 AWB）|
| Gaussian noise std | 4 | 模擬基礎 CMOS 感光元件噪點 |
| Webcam demo CONF_YOLO1 | 0.50 | 過濾低信心紙箱框 |
| Webcam demo CONF_YOLO2 | 0.40 | 過濾 noultaug FP（conf=0.30 時 FP 翻倍）|
| Puncture scale 下限 | 0.25 | 從 0.20 提高，避免 mask 過濾失敗（P003）|
| HDRI bg_strength | 0.8~1.5 隨機 | 從固定值改隨機，防黑場景（P004）|
| Puncture colorspace | Non-Color | 修 P006，避免 sRGB gamma 扭曲 |
| Render SAFETY_MARGIN | 1.30 | 補償 mask 過濾損耗的緩衝（P009 + P011 微調）|

---

## 📋 全專案結論清單（README 素材庫）

> 整理所有實驗交叉驗證後得到的結論，按重要性排序。供 README / Medium / 論文摘要直接使用。

### 主結論（README 三大發現）

**1. 真實 Negative Anchor 是 hybrid 成敗關鍵（不是正例多寡）**
- hybrid v1（4000 合成 + 12 真實正例，無 anchor）→ Clean FP **76%**（threshold 崩潰）
- hybrid_v2（4000 合成 + 12 正例 + **20 真實 clean anchor**）→ Clean FP **0%**
- 差別只有 20 張 clean ROI，效果從「不可部署」到「生產級」
- **業界 SDG 教學少提此細節，但決定 hybrid 能否部署**

**2. P001 epoch-early 鎖死的根本解法是「擾動 val 分布」，不是資料量**
- 4 組 YOLO1 + 5 組 YOLO2 對照：500→6427 張（13×）解不了；22 張 HN（0.34%）解
- HN 是規模無關的解；資料量擴大無效
- 一句話：**「best.pt 不可用時，先加 HN，不要加資料」**

**3. SDG 的價值是「把原型推向部署級」，不是「從零到一」**
- 純實拍 44 張（realonly_fair）：mAP50=0.69，Defect 71%，Clean FP 23% — **原型可用**
- 加 4000 合成（hybrid_v2）：mAP50=0.89，Defect 93%，Clean FP 0% — **部署級**
- 量化效益：+22pp defect、−23pp FP、+0.20 mAP50、Stain hit 翻倍

### 次級結論（Medium 長文 / 論文 insights）

**4. 雙層 augmentation 疊加會打壞細微特徵**
- Pipeline aug + Ultralytics aug 雙層 → val Puncture mAP50 **0.10**
- 任一層拿掉 → 0.385 / **0.76**（最高 7.6×）
- **多數 SDG 教學忽略「pipeline 內已有 aug 時應關掉框架預設 aug」這層**

**5. Sim-to-Real gap 不均勻分布於不同瑕疵類型**
- **Puncture（圓形深色破口）**：純合成可達 79% recall — shader 能表達好的特徵
- **Stain（半透明色塊）**：純合成 0% recall — shader 表達力不足
- **結論不是「合成有用 vs 沒用」，是「哪些瑕疵適合純合成取決於 shader 對該特徵的可表達性」**
- 工程啟示：先做 Stain/Puncture-like 的 shader 可表達性盤點，再決定資料策略

**6. 訓練分布形狀 > 訓練資料體積**
- 500→6427 張（13×）解不了 P001
- 22 張 HN（0.34% 佔比）卻能解
- 對照組：500 + 22 HN（4.4% HN 佔比）已解；5000 + 0 HN 仍未解
- **「擾動分布」比「加大訓練量」更有效率**

**7. Real domain anchor 比 cross-domain HN 更強（hybrid 比純 HN 升一級）**
- HN（real empty background）：YOLO1 best epoch 1→30
- Real anchor（real clean cardboard ROI）：YOLO2 best epoch 1→92（更晚才停）
- 兩者都解 P001，但 real anchor 因為與 train 分布更接近，效果更深

### 工程經驗結論

**8. 部署優先用 last.pt（除非 best.pt 已驗證不被 P001 鎖死）**
- 8 組無 HN/anchor 實驗：best.pt 全部不可用
- 3 組 + HN/anchor 實驗：best.pt 終於可用（且優於 last.pt）

**9. 5% padding 在 ROI cropping 中可能害多於利**
- v1 加 padding 引入紙箱邊緣 + 地面紋理 → background→Stain FP=35%
- v2 改 tight crop（無 padding）→ background→Stain FP=0%
- 教訓：訓練/推論 ROI 分布應嚴格鏡像，padding 增加 domain gap

**10. mask-to-bbox 過濾雷區**
- Stain mask threshold 過低 → 噪點被框（碎片化）
- threshold 過高 → 跨面 stain 被切
- `MORPH_CLOSE` kernel 太小無法連接跨面碎片，太大則框過大
- 各類別需獨立調參（Stain 7×7 / Puncture 5×5）

---

## ⚠️ 計畫弱點清單（Limitations，README 必揭露）

> 反 challenge 的關鍵：先承認 limitations 比讓 reviewer 挑出來高明 10 倍。
> 以下按嚴重度分類，務必寫進 README 的 Limitations 章節。

### 🔴 Critical（必承認）

**L1. Val 集只有 30 張，統計上不穩定**
- mAP 對單張表現極敏感
- noaug 的 val Puncture mAP50=0.76 卻 test 4/84，嚴重 val/test 不一致
- 學術 reviewer 會直接指出 "30-image val is statistically meaningless"

**L2. 不同實驗的 test 集大小不一致**
- baseline / noaug / noultaug：test = 84
- hybrid v1：test = 72（從 84 移 12 真實正例到 train）
- hybrid_v2 / realonly_fair：test = 40（從 84 移 44 真實圖到 train）
- **不同 test set 規模做數字對比有 apples-to-oranges 風險**
- **解決**：用 5 組 test 集的交集（= 40 張，恰好是 hybrid_v2/realonly 的 test 集）重新跑所有 best.pt，得到嚴格對齊的 apple-to-apple 對比 ↓

### Apple-to-Apple 對比（5 組 best.pt × 共同 40 張 test）

40 張交集組成：14 defect（5 stain + 5 punct + 4 both）+ 26 clean

`02_Scripts/_compare_intersection_yolo2.py` 一次性對比腳本，輸出 5-panel 並排到
`06_Raw_Output/predict_yolo2/_compare_intersection_40/`（40 張）。

| 實驗 | Defect Hit | Clean FP | Verdict |
|------|-----------|----------|---------|
| realonly_fair | 10/14 = 71% | 6/26 = 23% | 可用但 FP 偏高 |
| noaug | **0/14 = 0%** | 0/26 = 0% | ⚠️ 純合成 + 雙層 aug + val 過擬合 → 模型完全不 fire |
| **noultaug** | **12/14 = 86%** | **2/26 = 8%** | ✅ 純合成最佳，可作為 SDG 真實基線 |
| hybrid v1 | 14/14 = 100% | **16/26 = 62%** | ❌ 100% recall 是假象，FP 爆炸不可部署 |
| **hybrid_v2** 🏆 | **13/14 = 93%** | **0/26 = 0%** | ✅ 唯一同時通過「高 hit + 低 FP」的模型 |

**Apple-to-apple 修正後的關鍵更新：**
- 之前在 84 張上 noultaug 寫「40/84 = 48%」是因為 84 張中有 30 張是 hybrid_v2 train 用過的 anchor + 真實正例（noultaug 從沒見過、實際很難）
- 在交集 40 張（noultaug 也沒見過）上實測 86% — **純合成的真實能力比之前估計的高得多**
- 但 noultaug 的 8% Clean FP > hybrid_v2 的 0% — **hybrid 的價值仍清楚展現**

**這就是 L2 的解法：對所有比對改用 40 張交集的數字，避免 apples-to-oranges 質疑。**

**L3. 沒有 multiple seeds 平均**
- 單 seed 結果可能有 ±0.05 mAP 隨機性雜訊
- 學術界要求 3-5 seed 平均才能宣告差異顯著
- hybrid_v2 vs noultaug 從 0.21→0.89 看似巨大，但無 confidence interval

**L4. 缺乏統計顯著性檢定**
- 所有實驗對比都是 point estimate
- 沒有 t-test / bootstrap CI / p-value
- 需 Future Work 補做

### 🟡 Significant（可在 Future Work 帶過）

**L5. 沒做跨 domain 驗證**
- 訓練 + 測試都是同一台 webcam（Acer Nitro AN515 720p）+ 同一批紙箱
- 換相機品牌 / 紙箱類型表現未知
- Sim-to-Real 結論在 cross-device 上是否成立未驗證

**L6. 單一架構（YOLOv8n），未測 v8s/v8m/v8l**
- 結論可能對小模型敏感
- "你的結論在 YOLOv8s 上還成立嗎" 是合理 challenge

**L7. Ch.7 Two-Stage Fine-Tune 沒做**
- Nvidia 官方推薦方法（合成預訓練 → freeze + low-LR 真實 FT）
- 本專案用 single-stage hybrid 達到 0.89，**未驗證是否劣於 / 等同 / 優於 two-stage**
- Reviewer 會問「為什麼選 single-stage hybrid 而非 Nvidia 兩階段」

**L8. Augmentation 參數憑直覺，未做敏感度 sweep**
- 冷色偏移（R-6, B+10）/ JPEG quality 82 / GaussianBlur ksize=3 / noise std=4
- 都沒做 grid search 證明是最佳值
- 可能 0.89 還能更高，但沒探到

### 🟢 Minor（可寫可不寫）

**L9. Stain shader 沒從根本解決，靠 anchor 救場**
- 三組純合成 Stain recall = 0%，加 12 張真實 anchor 才達 80%
- 暴露合成資料對「半透明有機紋理」的能力上限
- 解法在 ComfyUI / Diffusion-augmented 方向，本研究未探

**L10. 沒測 deployment metrics**
- 推論延遲（FPS）、模型大小（MB）、GPU 記憶體佔用都沒記錄
- 工業面試官在意此數據（特別是 edge AI 職位）

### 防禦話術範本（求職/論文可直接用）

> 「Val/Test 集小是這個專案最大限制；30/84 的數據量讓單張結果產生較大方差。
> 我選擇接受這個取捨而非花時間蒐集更多真實資料，因為本研究的核心問題是
> 『合成資料能多大程度替代真實資料』，而非建立 production benchmark。
> Future work 包含 multi-seed 平均、跨相機驗證、以及 YOLOv8s/m 架構 sensitivity。」

主動講出 limitations 會讓你從「沒做完」變成「方法論成熟」。Reviewer 最怕看到沒意識到自己 limitation 的 candidate。

---

## 目前進度（截至 2026-04-21，YOLO2 aug ablation 完成，下一步 Webcam demo）

### 流水線腳本

| 步驟 | 狀態 |
|------|------|
| 00_scene_health_check.py | **✅ 完成** |
| 01_orchestrator.py | **✅ 完成（production_v1_5000 設定：DISTRIBUTION 5000 張 + SAFETY_MARGIN 1.30）** |
| 02_blender_render.py | **✅ 完成（三 Pass，640×640）** |
| 03_render_operator.py | **✅ 完成（含 config 驗證、UTF-8 fix）** |
| 04_annotator_yolo1.py | **✅ 完成（無增強，備份至 images_ready/yolo1）** |
| 05_cropper_yolo2.py | **✅ 完成（findNonZero 跨面修復 + Sim-to-Real 增強 + 備份，padding 停用）** |
| 06_build_dataset.py | **⚠️ 已部分取代**（val/test 實拍圖已由內嵌腳本直接寫入，原腳本邏輯保留備用）|
| 07_augment_test.py | **✅ 完成（視覺測試工具，增強參數待 YOLO fine-tune 後微調）** |
| 08_train.py | **✅ 完成（統一訓練腳本，--target yolo1\|yolo2）** |
| 09_predict_yolo1.py | **✅ 完成（YOLO1 單段推論 → predict_yolo1/）** |
| 10_predict_yolo2.py | **✅ 完成（YOLO2 單段推論 → predict_yolo2/）** |
| 11_realtime_demo.py | **✅ 完成（Webcam 雙段即時 demo）** |
| 12_qa_real_dataset.py | **✅ 完成（實拍 val/test QA 視覺化）** |

### 渲染 & 資料生成

| 步驟 | 狀態 |
|------|------|
| Blender 算圖（50 張驗證）| **✅ 畫面品質確認通過** |
| Blender 算圖（500 張測試）| **✅ 完成（501 張已存於 06_Raw_Output/images/）** |
| YOLO1 合成 train 標注 | **✅ 完成（501 張全圖 BBox）** |
| YOLO2 合成 train 裁切 + 增強 | **✅ 完成（197 張 ROI，147 有瑕疵 label，50 clean 空 label）** |
| Blender 算圖（5900 張正式，production_v1_5000）| **✅ 完成（6450 張，含補跑 puncture 550 張，2026-04-21）** |

### 實拍資料整理

| 步驟 | 狀態 |
|------|------|
| 實拍圖分類（5 類 × images/ + labels/）| **✅ 完成（136 張）** |
| Class remap（Roboflow 0↔2 → 統一格式）| **✅ 完成** |
| data.yaml（YOLO1 / YOLO2）| **✅ 完成** |
| YOLO1 val/test 填入 | **✅ 完成（val 40 張 / test 114 張）** |
| YOLO2 val/test 填入（ROI crop）| **✅ 完成（val 40 張 / test 114 張）** |
| 實拍 QA 視覺化（yolo1_real / yolo2_real）| **✅ 完成（308 張）** |

### 訓練 & 推論

| 步驟 | 狀態 |
|------|------|
| Python venv + CUDA torch + ultralytics | **✅ 完成（2026-04-17，詳見「開發環境」章節）** |
| YOLO1 訓練（500 張 smoke test） | **✅ 完成（51 epoch 早停，last.pt 109/114 檢出，詳見小結）** |
| YOLO1 test 集推論（09_predict） | **✅ 完成（last.pt + 粗框/大字 demo overlay）** |
| YOLO2 訓練（500 張 smoke test） | **✅ 完成（詳見 YOLO2 小結；Puncture 74%，Stain 0%，Background FP 高）** |
| YOLO2 test 集推論（10_predict） | **✅ 完成（目視評估：Puncture 合理，Stain 碎片化）** |
| smoke_test_v2_HN Dataset 建置 | **✅ 完成（YOLO1 train 522 含 22 HN / YOLO2 train 196 純合成 / tight val/test）**|
| smoke_test_v2_HN 訓練（YOLO1）| **✅ 完成（best.pt=epoch30，鎖死排除；mAP50=0.968）**|
| smoke_test_v2_HN 訓練（YOLO2）| **✅ 完成（Puncture 79%↑，Stain 仍 0%，背景 Stain FP 消除）**|
| smoke_test_v2_HN 推論（09+10 predict）| **✅ 完成（predict 圖已產出，目視評估完成）**|
| **Blender Stain shader 調整** | **✅ 完成（2026-04-18，材質重調後 100 張視覺 QA 通過）**|
| **100 張視覺 QA（新 shader + P1-P4 優化）** | **✅ 完成（Stain 有機形狀通過驗收，BBox 精準）**|
| **Pipeline 數量控制 + 路徑一致性修復（P009）** | **✅ 完成（2026-04-19，SAFETY_MARGIN + EXPERIMENT_NAME + 摘要表）**|
| Blender 算圖（production_v1_5000）| **✅ 完成（6450 張，含補跑 550）** |
| 04_annotator_yolo1.py（production_v1_5000）| **✅ 完成（YOLO1 train = 6427）** |
| 05_cropper_yolo2.py（production_v1_5000）| **✅ 完成（YOLO2 train = 4000，四類全滿）** |
| Val/Test 解耦（抽獨立 val 子集）| **✅ 完成（val=30 / test=84，val ⊄ test 驗證通過，data.yaml 已建）** |
| 正式訓練 YOLO1（production_v1_5000）| **✅ 完成（2026-04-21，22 epoch 早停，best@2 鎖死，last.pt 可用）** |
| 正式訓練 YOLO2（production_v1_5000）| **✅ 完成（2026-04-21，21 epoch 早停，best@1 鎖死，last.pt 優於 best）** |
| production_v1_5000 推論（09+10 predict）| **✅ 完成（best/last 對比完畢，詳見 production_v1_5000 訓練結果小結）** |
| Webcam 即時 demo（11_realtime_demo.py）| **🎯 下一步（Path A，conf 0.5/0.45，用 last.pt）** |
| **production_v2：加入 HN 負樣本對比實驗** | **⏳ v1 結果出爐後執行，量化 HN 對 5000 張規模的效益** |
| 所有實驗 predict test 圖疊圖對比 | **⏳ v1 訓練完成後（v1 / v2 / smoke_test_v2_HN）** |
| 增強參數微調（依 val mAP 回頭調整）| **⏳ 待 v1 訓練結果** |
| 飄浮模式 Dataset 實驗 | **⏳ 取得 v1 mAP 基準後評估** |
| 即時雙段推論測試（11_realtime_demo.py）| **🎯 下一步（與 Webcam demo 同一任務）** |

---

## 當前 Dataset 實際數量（2026-04-19）

### smoke_test_v2_HN（已完成，保留供對比）

| Dataset | Split | 圖片 | Labels | 備註 |
|---------|-------|------|--------|------|
| YOLO1 | train | **522** | 522 | 500 合成全圖 + **22 HN 空背景**（class 0 = Box）|
| YOLO1 | val | 40 | 40 | 20 clean + 20 defect（全 class 0）|
| YOLO1 | test | 114 | 114 | 全部實拍（全 class 0）|
| YOLO2 | train | 196 | 146 | 146 有瑕疵 / 50 clean（空 label，檔案不建立）|
| YOLO2 | val | 40 | 40 | 20 clean 空 label + 20 defect ROI（tight crop）|
| YOLO2 | test | 114 | 114 | 62 clean 空 label + 52 defect ROI（tight crop）|

### production_v1_5000（✅ Dataset 完全就緒）

| Dataset | Split | 張數 | 狀態 | 備註 |
|---------|-------|------|------|------|
| YOLO1 | train | **6427** | ✅ 完成 | 6450 合成 - 22 mask 失敗 - 1 無輪廓 = 6427（yield 99.6%）|
| YOLO1 | val | **30** | ✅ 已就緒（val ⊄ test）| both=2, clean_box=16, puncture=5, stain=7 |
| YOLO1 | test | **84** | ✅ 已就緒 | both=6, clean_box=46, puncture=15, stain=17 |
| YOLO2 | train | **4000** | ✅ 完成（四類各 1000，缺口=0）| 含補跑 550 張 puncture |
| YOLO2 | val | **30** | ✅ 已就緒（val ⊄ test）| 與 YOLO1 val 同一批，tight crop |
| YOLO2 | test | **84** | ✅ 已就緒 | 與 YOLO1 test 同一批，tight crop |

---

## 下一步行動計劃（smoke_test_v2_HN 完成後）

### Phase A：Blender Stain Shader 調整 ✅ 完成（2026-04-18）

**成果**：Stain 現呈有機水漬/污漬形態，邊緣不規則，顏色自然。100 張視覺 QA 確認 BBox 精準，MORPH_CLOSE 7×7 成功連接跨面碎片。

同步套用 pipeline 優化（P1-P4 + Non-Color），詳見「問題追蹤記錄」P006~P008。

---

### Phase B：5900 張正式算圖 🔄 進行中（2026-04-19 啟動）

**觸發條件**：✅ Stain shader 調整完成 → ✅ 06_Raw_Output 清空 → ✅ EXPERIMENT_NAME 全腳本切換完畢

**已套用設定：**
```python
# 01_orchestrator.py
DISTRIBUTION = {"clean": 2000, "stain_only": 1000, "puncture_only": 1000, "both": 1000}
SAFETY_MARGIN = 1.30   # 實際算圖 5900 張

# 03_render_operator.py
TOTAL_TARGET = 5900

# 05_cropper_yolo2.py
CLASS_LIMIT = 1000
EXPERIMENT_NAME = "production_v1_5000"   # 同步更新至所有 04~11 腳本
```

**執行流程：**
```powershell
cd D:\CardboardBox_detect
.\venv\Scripts\Activate.ps1
cd 02_Scripts

# Step 1：Blender 渲染（斷點續傳，約數小時）
python 03_render_operator.py    # configs 不存在時自動重跑 01_orchestrator.py

# Step 2：標注與裁切（渲染完成後）
python 04_annotator_yolo1.py     # → 04_Dataset_YOLO1/production_v1_5000/
python 05_cropper_yolo2.py       # → 05_Dataset_YOLO2/production_v1_5000/
                                 # ⚠️ 確認結束摘要「缺口」欄位全為 0（見下方說明）

# Step 3：正式訓練（Val/Test 解耦已完成，可直接訓練）
python 08_train.py --target yolo1 --patience 20
python 08_train.py --target yolo2 --patience 20
```

---

### Phase C：Val/Test 解耦 ✅ 完成（2026-04-19）

**成果**：從 114 張實拍圖中抽出 30 張為 val，84 張為 test（val ⊄ test 驗證通過，重疊=0）。

| 類別 | val | test |
|------|-----|------|
| both | 2 | 6 |
| clean_box | 16 | 46 |
| puncture | 5 | 15 |
| stain | 7 | 17 |
| **合計** | **30** | **84** |

YOLO1 + YOLO2 各自的 production_v1_5000 val/test images + labels 已複製完成，data.yaml 已建立，可在算圖完成後直接進入訓練。

### Phase D：正式版研究結論計劃

- **production_v1_5000 vs smoke_test_v2_HN**：Stain recall / Puncture FP 改善幅度
- **production_v2（含 HN）vs production_v1（純合成）**：量化 HN 對 5000 張規模的效益
- 所有實驗 test predict 圖疊圖對比（v1_500SDG / v2_HN / production_v1 / production_v2）
- **特別章節**：HN 空背景對 epoch-1 鎖死的改善
- 最終 mAP + Webcam FPS 整合入論文/報告

---

## 問題追蹤記錄（Problems Log）

> 此區記錄專案各階段遭遇的問題、根本原因與解法，**不因問題解決而刪除**。未來遇到新問題也持續追加。

---

### P001 — YOLO1 best.pt 被 epoch 1 鎖死
**狀態**：✅ 已解決（smoke_test_v2_HN）

**現象**：`best.pt` 推論 0/114 張偵測（max-conf 中位數 0.061），`last.pt` 卻能達到 109/114。

**根本原因**：三個因素同時湊巧：
1. YOLOv8n COCO pretrain 對「棕色方形物體」有強先驗（suitcase / backpack）
2. val ⊂ test（40 張 val 完全是 test 114 張的子集），epoch 1 的 COCO 先驗就能對 val 打出高分
3. LR warmup 在 epoch 1 學習率極低（0.0003），模型仍停在 COCO sweet spot

**解法**：將 22 張 HN（空背景）加入 YOLO1 train，稀釋 epoch 1 的假性高分（fitness 0.890→0.855），讓 epoch 30 的真實訓練分得以超越，best.pt 不再鎖死。

**殘留風險**：~~val ⊂ test 問題在 5000 張正式版前必須解耦~~ → ✅ production_v1_5000 val/test 已解耦（2026-04-19）。

---

### P010 — 05_cropper_yolo2.py 摘要表「缺口」說明
**狀態**：✅ 設計完成（P009 新增）

**「缺口」定義**：`CLASS_LIMIT - 實際保存數`。每個類別的目標張數（`CLASS_LIMIT=1000`）減去腳本實際成功保存的張數。若 > 0，代表該類別不足，原因通常是：
- mask 面積太小（`dw*dh < 60` 或邊長 `< 6px`）被過濾丟棄
- Blender 幀的 mask 完全空白（背景渲染失敗）

**因應方式**：結束時看摘要表，若有 ⚠️ 則提高 `01_orchestrator.py` 的 `SAFETY_MARGIN`（例如 1.30→1.50），清空 `06_Raw_Output/configs/`，重跑 `03_render_operator.py` + `05_cropper_yolo2.py`。

---

### P011 — Puncture yield 率低於 SAFETY_MARGIN 預期（54.9% vs 77%）
**狀態**：✅ 已用增量補跑機制解決（2026-04-21）

**現象**：production_v1_5000 首次跑 `05_cropper_yolo2.py`，puncture_only 類別 1300 configs 僅 714 張通過（缺口 286），yield 54.9%。`SAFETY_MARGIN=1.30` 原本預期 yield ≈ 77%，與實際差距大。

**根本原因**：Puncture 特徵偏小，`01_orchestrator.py` 的 `Puncture scale` 下限 0.25 在實際渲染後仍有大量 mask 被 `05_cropper_yolo2.py` 的面積過濾（`dw*dh < 60 或邊長 < 6px`）篩掉。Stain/Both 的 yield 較高未觸發問題。

**解法（本次採用，增量補跑）**：
1. `01_orchestrator.py` 新增 `FRAME_START_OFFSET` 常數（預設 0），補跑時設為現有最大 frame index + 1，避免 frame_id 衝突。
2. 暫時改 `DISTRIBUTION` 僅保留 `puncture_only`、`SAFETY_MARGIN=1.0`，依實際 yield 估算補跑量（286 ÷ 0.549 ≈ 521，取 550 作為 buffer）。
3. 跑 `01_orchestrator.py` 生成新 configs → `03_render_operator.py`（`TOTAL_TARGET` 同步+550）→ `05_cropper_yolo2.py`（既有已滿類別自動跳過，只填 puncture 缺口）。
4. 完成後將 `01_orchestrator.py` / `03_render_operator.py` 常數還原。

**長期改善建議（下次新實驗時）**：
- 若 puncture 仍偏低，`SAFETY_MARGIN` 可對 puncture 獨立設定（例如 1.50）
- 或提高 `01_orchestrator.py` 的 Puncture `scale` 下限 0.25→0.30

---

### P002 — YOLO2 Stain recall = 0%
**狀態**：⏳ 部分解決（shader 已調整，需等 5000 張正式訓練驗證）

**現象**：smoke_test v1+v2 的混淆矩陣顯示 Stain recall = 0%，所有 Stain GT 被預測為 background。但 predict 圖目視可見模型「有 fire Stain」（conf 0.61~0.84），只是同一塊污漬出現 3-4 個碎片框，各 IoU < 0.5，全部計為 FP。

**根本原因**：
1. 合成 Stain shader 與真實污漬外觀差距大（顏色、透明度、邊緣柔化不符）
2. 訓練資料量嚴重不足（196 張，目標 3000 張）
3. Stain mask 偵測不完整（threshold=8 + MORPH_CLOSE 5×5 未能完整連接跨面碎片）

**解法**：
- Blender Stain shader 重調（2026-04-18，用戶手動完成）
- MORPH_CLOSE kernel Stain 5×5→7×7（改善碎片連接）
- 等待 5000 張正式算圖（3000 張 ROI）驗證 recall 是否恢復

---

### P003 — YOLO2 Background FP 過高（Puncture 誤判）
**狀態**：⏳ 部分改善（tight crop 消除 Stain BG FP，Puncture BG FP 仍 64%）

**現象**：smoke_test v1 混淆矩陣：background→Stain FP=35%、background→Puncture FP=65%。

**根本原因**：
1. 加了 5% padding 的 ROI 包含紙箱邊緣、地面紋理等非瑕疵背景
2. 紙板印刷文字紋理與 Puncture（深色小圓）外觀相似
3. 負樣本不足（train 中只有 50 張 clean）

**解法（v2）**：
- 取消 padding（tight crop）→ background→Stain FP 從 35%→0%（完全消除）
- background→Puncture FP 從 65%→64%（幾乎不變，紙板紋理相似問題持續）

**待解**：Puncture BG FP 需更多 clean 負樣本（5000 張正式版中 clean 比例拉高）以及更多 Puncture 多樣性。

---

### P004 — Val 指標劇烈震盪
**狀態**：⏳ 待正式訓練改善

**現象**：YOLO1 val mAP50 在 0.32~0.94 間跳動，無法作為訓練監控依據。

**根本原因**：val 集僅 40 張（20 clean + 20 defect），對真實 domain 敏感度高，單張偏差即影響整體指標。加上 val ⊂ test（非獨立驗證集）。

**解法**：
- 正式訓練前 val/test 解耦（從 test 114 張獨立抽出 20-30 張為 val）
- 擴大 val 至 60-80 張
- `--patience 20` 防止 val 雜訊造成早停

---

### P005 — results.csv 被 exist_ok=True 覆蓋消失
**狀態**：✅ 已確認機制，接受此行為

**現象**：`08_train.py` 的 `exist_ok=True` 覆蓋同名 run 時，Ultralytics 不重建 results.csv，導致 `07_Models/yolo2/` 無訓練曲線 CSV。

**解法**：每次新實驗使用新的 `EXPERIMENT_NAME`，避免覆蓋舊 run。訓練中途不更改 EXPERIMENT_NAME。

---

### P006 — Puncture decal 載入時 colorspace 未設定
**狀態**：✅ 已修復（2026-04-18）

**現象**：`apply_dynamic_decal()` 只替換 TEX_IMAGE node 的 image，未設定 colorspace。新載入的 PNG 預設 sRGB，導致 Puncture 深色破口被 gamma curve 扭曲，破口偏灰。

**修復**：在 `02_blender_render.py` 的 `apply_dynamic_decal()` 加入：
```python
if keyword == "puncture":
    new_img.colorspace_settings.name = 'Non-Color'
```

---

### P007 — bg_strength 多幀累乘 Bug
**狀態**：✅ 已修復（2026-04-18）

**現象**：P4 加入 `bg_strength` 倍率後，若 frame 結束時還原到「乘過倍率後的值」而非原始值，下一幀讀到污染後的 default_value，再次乘以新倍率，導致亮度逐幀累積偏移。

**修復**：在 frame 開始時獨立存 `orig_main_bg_strength`，frame 結束時強制還原此原始值（不使用 PASS2 區段的 `orig_bg_strength` 殘值）。

---

### P008 — JPEG quality=72 在暗部產生假紋理
**狀態**：✅ 已改善（2026-04-18）

**現象**：JPEG 壓縮 quality=72 在暗色 ROI 中產生明顯色塊，這些色塊有時比 Stain 本身更顯眼，可能讓模型學到錯誤特徵（把 JPEG artifact 當 Stain）。

**修復**：quality 72→82，保留 Sim-to-Real 壓縮效果同時減少暗部假紋理。

---

### P009 — YOLO2 資料集數量無法對齊目標（mask 過濾缺口 + 路徑不一致）
**狀態**：✅ 已修復（2026-04-19）

**現象**：100 張視覺 QA 測試中，YOLO2 各類實際保存量低於計劃：
- `stain_only`：目標 20，實際 16（缺口 4）
- `puncture_only`：目標 20，實際 14（缺口 6）

另外，`04_annotator_yolo1.py` / `05_cropper_yolo2.py` 寫入 flat 路徑，`08_train.py` 讀取版本化路徑，每次新實驗須手動 `mv` 檔案，且路徑一旦搞錯訓練資料就用錯版本。

**根本原因**：
1. Orchestrator 生成的 config 數量恰好等於 CLASS_LIMIT，沒有留緩衝給 mask 面積過濾（`dw*dh < 60` 等條件）丟棄的幀。
2. 被過濾的 defect 幀重分類為 clean，若 clean 額度已滿則靜默丟棄，無任何警告輸出。
3. 04/05 腳本輸出路徑為 flat，與 08_train.py 的版本化路徑不匹配。

**修復**：
- `01_orchestrator.py`：新增 `SAFETY_MARGIN=1.30`；defect 類別改為生成 `ceil(num × 1.30)` 份 config，讓 Blender 多渲染 30% 備用幀。mask 過濾防禦機制完全不變（壞幀仍被丟棄），只是備用幀足夠多讓 CLASS_LIMIT 能填滿。
- `04_annotator_yolo1.py`：新增 `EXPERIMENT_NAME` 常數，輸出至 `04_Dataset_YOLO1/{EXP}/images/train/`；自動生成 `data.yaml`（若不存在）。
- `05_cropper_yolo2.py`：同上改為版本化路徑；新增 `configs_seen` / `reclassified` 計數器；結束時印出摘要表，缺口 > 0 觸發 ⚠️ 警告，提示使用者調高 `SAFETY_MARGIN`。

---

### P012 — P001 epoch-1 鎖死不限 YOLO1，任何模型只要無 HN 都會中招
**狀態**：✅ 已識別（2026-04-21，production_v1_5000 實測確認）；⏳ 解法待 v2 驗證

**現象**：production_v1_5000 訓練結果顯示：
- YOLO1 best.pt = **epoch 2**（fitness 0.951，train 5900 張純合成無 HN）
- YOLO2 best.pt = **epoch 1**（fitness 0.166，train 4000 張純合成無 HN）
- 兩者皆於 best_epoch + 20 觸發 patience 早停

與先前實驗對照形成完美 ablation：

| 實驗 | 資料量 | HN 空背景 | YOLO1 best epoch | YOLO2 best epoch |
|------|--------|-----------|------------------|-----------------|
| smoke_test_v1_500SDG | 500 | ❌ 無 | **1**（鎖死）| 未追溯 |
| smoke_test_v2_HN | 500 | ✅ 22 張 | **30**（排除）| 未追溯 |
| production_v1_5000 | 5000 | ❌ 無 | **2**（鎖死）| **1**（鎖死）|

**根本原因修正**：CLAUDE.md 先前 P001 條目推測「YOLO2 在 COCO 無 Stain/Puncture 先驗，epoch 1 不會複製問題」— **此判斷錯誤**。實測顯示 P001 的本質機制是：
1. Val 集小 + 真實 domain 震盪 → mAP50-95 曲線起伏劇烈
2. Ultralytics fitness 公式 `0.1×mAP50 + 0.9×mAP50-95` 偏重 mAP50-95 → epoch 1 的 warmup LR 低，模型仍接近 pretrain 權重，某種意義上穩定
3. Patience 邏輯 `best_epoch + patience` → 早期 epoch 鎖死 best.pt 後整個訓練時長被綁架

**→ 與 COCO bias 無關，與模型是否有相關類別先驗無關，任何「val 震盪大 + 無 HN」組合都會中招**。

**解法**：
1. **加入 HN 空背景稀釋 epoch 1 假性高分**（smoke v2 已驗證對 YOLO1 有效；YOLO2 待 production_v2_HN 驗證）
2. 或 **`save_period=10`** 每 10 epoch 存 checkpoint 人工挑
3. 或 **部署一律用 last.pt**（production_v1 實測 last.pt 顯著優於 best.pt）

**附帶產出**：三組 YOLO1 + 兩組 YOLO2 ablation 對照，為論文/報告的「HN 負樣本效益」章節提供硬數據。

---

### P013 — YOLO2 雙重增強（Cropper + Ultralytics default）可能打壞細微特徵
**狀態**：✅ 已驗證（2026-04-21，noaug / noultaug 對照實驗）— **Puncture 最受影響**：去除 cropper aug 讓 val Puncture mAP50 從 ~0.10→0.76，去除 Ultralytics aug 讓 mAP50→0.385；單獨任一層移除都有顯著改善。Stain 不受 aug 影響（三組皆 ~0%）→ Stain 問題是 shader domain gap。

**現象**：YOLO2 production_v1 predict 圖顯示：
- 明顯瑕疵（深色水漬、圓形破口）可正確檢出（Stain 0.55 / Puncture 0.76）
- 淡色 Stain 退化成碎片化低信心框（0.31~0.35），且框對在文字/插圖而非真實污漬

**嫌疑根因**：`07_Models/production_v1_5000/yolo2/args.yaml` 顯示 Ultralytics 預設增強全開：
```yaml
mosaic: 1.0            # 4 圖拼貼
hsv_h: 0.015, hsv_s: 0.7, hsv_v: 0.4
erasing: 0.4           # 隨機遮蔽
auto_augment: randaugment
```

而 `05_cropper_yolo2.py` 已在寫入 train 前套用 Sim-to-Real 增強（冷色 / 對比 / blur / JPEG / 噪點）。**兩層疊加**可能破壞 Stain 半透明邊緣與色調微差，Puncture 因特徵粗壯較不受影響（但 recall 仍從 smoke v2 的 79% 退至 25%，可能部分原因於此）。

**解法（production_v2 實驗時套用）**：
- `mosaic=0.0`（ROI 不適合拼貼）
- `erasing=0.0`（ROI 小，再遮就沒東西）
- `hsv_s=0.3 hsv_v=0.2`（降色彩擾動，cropper 已做）

---

## 各規模切換設定

> 每次調整只需改以下常數，`01_orchestrator.py` 會自動依 `SAFETY_MARGIN` 計算實際渲染量。

### 500 張測試（供參考，已完成）

```python
# 01_orchestrator.py
DISTRIBUTION = {"clean": 250, "stain_only": 100, "puncture_only": 100, "both": 50}
SAFETY_MARGIN = 1.30   # defect 類別實際算圖量 = ceil(num × 1.30)，約 565 張

# 03_render_operator.py
TOTAL_TARGET = 565

# 04_annotator_yolo1.py / 05_cropper_yolo2.py / 08_train.py / 09~11
EXPERIMENT_NAME = "smoke_test_v2_HN"

# 05_cropper_yolo2.py
CLASS_LIMIT = 50
```

### ✅ 正式版 5000 張（production_v1_5000 — 目前使用中）

```python
# 01_orchestrator.py
DISTRIBUTION = {"clean": 2000, "stain_only": 1000, "puncture_only": 1000, "both": 1000}
SAFETY_MARGIN = 1.30   # 實際算圖量 = 2000 + 1300×3 = 5900 張

# 03_render_operator.py
TOTAL_TARGET = 5900

# 04_annotator_yolo1.py / 05_cropper_yolo2.py / 08_train.py / 09~11
EXPERIMENT_NAME = "production_v1_5000"

# 05_cropper_yolo2.py
CLASS_LIMIT = 1000
```

### 後續：production_v2（HN 負樣本對比實驗）

```python
# 與 v1 相同設定，差異只有：
# 1. EXPERIMENT_NAME = "production_v2_HN"
# 2. 手動將 empty_background/ 22+ 張複製到 YOLO1 train 資料夾（空 label）
# 3. YOLO2 不加 HN（推論輸入永遠是 ROI，不會看到純背景）
```

---

## YOLO 訓練指令備忘（升級 3）

```bash
# RTX 4050 6GB — YOLOv8n，batch=32，混合精度省 ~30% VRAM
yolo train model=yolov8n.pt data=data.yaml imgsz=640 batch=32 amp=True epochs=300

# 升級 4（待 500 張 smoke test 後視 mAP 決定是否切換）
# YOLOv8s：~3.8GB VRAM @batch=32，預期 +3-5% mAP，訓練時間 x2
yolo train model=yolov8s.pt data=data.yaml imgsz=640 batch=32 amp=True epochs=300
```

---

## 待驗證實驗：飄浮模式 Dataset（待 500 張 smoke test 後執行）

### 背景

目前 dataset 全為「落地模式」（紙箱放地面，相機俯視），但部署場景為**手持紙箱對鏡頭**。
實拍時紙箱六個面都會被拍到，訓練資料缺乏側面/底面視角可能造成 domain gap。

Nvidia Omniverse Replicator 的業界做法即為物件任意飄浮（不貼地）+ 全軸旋轉，
以強迫模型見到各種姿態。

### 執行前置條件

- [ ] 500 張 smoke test 跑完，YOLO1 mAP@50 有基準數字
- [ ] 研究文獻確認飄浮 vs 貼地對 YOLO 效能的具體影響（目前缺乏直接數據）
- [ ] 決定比例：建議先測 `FLOAT_RATIO = 0.40`（四成飄浮）

### 飄浮模式實作要點（已設計，待啟用）

```python
# 01_orchestrator.py 新增常數
FLOAT_RATIO = 0.40   # 0.0=全落地 / 1.0=全飄浮

# generate_config() 新增邏輯
is_floating = random.random() < FLOAT_RATIO

if is_floating:
    rot_x    = random.uniform(-math.pi, math.pi)   # 任意俯仰
    rot_y    = random.uniform(-math.pi, math.pi)   # 任意側滾
    z_offset = random.uniform(0.4, 1.2)            # 高度 0.4-1.2m（手持範圍）
    elevation = random.uniform(5, 85)              # 相機可接近水平
else:
    rot_x    = random.uniform(-math.radians(12), math.radians(12))  # 輕微傾斜
    rot_y    = random.uniform(-math.radians(12), math.radians(12))
    z_offset  = 0.0
    elevation = random.uniform(20, 80)             # 現有擴大範圍

# 新 box 欄位
"box": {
    ...,
    "rotation_x": rot_x,    # 新增
    "rotation_y": rot_y,    # 新增
    "z_offset":   z_offset  # 新增：0=貼地, >0=飄浮高度(m)
}
```

```python
# 02_blender_render.py 落地/飄浮邏輯
rot_x    = config["box"].get("rotation_x", 0.0)
rot_y    = config["box"].get("rotation_y", 0.0)
z_offset = config["box"].get("z_offset",   0.0)

box_copy.rotation_euler = (rot_x, rot_y, rot_z)
bpy.context.view_layer.update()

if z_offset > 0:
    box_copy.location.z = z_offset          # 飄浮：直接設高度
else:
    lowest_z = min([...bound_box...])
    box_copy.location.z -= lowest_z         # 落地：貼齊地面
```

### 方法論一致性評估

| 問題 | 評估 |
|------|------|
| 手持場景 → 飄浮訓練資料是否合理？| ✅ 部署場景即為手持（非地面），飄浮更貼近真實 |
| 背景問題（地面 HDRI vs 室內背景）| ⚠️ 飄浮時地面材質仍可見，需加入室內牆面 HDRI |
| YOLO1 學習混淆風險 | ⚠️ 若大量飄浮，「紙箱在地面」反而成少見情境，需保留落地比例 |
| GitHub demo 效果 | ✅ 手持展示時各面都能被偵測到 |

---

## 硬體規格

| 項目 | 規格 | 可行性 |
|------|------|--------|
| GPU | NVIDIA RTX 4050 Laptop GPU（6.44GB VRAM）| — |
| YOLOv8n 訓練 | batch=32, 640px, amp=True → ~3.5GB | ✅ 可提升至 batch=32 |
| Blender Cycles 算圖 | 640×640，128 samples，OptiX | ✅ GPU 加速後可行 |
| 即時雙段 YOLO 推論 | <1.5GB VRAM | ✅ 輕鬆 |

---

## 開發環境（已驗證 2026-04-17）

> **正式 venv 位於 `d:\CardboardBox_detect\venv\`（`pyvenv.cfg` 指向 Python 3.11.9）。**
> 所有訓練/推論指令都必須在此 venv 內執行，不要用 conda `nv` 或系統 Python。

### 啟動方式（PowerShell）

```powershell
cd D:\CardboardBox_detect
.\venv\Scripts\Activate.ps1
# prompt 應變成：(venv) PS D:\CardboardBox_detect>
where.exe python    # 驗證：應顯示 D:\CardboardBox_detect\venv\Scripts\python.exe
```

### 已驗證套件版本

| 套件 | 版本 | 備註 |
|------|------|------|
| Python | 3.11.9 | venv 基底 |
| torch | 2.5.1+cu121 | **GPU 版** |
| torchvision | 0.20.1+cu121 | |
| CUDA | 12.1 | `torch.cuda.is_available() == True` |
| GPU 偵測 | NVIDIA GeForce RTX 4050 Laptop GPU（6.44 GB VRAM） | ✅ |
| ultralytics | 8.4.38 | YOLOv8 訓練主框架 |
| opencv-python | 4.13.0.92 | 02_Scripts 內標註/裁切使用 |
| numpy | 2.4.3 | |
| pillow | 12.1.1 | |
| matplotlib | 3.10.8 | 訓練圖表 |
| scipy | 1.17.1 | ultralytics 相依 |
| PyYAML | 6.0.3 | data.yaml 讀取 |

### CUDA 快速自測指令

```powershell
.\venv\Scripts\python.exe -c "import torch; print('cuda:', torch.cuda.is_available(), '|', torch.cuda.get_device_name(0) if torch.cuda.is_available() else '-')"
```
預期輸出：`cuda: True | NVIDIA GeForce RTX 4050 Laptop GPU`

### 如何重建 venv（災難恢復）

若 venv 損毀或要換機，完整重建指令：

```powershell
cd D:\CardboardBox_detect
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip install ultralytics
```

---

## 系統架構圖

### 1. 全流水線總覽

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║                        CardboardBox_detect  全流水線                            ║
╠══════════════════════════════════════════════════════════════════════════════════╣
║                                                                                  ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 0 ── 場景健檢                                                     │    ║
║  │  00_scene_health_check.py  →  scene_spec.md / scene_spec.json           │    ║
║  │  確認 Blender 場景物件名稱、相機、地面、貼圖皆符合規格                    │    ║
║  └──────────────────────────────┬──────────────────────────────────────────┘    ║
║                                 │                                                ║
║                                 ▼                                                ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 1 ── 劇本生成                                                     │    ║
║  │  01_orchestrator.py                                                      │    ║
║  │  輸出 500 / 5000 個 JSON 到 06_Raw_Output/configs/                       │    ║
║  │  每個 JSON 描述：box 位置、camera 視角、HDRI、地面材質、瑕疵 decal 規格  │    ║
║  └──────────────────────────────┬──────────────────────────────────────────┘    ║
║                                 │                                                ║
║                                 ▼                                                ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 2 ── Blender 渲染（守衛啟動）                                     │    ║
║  │  03_render_operator.py  →  02_blender_render.py (Blender -b --python)          │    ║
║  │  每幀三個 Pass：主圖 / Box Mask / 瑕疵 Mask(s)                           │    ║
║  │  輸出到 06_Raw_Output/images/ + masks/                                   │    ║
║  └──────────────────────────────┬──────────────────────────────────────────┘    ║
║                                 │                                                ║
║                    ┌────────────┴────────────┐                                  ║
║                    ▼                         ▼                                   ║
║  ┌─────────────────────────┐   ┌─────────────────────────────────────────┐      ║
║  │  PHASE 3a ── YOLO1 標注  │   │  PHASE 3b ── YOLO2 裁切 + 增強         │      ║
║  │  04_annotator_yolo1.py  │   │  05_cropper_yolo2.py                    │      ║
║  │  Box Mask → BBox label  │   │  Box BBox → ROI crop → Sim-to-Real 增強 │      ║
║  │  無增強，直接複製        │   │  → 瑕疵 Mask → YOLO2 label             │      ║
║  └────────────┬────────────┘   └──────────────────┬──────────────────────┘      ║
║               │                                   │                              ║
║               ▼                                   ▼                              ║
║  ┌─────────────────────────┐   ┌─────────────────────────────────────────┐      ║
║  │  04_Dataset_YOLO1/      │   │  05_Dataset_YOLO2/                      │      ║
║  │  images/train/          │   │  images/train/  (增強後 ROI)            │      ║
║  │  labels/train/          │   │  labels/train/  (Stain cls0/Punct cls1) │      ║
║  └────────────┬────────────┘   └──────────────────┬──────────────────────┘      ║
║               │                                   │                              ║
║               └──────────────┬────────────────────┘                             ║
║                              ▼                                                   ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 4 ── Dataset 整合（06_build_dataset.py ─ 已取代）                │    ║
║  │  real_refs/0408/ 實拍圖已由內嵌腳本直接寫入 YOLO1/2 val/test           │    ║
║  │  12_qa_real_dataset.py 產生人眼 QA 視覺化                              │    ║
║  └──────────────────────────────┬──────────────────────────────────────────┘    ║
║                                 │                                                ║
║                    ┌────────────┴────────────┐                                  ║
║                    ▼                         ▼                                   ║
║  ┌─────────────────────────┐   ┌─────────────────────────────────────────┐      ║
║  │  PHASE 5a               │   │  PHASE 5b                               │      ║
║  │  YOLO1 訓練             │   │  YOLO2 訓練                             │      ║
║  │  YOLOv8n, 640×640       │   │  YOLOv8n, 640×640                      │      ║
║  │  class: The Box (0)     │   │  class: Stain(0), Puncture(1)           │      ║
║  │  → 07_Models/yolo1/     │   │  → 07_Models/yolo2/                    │      ║
║  └────────────┬────────────┘   └──────────────────┬──────────────────────┘      ║
║               │                                   │                              ║
║               └──────────────┬────────────────────┘                             ║
║                              ▼                                                   ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 6a ── Test 集單段驗證                                            │    ║
║  │  09_predict_yolo1.py  →  04_Dataset_YOLO1/test → predict_yolo1/         │    ║
║  │  10_predict_yolo2.py  →  05_Dataset_YOLO2/test → predict_yolo2/         │    ║
║  └──────────────────────────────┬──────────────────────────────────────────┘    ║
║                                 ▼                                                ║
║  ┌─────────────────────────────────────────────────────────────────────────┐    ║
║  │  PHASE 6b ── 即時雙段 Demo（11_realtime_demo.py）                       │    ║
║  │  Webcam → YOLO1 偵測紙箱 → Crop ROI → YOLO2 偵測瑕疵 → 疊框輸出       │    ║
║  │  S 鍵截圖到 predict_snapshots/                                          │    ║
║  └─────────────────────────────────────────────────────────────────────────┘    ║
║                                                                                  ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

---

### 2. Blender 渲染詳細資料流

```
01_Assets/                          03_Blender_Project/
├── HDRIs/          ───────┐        └── *.blend
├── Ground/ (PBR)  ───────┤                │
├── decals/defects/ ──────┤                │  02_blender_render.py
│   ├── Stain/      ──────┼────────────────┤  (Blender -b --python)
│   ├── Puncture/   ──────┤                │
│   └── ...         ──────┘                │
                                           │
06_Raw_Output/configs/{frame_id}.json ─────┘
(由 01_orchestrator.py 生成的場景劇本)
         │
         │  每幀執行三個 Render Pass
         │
         ├── PASS 1 (samples=64, Denoising ON)
         │    └── 06_Raw_Output/images/{frame_id}.png          ← 主圖 640×640 RGBA
         │
         ├── PASS 2 (samples=8, Denoising OFF)
         │    └── 06_Raw_Output/masks/{frame_id}_mask_0_The_Box.png   ← 紙箱白色 Emission Mask
         │
         └── PASS 3 (samples=8, Denoising OFF) ← 僅 stain/puncture/both 幀執行
              ├── 06_Raw_Output/masks/{frame_id}_mask_1_Stain.png
              └── 06_Raw_Output/masks/{frame_id}_mask_2_Puncture.png
```

---

### 3. 標注流水線詳細資料流

```
                    06_Raw_Output/
                    ├── images/{frame_id}.png
                    └── masks/
                        ├── {frame_id}_mask_0_The_Box.png
                        ├── {frame_id}_mask_1_Stain.png     (若有)
                        └── {frame_id}_mask_2_Puncture.png  (若有)
                               │
          ┌────────────────────┴────────────────────┐
          │                                         │
          ▼  04_annotator_yolo1.py                  ▼  05_cropper_yolo2.py
          │                                         │
          │  Box Mask → findContours                │  Box Mask → BBox (+ 5% padding)
          │  → BBox 正規化 → YOLO格式              │  → 裁切 ROI 640×640
          │                                         │
          │  【無增強，直接複製】                   │  缺陷 Mask 處理：
          │                                         │  ├── threshold: Stain=8, Puncture=25
          │                                         │  ├── MORPH_CLOSE 5×5（連接碎片）
          │                                         │  ├── findNonZero → boundingRect
          │                                         │  └── 濾除雜訊 (面積<60px / 邊長<6px)
          │                                         │
          ├── 備份                                  ├── 備份（未增強）
          │   06_Raw_Output/images_ready/yolo1/     │   06_Raw_Output/images_ready/yolo2/
          │                                         │
          ├── 訓練圖                                ├── augment_yolo2() 增強：
          │   04_Dataset_YOLO1/images/train/        │   ├── 冷色偏移 (R-6, B+10)
          │                                         │   ├── 對比壓縮 (×0.90 + 13)
          ├── 訓練標注                              │   ├── GaussianBlur ksize=3
          │   04_Dataset_YOLO1/labels/train/        │   ├── JPEG quality=72
          │   格式: 0 cx cy w h                     │   └── Gaussian noise std=4
          │                                         │
          └── QA 視覺化                             ├── 訓練圖（增強後）
              06_Raw_Output/qa_visualization/yolo1/ │   05_Dataset_YOLO2/images/train/
              綠框 = The Box                        │
                                                    ├── 訓練標注
                                                    │   05_Dataset_YOLO2/labels/train/
                                                    │   格式: 0 cx cy w h (Stain)
                                                    │         1 cx cy w h (Puncture)
                                                    │
                                                    └── QA 視覺化
                                                        06_Raw_Output/qa_visualization/yolo2/
                                                        藍框=Stain / 黃框=Puncture
```

---

### 4. Dataset 分割規劃（含真實資料整合）

```
                    合成資料來源                           真實資料來源
              04_Dataset_YOLO1/train/             01_Assets/real_refs/0408/
              (500 張測試 / 5000 張正式)           ├── clean_box/  (60 張)
              ├── clean        250 / 2000          ├── stain/      (24 張)
              ├── stain_only   100 / 1000          ├── puncture/   (20 張)
              ├── puncture_only 100 / 1000         └── both/       ( 8 張)
              └── both          50 / 1000
                      │                                    │
                      └──────────────┬─────────────────────┘
                                     │  06_build_dataset.py (已取代，實拍資料由內嵌腳本直寫)
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
        ┌───────────┐          ┌───────────┐          ┌───────────┐
        │  YOLO1    │          │  YOLO1    │          │  YOLO1    │
        │  train    │          │  val      │          │  test     │
        │  合成全類  │          │  真實 20  │          │  真實 100 │
        │  3000 張  │          │  合成 10  │          │  clean 80 │
        └───────────┘          └───────────┘          │  defect 20│
                                                       └───────────┘

              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
        ┌───────────┐          ┌───────────┐          ┌───────────┐
        │  YOLO2    │          │  YOLO2    │          │  YOLO2    │
        │  train    │          │  val      │          │  test     │
        │  合成 ROI  │          │  真實 10  │          │  真實 80  │
        │  3000 張  │          │  合成 20  │          │  clean 40 │
        │  類別平衡  │          │  clean 限定│          │  defect 40│
        └───────────┘          └───────────┘          └───────────┘

  注意：真實資料全部用於 val / test，絕對不混入 train。
```

---

### 5. 即時雙段推論架構（目標狀態）

```
  Acer Nitro AN515 Webcam (720p, H.264)
          │
          │  frame (1280×720 or 640×480)
          ▼
  ┌──────────────────────────────────┐
  │  PRE-PROCESS                     │
  │  resize → 640×640                │
  └──────────────────┬───────────────┘
                     │
                     ▼
  ┌──────────────────────────────────┐
  │  YOLO1  (YOLOv8n)               │
  │  07_Models/yolo1/best.pt         │
  │  Input:  640×640 全幀            │
  │  Output: BBox [x1,y1,x2,y2]     │
  │          class 0 = The Box       │
  └──────────────────┬───────────────┘
                     │  BBox (含 5% padding)
                     ▼
  ┌──────────────────────────────────┐
  │  CROP PROGRAM                    │
  │  裁切紙箱 ROI → resize 640×640  │
  │  （與訓練時 05_cropper 邏輯鏡像）│
  └──────────────────┬───────────────┘
                     │
                     ▼
  ┌──────────────────────────────────┐
  │  YOLO2  (YOLOv8n)               │
  │  07_Models/yolo2/best.pt         │
  │  Input:  640×640 ROI             │
  │  Output: BBox list               │
  │          class 0 = Stain         │
  │          class 1 = Puncture      │
  └──────────────────┬───────────────┘
                     │  BBox 座標反投影回全幀
                     ▼
  ┌──────────────────────────────────┐
  │  OVERLAY RENDER                  │
  │  綠框 = The Box  (YOLO1)         │
  │  藍框 = Stain    (YOLO2)         │
  │  黃框 = Puncture (YOLO2)         │
  └──────────────────────────────────┘
          │
          ▼
    螢幕即時顯示（目標 ≥15 FPS）
```

---

### 6. 目錄依賴關係圖

```
  01_Assets/          02_Scripts/              06_Raw_Output/         Datasets/
  ──────────          ───────────              ──────────────         ─────────
  HDRIs/ ──────────► 02_blender_render.py ──► images/      ──►──┐
  Ground/ ─────────►    (via Blender)       ► masks/        ──►──┤
  decals/ ─────────►                                             │
  real_refs/ ─────────────────────────────────────────────────►──┼──► 06_build_dataset.py
                     01_orchestrator.py  ──► configs/       ──►──┤       │
                     03_render_operator.py ────►    (劇本 JSON)         │       ▼
                     04_annotator_yolo1.py ◄─ images/ + masks/ ──┘  04_Dataset_YOLO1/
                          │                                     ──►  05_Dataset_YOLO2/
                          └──► images_ready/yolo1/                        │
                               04_Dataset_YOLO1/                          ▼
                               qa_visualization/yolo1/              07_Models/
                                                                    yolo1/best.pt
                     05_cropper_yolo2.py ◄── images/ + masks/      yolo2/best.pt
                          │                                              │
                          └──► images_ready/yolo2/                       │
                               05_Dataset_YOLO2/                         │
                               qa_visualization/yolo2/                   │
                                                                         ▼
                     09_predict_yolo1.py ──► predict_yolo1/ ◄── yolo1/best.pt
                     10_predict_yolo2.py ──► predict_yolo2/ ◄── yolo2/best.pt
                     11_realtime_demo.py ──► predict_snapshots/ ◄── yolo1+yolo2
                     12_qa_real_dataset.py ─► qa_visualization/yolo{1,2}_real/
```
🎯 下一步 Action Items（2026-04-22 更新 · hybrid_v2 完成後）

### 主線：Ch.7 Two-Stage Fine-Tune 對比實驗 ⭐

驗證「Nvidia 方法論（合成預訓練 → 真實微調）」能否勝過已驗證成功的 single-stage hybrid_v2。

**Stage 1（已完成，免重跑）**：
- 基底權重 = `07_Models/production_v1_noaug/yolo2/weights/best.pt`（純合成預訓練，未沾真實資料）

**Stage 2（待執行）**：
- 新 Dataset：`05_Dataset_YOLO2/production_v1_finetune/` = 僅 44 張真實（沿用 hybrid_v2 的 train real，確保 ablation 乾淨）
- Val：沿用現有 30 張實拍 val
- 超參（建議起手）：
  ```
  --weights 07_Models/production_v1_noaug/yolo2/weights/best.pt
  --freeze 10         # 凍結 backbone 前 10 層
  --lr0 0.0005        # 預設 0.01 會砸爛既有權重
  --epochs 50
  --patience 15
  --no-yolo-aug
  ```
- 輸出：`07_Models/production_v1_finetune/yolo2/`
- 預期三種結果判讀：
  - C > B（hybrid_v2）→ Nvidia 方法論勝，故事收斂到正統 fine-tune
  - C ≈ B → 證明 single-stage hybrid 配置對就夠用（本身即論文貢獻）
  - C < B → 44 張太少撐不起獨立 Stage 2

**待決定參數**：freeze 層數（建議 10，備案 0 / 22 三組 ablation）

---

### 並行支線（非阻塞）

**A. Webcam 即時 demo（1 小時，可隨時拍）**
- YOLO1：`07_Models/production_v1_5000/yolo1/weights/last.pt`
- YOLO2：**`07_Models/production_v1_hybrid_v2/yolo2/weights/best.pt`** ✨（新最強）
- Conf：`CONF_YOLO1=0.50`、`CONF_YOLO2=0.40`
- 更新 `11_realtime_demo.py` 的 `EXPERIMENT_NAME = "production_v1_hybrid_v2"`

**B. 最終疊圖對比（README 封面素材）**
- 6 組 badge wall：v1_500SDG / v2_HN / v1_5000 / noaug / noultaug / **hybrid_v2**（+ 之後加 finetune）
- 用相同 test 圖（交集區段）做並排對比

**C. Blender Stain Shader 再調（中期，僅當 Ch.7 後仍想提升 Stain recall）**
- hybrid_v2 已把 Stain 從三組 ablation 的 0% 拉到 test 80%，shader 優先度下降
- 若需再提升，目標：邊緣透明度梯度平滑 / 顏色偏暖 / alpha noise

**D. production_v2_HN 對比實驗（論文用 ablation，低優先）**
- 量化 HN 對 5000 張規模的效益
- 現在 hybrid_v2 已提供更強解法，此實驗學術意義 > 實用意義

---

## 進度更新（2026-05-11）

### 腳本救回

`11b_predict_video.py` 與對應影片資料夾於先前 `pre-cleanse-all` stash 操作中被誤封入 stash，今日透過 `git checkout stash@{0}^3 --` 從 untracked 區精準取回，其他 stash 內容未動：

| 復原項目 | 路徑 |
|---------|------|
| 腳本 | `02_Scripts/11b_predict_video.py` |
| 實拍影片輸入 | `06_Raw_Output/predict_videos_input/` (IMG_9133/9134/9135.MOV) |
| 推論輸出影片 | `06_Raw_Output/predict_videos_output/` (IMG_9132~9135_annotated.mp4) |

用途：輸入實拍 .MOV 影片 → YOLO1+YOLO2 兩段推論 → 輸出帶框 .mp4。

---

### 影片結論素材（12b_make_conclusion_assets.py）

新增腳本 `02_Scripts/12b_make_conclusion_assets.py`，一次生成三個影片最後 30 秒用的視覺素材，輸出至 `08_records/charts/conclusion_assets/`：

| 素材 | 檔名 | 尺寸 | 說明 |
|------|------|------|------|
| 1 實驗總表 | `asset1_summary_table.png` | ~1910×1052 | YOLO1（4 組）+ YOLO2（5 組）分段表格，黑底，重要實驗色彩高亮，locked/OK 用紅/綠標示 |
| 2 比較圖格 | `asset2_compare_grid.png` | 1920×1080 | 2 欄 × 4 列，上方全幅貼 5 組實驗名稱 header（重要實驗有色底），各列取 `_compare_intersection_40` 代表圖（both/stain/puncture/clean_box 各 2 張） |
| 3 Results 疊圖 | `asset3_results_overlay.png` | ~1880×956 | YOLO1（4 個）+ YOLO2（8 個）results.png 縮圖排格，重要實驗彩色邊框 + 彩色 label bar，次要/早期實驗自動去飽和灰階 |

**Compare grid 5 組實驗順序（左→右）**：
1. `realonly_fair`（Real-Only）灰色
2. `noaug`（Pure-Syn noaug）藍
3. `noultaug`（Pure-Syn noultaug）淺藍 ★重要
4. `hybrid_v1`（Hybrid v1 no anchor）紅
5. `hybrid_v2`（Hybrid v2 +anchor）綠 ★重要

重新跑腳本指令：
```powershell
cd D:\CardboardBox_detect
.\venv\Scripts\Activate.ps1
python 02_Scripts\12b_make_conclusion_assets.py
```

---

### 流水線腳本進度表（更新至 2026-05-11）

| 步驟 | 狀態 |
|------|------|
| 11b_predict_video.py | **✅ 已救回**（可輸入 .MOV 影片、輸出帶框 .mp4） |
| 12b_make_conclusion_assets.py | **✅ 新建**（影片結論三素材生成腳本） |
| 影片結論素材（asset1/2/3） | **✅ 完成**（`08_records/charts/conclusion_assets/`）|

---

## 📢 GitHub README 大改版（2026-05-12）

### 改版動機

研究實驗已於 2026-04-22 全部收官，此次進行 GitHub 公開呈現的完整整理，目標受眾為 NVIDIA GTC 與會者 / 技術主管 / 招募人員，全程以英文撰寫。

---

### 主要變動清單

#### README 結構

| 區塊 | 變動 |
|------|------|
| 語言 | 全文改英文（繁中版保留於本 CLAUDE.md）|
| 開頭 | 新增 YouTube demo 影片縮圖連結（點擊跳 YouTube）|
| Demo 區塊 | 4 支 GIF（從推論 MP4 轉製，各取前 10 秒，5fps，360px）|
| Dataset 區塊 | 3 支 GIF（render_images / mask_passes / annotated，各 60 frames）|
| Limitations | 改寫為「3 個月個人 AI 協作 side project」語境；補 L9 物件可複製性低 |
| Future Work | 新增一節：Generative AI 閉合 Sim-to-Real gap（Blender 可控性 + diffusion 後處理方向）|
| Citations | 補全三篇論文：Tobin 2017 / Ghiasi CVPR 2021 / Boikov et al. Symmetry 2021 |
| Disclaimer | 新增免責聲明：紙箱為商業品牌，本專案純學術用途、無商業利益 |
| AI 聲明 | 移除（不公開 AI 協作記錄） |

#### Git 管理

| 項目 | 說明 |
|------|------|
| 檔案重命名 | `PROJECT_NOTES.md` → `CLAUDE.md`（恢復原名） |
| .gitignore 新增 | `06_Raw_Output/*.MOV`、`06_Raw_Output/predict_videos_input/`、`06_Raw_Output/predict_videos_output/*.mp4`（超過 GitHub 100MB 限制） |
| Commit author | 移除 `Co-Authored-By: Claude Sonnet 4.6`，避免 GitHub 顯示「bbKurt11 and claude」|
| Push 方式 | `git push --force origin main`（修正 diverged history）|

---

### 新增腳本

| 腳本 | 路徑 | 功能 |
|------|------|------|
| `make_dataset_gifs.py` | `02_Scripts/` | 從 `06_Raw_Output/images/`、`masks/`、`qa_visualization/yolo1/` 各取 60 frames 產生 dataset showcase GIF |
| `make_demo_gifs.py` | `02_Scripts/` | 從 4 支推論 MP4 各取前 10 秒（每 5 幀取 1，5fps，360px）轉 GIF |

**重新產生 GIF 指令：**
```powershell
cd D:\CardboardBox_detect
.\venv\Scripts\Activate.ps1
python 02_Scripts\make_dataset_gifs.py   # → 08_records/charts/conclusion_assets/gif/render_images.gif 等
python 02_Scripts\make_demo_gifs.py      # → 08_records/charts/conclusion_assets/gif/IMG_9132_annotated.gif 等
```

---

### GIF 檔案清單（`08_records/charts/conclusion_assets/gif/`）

| 檔案 | 內容 | 大小 |
|------|------|------|
| `render_images.gif` | Blender 算圖（6450 張取 60 幀）| 10.1 MB |
| `mask_passes.gif` | The_Box mask pass（取 60 幀）| 0.6 MB |
| `annotated.gif` | YOLO QA 標注疊圖（6427 張取 60 幀）| 9.7 MB |
| `IMG_9132_annotated.gif` | Stain 偵測 demo | 7.2 MB |
| `IMG_9133_annotated.gif` | Puncture 偵測 demo | 5.1 MB |
| `IMG_9134_annotated.gif` | Clean box（0% FP）demo | 7.7 MB |
| `IMG_9135_annotated.gif` | Dual-defect demo | 8.4 MB |

---

### YouTube Demo 影片

- **URL**：https://www.youtube.com/watch?v=2sVSZpJezgQ
- **原始檔**：`video_demo/08_Final_Assets/CardboardBox_video_README.mp4`（107.7MB，89秒，1920×1080）
- README 嵌入方式：YouTube 縮圖圖片連結（`img.youtube.com/vi/{ID}/maxresdefault.jpg`），點擊跳 YouTube

---

### 目前 GitHub 狀態（2026-05-12）

- **Repo**：https://github.com/bbKurt11/SDG_CardboardBox_detect
- **Branch**：`main`（latest commit: `fd93c97`）
- **README**：全英文，已含 YouTube embed、7 支 GIF、4 大發現、完整 Limitations & Future Work
- **CLAUDE.md**：本檔，保存全專案研究記錄（非公開工作記錄）

---
> Source: [bbKurt11/SDG_CardboardBox_detect](https://github.com/bbKurt11/SDG_CardboardBox_detect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
