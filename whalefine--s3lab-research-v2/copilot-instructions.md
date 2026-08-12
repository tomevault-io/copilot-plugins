## benchmark-results

> SGLATrack historical benchmark results for comparing experiments


# SGLATrack Benchmark Results

Pre-train: MAE ViT-Base (mae_pretrain_vit_base.pth)
Training: COCO17 + GOT10K_train_full, Epoch 50, Batch 32, LR 5e-5

## COCO+GOT10K（Epoch 50）彙整表

（AUC / Prec 皆為 %；Avg. 為五個資料集 AUC 平均）

| Model | UAV123 AUC | UAV123 P | UAV123_10fps AUC | UAV123_10fps P | UAVTrack112 AUC | UAVTrack112 P | UAVTrack112L AUC | UAVTrack112L P | DTB70 AUC | DTB70 P | Avg. AUC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 論文寫的 vit | 66.10 | 83.90 | 64.50 | 82.20 | 66.90 | 81.30 | 64.70 | 81.70 | 63.80 | 83.50 | 64.70 |
| vit.py（原架構） | 63.79 | 83.07 | 63.30 | 82.40 | 64.10 | 82.50 | 60.60 | 80.10 | 58.90 | 77.40 | 62.10 |
| vit_sima.py（原） | 59.21 | 77.62 | 58.62 | 77.02 | 57.28 | 75.40 | 53.73 | 73.51 | 52.69 | 71.24 | 56.31 |
| vit_CARE.py（attention=elu()+1） | 55.80 | 73.77 | 57.14 | 75.79 | 53.57 | 71.88 | 49.98 | 64.38 | 51.76 | 69.31 | 52.65 |
| vit_CARE_relu.py（attention=relu()） | 62.59 | 81.67 | 62.33 | 81.02 | 59.97 | 77.95 | 55.97 | 75.64 | 54.82 | 72.94 | 59.14 |
| vit_CARE_relu6.py（attention=relu6(), MLP=GELU） | 62.59 | 81.67 | 62.33 | 81.02 | 59.97 | 77.95 | 55.97 | 75.64 | 54.82 | 72.94 | 59.14 |
| vit_CARE_relu6.py（attention=relu6(), MLP=ReLU） | 60.88 | 79.31 | 61.44 | 79.96 | 60.62 | 79.17 | 56.52 | 76.36 | 53.84 | 71.45 | 58.66 |
| vit_CARE_relu6_fixed.py（relu6(), fixed-point, MLP=ReLU） | 60.65 | 78.83 | 61.49 | 80.07 | 60.16 | 78.56 | 56.45 | 76.56 | 53.37 | 71.05 | 58.43 |
| vit_CARE_relu6_fixed.py（fixed-point 自訓/自測） | 34.30 | 44.83 | 34.80 | 45.20 | 29.87 | 37.70 | 17.39 | 22.22 | 33.65 | 44.09 | 30.00 |
| vit_CARE_relu_BN.py | 58.85 | 77.08 | 58.94 | 77.16 | 58.87 | 77.85 | 54.12 | 75.07 | 52.66 | 70.49 | 56.69 |
| vit_CARE_relu6_BN.py | 59.10 | 77.49 | 58.44 | 76.48 | 56.99 | 74.56 | 54.82 | 74.18 | 54.51 | 73.39 | 56.77 |
| vit_CARE_gelu.py | 55.40 | 74.20 | 55.00 | 73.50 | 52.10 | 70.00 | 45.00 | 64.70 | 50.90 | 68.70 | 51.70 |
| vit_MALA.py（原） | 61.51 | 79.88 | 62.63 | 81.68 | 61.53 | 79.49 | 57.40 | 76.44 | 55.74 | 74.18 | 59.76 |
| vit_MALA_relu.py（attention=relu()） | 61.66 | 80.44 | 62.60 | 81.70 | 60.30 | 78.90 | 57.30 | 77.70 | 55.90 | 74.50 | 59.60 |
| vit_MALA_relu6.py（attention=relu6()） | 62.08 | 80.90 | 61.75 | 80.02 | 61.55 | 80.04 | 58.08 | 78.21 | 55.45 | 74.12 | 59.78 |
| vit_MALA_relu6_BN.py | 57.65 | 75.63 | 58.10 | 76.85 | 58.72 | 76.74 | 55.21 | 75.17 | 53.66 | 73.19 | 56.67 |
| vit_square.py（原） | 58.64 | 77.24 | 58.80 | 77.50 | 56.60 | 74.20 | 52.20 | 70.40 | 56.10 | 74.30 | 56.50 |

