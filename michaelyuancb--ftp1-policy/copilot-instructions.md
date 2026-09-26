## ftp1-policy

> 本项目实现了 **FTP1 (Robot Tactile Action 1)** 模型，这是一个基于扩散的多模态策略模型，扩展自 Pi0.5 架构，添加了异构触觉传感器支持。

# OpenPI 模型架构文档

## 概述

本项目实现了 **FTP1 (Robot Tactile Action 1)** 模型，这是一个基于扩散的多模态策略模型，扩展自 Pi0.5 架构，添加了异构触觉传感器支持。

训练入口脚本: `scripts/zarr_train_ftp1_pytorch.py`

---

## 模型架构总览

```
FTP1Pytorch
├── paligemma_with_expert (FTP1PaliGemmaWithExpertModel)
│   ├── paligemma (PaliGemmaForConditionalGeneration)  -- VLM Expert (Gemma 2B)
│   │   ├── vision_tower (SiglipVisionModel)           -- 图像编码器
│   │   └── language_model (GemmaForCausalLM)          -- 语言模型
│   ├── gemma_expert (GemmaForCausalLM)                -- Action Expert (Gemma 300M)
│   └── gemma_tactile_expert (GemmaForCausalLM)        -- Tactile Expert (Gemma 300M, 可选)
├── hpt_tactile_encoder (FTP1HptTactileEncoder)       -- 异构触觉编码器
│   ├── tokenizers (ModuleDict)                        -- 不同传感器类型的编码器
│   │   ├── TactileDataEncoder (binary/state)          -- 二值/状态类型编码
│   │   ├── MatrixCNNEncoder                           -- 矩阵类型CNN编码
│   │   └── ImageTactileEncoder (ViT)                  -- 图像类型ViT编码
│   └── shared_image_chunk_encoder                     -- T3预训练共享块编码器
├── state_encoder (FourierStateEncoder)                -- 状态傅里叶编码器
├── action_in_proj (Linear)                            -- 动作输入投影
├── action_out_proj (Linear)                           -- 动作输出投影
├── time_mlp_in (Linear)                               -- 时间步MLP输入
└── time_mlp_out (Linear)                              -- 时间步MLP输出
```

---