## 論文 vit (reported)
- UAV123: AUC=66.1, Prec=83.9
- UAV123_10fps: AUC=64.5, Prec=82.2
- UAVTrack112: AUC=66.9, Prec=81.3
- UAVTrack112L: AUC=64.7, Prec=81.7
- DTB70: AUC=63.8, Prec=83.5

## vit.py (原架構 baseline)
- UAV123: AUC=63.79, Prec=83.07
- UAV123_10fps: AUC=63.3, Prec=82.4
- UAVTrack112: AUC=64.1, Prec=82.5
- UAVTrack112L: AUC=60.6, Prec=80.1
- DTB70: AUC=58.9, Prec=77.4
- Avg AUC=62.1, FPS=160.0

## vit_sima.py
- UAV123: AUC=59.21, Prec=77.62
- UAV123_10fps: AUC=58.62, Prec=77.02
- UAVTrack112: AUC=57.28, Prec=75.40
- UAVTrack112L: AUC=53.73, Prec=73.51
- DTB70: AUC=52.69, Prec=71.24
- Avg AUC=56.31, FPS=161.49

## vit_CARE.py (原; attention 改 `elu()`)
- UAV123: AUC=55.80, Prec=73.77
- UAV123_10fps: AUC=57.14, Prec=75.79
- UAVTrack112: AUC=53.57, Prec=71.88
- UAVTrack112L: AUC=49.98, Prec=64.38
- DTB70: AUC=51.76, Prec=69.31
- UAVDT: AUC=36.4, Prec=55.6
- Avg AUC=52.65, FPS=158.14

## vit_CARE_relu.py (attention 改 `relu()`)
- UAV123: AUC=**62.59**, Prec=81.67
- UAV123_10fps: AUC=62.33, Prec=81.02
- UAVTrack112: AUC=59.97, Prec=77.95
- UAVTrack112L: AUC=55.97, Prec=75.64
- DTB70: AUC=54.82, Prec=72.94
- Avg AUC=59.14, FPS=159.31

## vit_CARE_relu6.py (attention 改 `relu6()`, MLP `GELU`)
- UAV123: AUC=**62.59**, Prec=81.67
- UAV123_10fps: AUC=62.33, Prec=81.02
- UAVTrack112: AUC=59.97, Prec=77.95
- UAVTrack112L: AUC=55.97, Prec=75.64
- DTB70: AUC=54.82, Prec=72.94
- Avg AUC=59.14, FPS=160.00

## vit_CARE_relu6.py (attention 改 `relu6()`, MLP `ReLU`)
- UAV123: AUC=60.88, Prec=79.31
- UAV123_10fps: AUC=61.44, Prec=79.96
- UAVTrack112: AUC=60.62, Prec=79.17
- UAVTrack112L: AUC=56.52, Prec=76.36
- DTB70: AUC=53.84, Prec=71.45
- Avg AUC=58.66, FPS=158.63

## vit_CARE_relu6_fixed.py (attention `relu6()`, fix-point, MLP `ReLU`)
- UAV123: AUC=60.65, Prec=78.83
- UAV123_10fps: AUC=61.49, Prec=80.07
- UAVTrack112: AUC=60.16, Prec=78.56
- UAVTrack112L: AUC=56.45, Prec=76.56
- DTB70: AUC=53.37, Prec=71.05
- Avg AUC=58.43, FPS=120.26

## vit_CARE_relu_BN.py
- UAV123: AUC=58.85, Prec=77.08
- UAV123_10fps: AUC=58.94, Prec=77.16
- UAVTrack112: AUC=58.87, Prec=77.85
- UAVTrack112L: AUC=54.12, Prec=75.07
- DTB70: AUC=52.66, Prec=70.49
- Avg AUC=56.69, FPS=149.69