## 模型架构图
 输入数据
      ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
      │     Images      │ │    Language     │ │     State       │ │    Tactiles     │ │ Actions + Noise │
      │   (H,W,3) x N   │ │    Prompt       │ │   (B,T,dim)     │ │   (dict)        │ │  (B,T,act_dim)  │
      └────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────┬────────┘
               │                   │                   │                   │                   │
               ▼                   ▼                   │                   ▼                   ▼
  ┌────────────────────────────────────────────┐      │    ┌──────────────────────────────────────────────────────┐
  │                                            │      │    │                                                      │
  │           embed_prefix()                   │      │    │                 embed_suffix()                       │
  │                                            │      │    │                                                      │
  │  ┌──────────────────────────────────────┐  │      │    │  ┌────────────────────┐    ┌─────────────────────┐   │
  │  │        SiglipVisionModel             │  │      │    │  │   action_in_proj   │    │    time_mlp_in      │   │
  │  │  ┌────────────────────────────────┐  │  │      │    │  │    (nn.Linear)     │    │    time_mlp_out     │   │
  │  │  │ transformers_replace/models/   │  │  │      │    │  │                    │    │    (nn.Linear)      │   │
  │  │  │ siglip/modeling_siglip.py      │  │  │      │    │  │  32 ──► 1024       │    │                     │   │
  │  │  └────────────────────────────────┘  │  │      │    │  └─────────┬──────────┘    └──────────┬──────────┘   │
  │  │  Patches:14x14, Layers:27, Dim:1152  │  │      │    │            │     ┌──────────────────────┘            │
  │  └──────────────────┬───────────────────┘  │      │    │            │     │  (AdaRMS conditioning)            │
  │                     │                      │      │    │            ▼     ▼                                   │
  │                     ▼                      │      │    │  ┌─────────────────────┐                             │
  │  ┌──────────────────────────────────────┐  │      │    │  │  Suffix Embeddings  │                             │
  │  │     PaliGemma Multi-modal Projector  │  │      │    │  │  (B, T, 1024)       │                             │
  │  └──────────────────┬───────────────────┘  │      │    │  └─────────────────────┘                             │
  │                     │                      │      │    │                                                      │
  │                     ▼                      │      │    │  models_pytorch/ftp1_pytorch.py                     │
  │  ┌──────────────────────────────────────┐  │      │    └──────────────────────────────────┬───────────────────┘
  │  │         Gemma Embedding Layer        │◄─┼──────┘                                       │
  │  └──────────────────┬───────────────────┘  │                                              │
  │                     │                      │                                              │
  │                     ▼                      │                                              │
  │  ┌──────────────────────────────────────┐  │                                              │
  │  │       Prefix Embeddings              │  │                                              │
  │  │       (B, seq_len, 2048)             │  │                                              │
  │  └──────────────────────────────────────┘  │                                              │
  │                                            │                                              │
  │  models_pytorch/ftp1_pytorch.py           │                                              │
  │  transformers_replace/.../modeling_        │                                              │
  │  paligemma.py                              │                                              │
  └─────────────────────┬──────────────────────┘                                              │
                        │                                                                     │
                        │         ┌───────────────────────────────────────────────────────────┘
                        │         │
                        │         │        ┌──────────────────────────────────────────────────────────────────────┐
                        │         │        │                      embed_tactile()                                 │
                        │         │        │  ┌────────────────────────────────────────────────────────────────┐  │
                        │         │        │  │                  FTP1HptTactileEncoder                        │  │
                        │         │        │  │                  models_pytorch/ftp1_blocks.py:550            │  │
                        │         │        │  │                                                                │  │
                        │         │        │  │   ┌────────────────┐ ┌────────────────┐ ┌───────────────────┐  │  │
                        │         │        │  │   │ Binary/State   │ │    Matrix      │ │      Image        │  │  │
                        │         │        │  │   │    Encoder     │ │    Encoder     │ │     Encoder       │  │  │
                        │         │        │  │   │ ┌────────────┐ │ │ ┌────────────┐ │ │ ┌───────────────┐ │  │  │
                        │         │        │  │   │ │ Fourier    │ │ │ │ MatrixCNN  │ │ │ │ImageTactile  │ │  │  │
                        │         │        │  │   │ │ State      │ │ │ │ Encoder    │ │ │ │Encoder       │ │  │  │
                        │         │        │  │   │ │ Encoder    │ │ │ │            │ │ │ │              │ │  │  │
                        │         │        │  │   │ │ :20        │ │ │ │ :119       │ │ │ │ :343         │ │  │  │
                        │         │        │  │   │ └────────────┘ │ │ └────────────┘ │ │ └───────┬───────┘ │  │  │
                        │         │        │  │   └────────────────┘ └────────────────┘ │         │         │  │  │
                        │         │        │  │                                         │         ▼         │  │  │
                        │         │        │  │                                         │ ┌───────────────┐ │  │  │
                        │         │        │  │                                         │ │ViTEncoder     │ │  │  │
                        │         │        │  │                                         │ │t3_tactile_    │ │  │  │
                        │         │        │  │                                         │ │encoder.py:140 │ │  │  │
                        │         │        │  │                                         │ └───────┬───────┘ │  │  │
                        │         │        │  │                                         │         ▼         │  │  │
                        │         │        │  │                                         │ ┌───────────────┐ │  │  │
                        │         │        │  │                                         │ │Transformer    │ │  │  │
                        │         │        │  │                                         │ │Trunk (T3)     │ │  │  │
                        │         │        │  │                                         │ │:180           │ │  │  │
                        │         │        │  │                                         │ └───────────────┘ │  │  │
                        │         │        │  │                                         └───────────────────┘  │  │
                        │         │        │  │                                                                │  │
                        │         │        │  │   Output: Tactile Tokens (B, T, num_areas, 1024)               │  │
                        │         │        │  └────────────────────────────────────────────────────────────────┘  │
                        │         │        │  models_pytorch/ftp1_pytorch.py                                     │
                        │         │        └──────────────────────────────────────────────────┬───────────────────┘
                        │         │                                                           │
                        ▼         ▼                                                           ▼
  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │                                                                                                                 │
  │                                      FTP1PaliGemmaWithExpertModel                                              │
  │                                      models_pytorch/ftp1_gemma_pytorch.py                                      │
  │                                                                                                                 │
  │  ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────┐  │
  │  │                                                                                                           │  │
  │  │  ┌─────────────────────────────────┐                                                                      │  │
  │  │  │     VLM Expert (Gemma 2B)       │◄────── Prefix Tokens                                                 │  │
  │  │  │  paligemma.language_model       │                                                                      │  │
  │  │  │  ┌───────────────────────────┐  │        Attention: Self only                                          │  │
  │  │  │  │ GemmaDecoderLayer x 18   │  │                                                                      │  │
  │  │  │  │ Width: 2048, Heads: 8    │  │                                                                      │  │
  │  │  │  │ modeling_gemma.py        │  │                                                                      │  │
  │  │  │  └───────────────────────────┘  │                                                                      │  │
  │  │  └─────────────────────────────────┘                                                                      │  │
  │  │                    │                                                                                      │  │
  │  │                    │ (hidden states)                                                                      │  │
  │  │                    ▼                                                                                      │  │
  │  │  ┌─────────────────────────────────┐      ┌─────────────────────────────────┐                             │  │
  │  │  │  Tactile Expert (Gemma 300M)    │      │   Action Expert (Gemma 300M)    │◄────── Suffix Tokens        │  │
  │  │  │  gemma_tactile_expert           │      │   gemma_expert                  │        + Time Cond          │  │
  │  │  │  ┌───────────────────────────┐  │      │  ┌───────────────────────────┐  │                             │  │
  │  │  │  │ GemmaDecoderLayer x 18   │  │      │  │ GemmaDecoderLayer x 18   │  │        Attention:            │  │
  │  │  │  │ Width: 1024, Heads: 8    │  │      │  │ Width: 1024, Heads: 8    │  │        Cross to ALL          │  │
  │  │  │  │ No AdaRMS                │  │      │  │ AdaRMS ✓                 │  │                             │  │
  │  │  │  │ modeling_gemma.py        │  │      │  │ modeling_gemma.py        │  │                             │  │
  │  │  │  └───────────────────────────┘  │      │  └───────────────────────────┘  │                             │  │
  │  │  └─────────────────────────────────┘      └────────────────┬────────────────┘                             │  │
  │  │                                                            │                                              │  │
  │  │  ◄──────── Tactile Tokens                                  │                                              │  │
  │  │            Attention: Self only                            │                                              │  │
  │  │                                                            │                                              │  │
  │  └────────────────────────────────────────────────────────────┼──────────────────────────────────────────────┘  │
  │                                                               │                                                 │
  │  transformers_replace/models/gemma/modeling_gemma.py          │                                                 │
  │  transformers_replace/models/gemma/configuration_gemma.py     │                                                 │
  └───────────────────────────────────────────────────────────────┼─────────────────────────────────────────────────┘
                                                                  │
                                                                  │ Action Expert Output
                                                                  ▼
                                                ┌─────────────────────────────────┐
                                                │        action_out_proj          │
                                                │         (nn.Linear)             │
                                                │        1024 ──► 32              │
                                                │                                 │
                                                │  models_pytorch/ftp1_pytorch.py│
                                                └────────────────┬────────────────┘
                                                                 │
                                                                 ▼
                                                ┌─────────────────────────────────┐
                                                │      Predicted Velocity u_t     │
                                                │      Shape: (B, horizon, 32)    │
                                                │                                 │
                                                │      Loss = MSE(u_t, noise-act) │
                                                └─────────────────────────────────┘


  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════
                                                  配置文件
  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
  │                                          FTP1ModelConfig                                                       │
  │                                   models_pytorch/ftp1_model_config.py                                          │
  │                                                                                                                 │
  │   paligemma_variant: "gemma_2b"          action_expert_variant: "gemma_300m"                                    │
  │   tactile_expert_variant: "gemma_300m"   action_dim: 32    action_horizon: 32                                   │
  │   state_input_mode: 'none'               use_tactile_input: True                                                │
  │                                                                                                                 │
  │   ┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐   │
  │   │                                  FTP1TactileTokenizerConfig                                            │   │
  │   │   single_hand_num_tactile_areas: 24  fourier_dim: 8                                                     │   │
  │   │   frozen_shared_chunk: False         load_t3_pretrained_checkpoint: True                                │   │
  │   └─────────────────────────────────────────────────────────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                      │
                                      uses ──────────►│
                                                      ▼
  ┌──────────────────────────────────────────────────────────────────────────────────────────┐
  │                                      models/gemma.py                                     │
  │                                                                                          │
  │   get_config(variant) ──► Returns GemmaConfig with width, depth, heads, etc.             │
  │                                                                                          │
  │   "gemma_2b":   width=2048, depth=18, heads=8,  kv_heads=1, head_dim=256, mlp_dim=16384  │
  │   "gemma_300m": width=1024, depth=18, heads=8,  kv_heads=1, head_dim=256, mlp_dim=4096   │
  └──────────────────────────────────────────────────────────────────────────────────────────┘


  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════
                                                文件路径汇总 (相对于 src/openpi)
  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════

  ┌──────────────────────────────────────┬───────────────────────────────────────────────────────────────────────────┐
  │              模块                    │                              文件路径                                      │
  ├──────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ FTP1Pytorch (主模型)                │ models_pytorch/ftp1_pytorch.py                                           │
  │ FTP1ModelConfig                     │ models_pytorch/ftp1_model_config.py                                      │
  │ FTP1PaliGemmaWithExpertModel        │ models_pytorch/ftp1_gemma_pytorch.py                                     │
  │ FTP1HptTactileEncoder               │ models_pytorch/ftp1_blocks.py:550                                        │
  │ FourierStateEncoder                  │ models_pytorch/ftp1_blocks.py:20                                         │
  │ MatrixCNNEncoder                     │ models_pytorch/ftp1_blocks.py:119                                        │
  │ ImageTactileEncoder                  │ models_pytorch/ftp1_blocks.py:343                                        │
  │ TactileDataEncoder                   │ models_pytorch/ftp1_blocks.py:343                                        │
  │ SharedImageChunkEncoder              │ models_pytorch/ftp1_blocks.py:245                                        │
  │ ViTEncoder (T3)                      │ models_pytorch/t3_tactile_encoder.py:140                                  │
  │ TransformerTrunk (T3)                │ models_pytorch/t3_tactile_encoder.py:180                                  │
  │ PaliGemmaForConditionalGeneration    │ models_pytorch/transformers_replace/models/paligemma/modeling_paligemma.py│
  │ GemmaForCausalLM                     │ models_pytorch/transformers_replace/models/gemma/modeling_gemma.py        │
  │ SiglipVisionModel                    │ models_pytorch/transformers_replace/models/siglip/modeling_siglip.py      │
  │ preprocess_observation_pytorch       │ models_pytorch/preprocessing_pytorch.py                                   │
  │ get_config (Gemma配置)               │ models/gemma.py                                                           │
  │ sample_beta, make_att_2d_masks       │ models_pytorch/pi0_pytorch.py                                             │
  └──────────────────────────────────────┴───────────────────────────────────────────────────────────────────────────┘


  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════
                                            FTP1 当前训练路径
  ═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════