## vit_CARE_relu6_BN.py
- UAV123: AUC=59.10, Prec=77.49
- UAV123_10fps: AUC=58.44, Prec=76.48
- UAVTrack112: AUC=56.99, Prec=74.56
- UAVTrack112L: AUC=54.82, Prec=74.18
- DTB70: AUC=54.51, Prec=73.39
- Avg AUC=56.77

## vit_CARE_relu6_fixed.py（fixed-point 自訓/自測）
- UAV123: AUC=34.30, Prec=44.83
- UAV123_10fps: AUC=34.80, Prec=45.20
- UAVTrack112: AUC=29.87, Prec=37.70
- UAVTrack112L: AUC=17.39, Prec=22.22
- DTB70: AUC=33.65, Prec=44.09
- Avg AUC=30.00

## vit_CARE_gelu.py
- UAV123: AUC=55.4, Prec=74.2
- UAV123_10fps: AUC=55.0, Prec=73.5
- UAVTrack112: AUC=52.1, Prec=70.0
- UAVTrack112L: AUC=45.0, Prec=64.7
- DTB70: AUC=50.9, Prec=68.7
- Avg AUC=51.7, FPS=159.1

## vit_MALA.py (原)
- UAV123: AUC=61.51, Prec=79.88
- UAV123_10fps: AUC=62.63, Prec=81.68
- UAVTrack112: AUC=61.53, Prec=79.49
- UAVTrack112L: AUC=57.40, Prec=76.44
- DTB70: AUC=55.74, Prec=74.18
- Avg AUC=59.76, FPS=154.42

## vit_MALA_relu.py (attention 改 `relu()`)
- UAV123: AUC=61.66, Prec=80.44
- UAV123_10fps: AUC=62.6, Prec=81.7
- UAVTrack112: AUC=60.3, Prec=78.9
- UAVTrack112L: AUC=57.3, Prec=77.7
- DTB70: AUC=55.9, Prec=74.5
- Avg AUC=59.6, FPS=155.75

## vit_MALA_relu6.py
- UAV123: AUC=62.08, Prec=80.90
- UAV123_10fps: AUC=61.75, Prec=80.02
- UAVTrack112: AUC=61.55, Prec=80.04
- UAVTrack112L: AUC=58.08, Prec=78.21
- DTB70: AUC=55.45, Prec=74.12
- Avg AUC=59.78, FPS=156.14

## vit_MALA_relu6_BN.py
- UAV123: AUC=57.65, Prec=75.63
- UAV123_10fps: AUC=58.10, Prec=76.85
- UAVTrack112: AUC=58.72, Prec=76.74
- UAVTrack112L: AUC=55.21, Prec=75.17
- DTB70: AUC=53.66, Prec=73.19
- Avg AUC=56.67, FPS=147.44

## vit_square.py (原)
- UAV123: AUC=58.64, Prec=77.24
- UAV123_10fps: AUC=58.8, Prec=77.5
- UAVTrack112: AUC=56.6, Prec=74.2
- UAVTrack112L: AUC=52.2, Prec=70.4
- DTB70: AUC=56.1, Prec=74.3
- Avg AUC=56.5, FPS=155.46

## Key Observations

- Baseline (vit.py): Avg AUC=62.1, FPS=160.0
- Best CARE linear-attention family (by Avg AUC): vit_CARE_relu 與 vit_CARE_relu6（MLP GELU）同為 Avg AUC=59.14、UAV123 AUC=**62.59**；換成 MLP ReLU 後 Avg AUC 降至 58.66
- vit_CARE_relu6_fixed（fix-point + MLP ReLU）Avg AUC=58.43，FPS=120.26（fixed-point 較慢）
- Adding BN hurts: vit_CARE_relu_BN（Avg 56.69, FPS 149.69）、vit_MALA_relu6_BN（Avg 56.67, FPS 147.44）
- GELU activation performs worst among listed CARE variants without ReLU attention (Avg AUC=51.7 for vit_CARE_gelu)
- ReLU attention improves CARE base (52.65→59.14)；MALA 方面 vit_MALA_relu / vit_MALA_relu6 與原 vit_MALA 相近或略好（Avg 約 59.6–59.78）
- Among MALA variants, vit_MALA_relu6 has highest Avg AUC (59.78)

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