### 概述

当前仓库仅维护标准 FTP1 训练与推理路径：

- 保留视觉、触觉、状态三类输入的标准配置组合
- 训练/推理统一使用 `FTP1TrainConfig`、`FTP1ModelConfig` 与 `FTP1InferenceWrapper`
- 数据、归一化与 UniVTAC 评测均按当前 FTP1 action/state 布局工作

### 关键约束

1. **配置层面**
   - `action_dim` 由当前 FTP1 action layout 决定
   - 需要同时保持 `training/config.py`、`dataset_zarr.py`、推理 wrapper 的字段契约一致

2. **数据层面**
   - zarr 数据集直接输出当前 FTP1 所需的 state / action / tactile / image 结构
   - 若调整 action/state 语义，需要重新计算 norm stats 或验证复用资产仍然匹配

3. **推理层面**
   - `FTP1InferenceWrapper.infer()` 接收标准 FTP1 state/action 维度
   - UniVTAC 评测与部署侧应使用 wrapper 的标准 `get_state_dim()` / `get_action_dim()` 接口

### 相关文件

| 文件                                   | 职责                                   |
|----------------------------------------|----------------------------------------|
| `models_pytorch/ftp1_model_config.py`  | FTP1 模型配置定义                      |
| `training/config.py`                   | 训练配置定义与收敛校验                 |
| `dataset_zarr.py`                      | zarr 数据取样、模态拼装与标注处理      |
| `policies/ftp1_inference_wrapper.py`   | FTP1 推理输入预处理与动作反归一化      |
| `scripts/zarr_train_ftp1_pytorch.py`   | FTP1 训练入口                          |

---
> Source: [michaelyuancb/ftp1-policy](https://github.com/michaelyuancb/ftp1-policy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
