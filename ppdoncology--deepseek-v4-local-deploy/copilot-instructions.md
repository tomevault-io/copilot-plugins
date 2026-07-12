## deepseek-v4-local-deploy

> 本仓库整理自 GitHub 公开资源，汇总 DeepSeek V4 Pro / Flash 的本地部署方案。

# DeepSeek V4 本地部署资源库

## 项目简介

本仓库整理自 GitHub 公开资源，汇总 DeepSeek V4 Pro / Flash 的本地部署方案。

## 模型说明

| 模型 | 总参数 | 激活参数 | 上下文 | HuggingFace |
|---|---|---|---|---|
| DeepSeek-V4-Pro | 1.6T | 37B | 128K | deepseek-ai/DeepSeek-V4 |
| DeepSeek-V4-Flash | 280B | 37B | 1M | deepseek-ai/DeepSeek-V4-Flash |

## 目录结构

```
guides/
  01-vllm.md          # vLLM：NVIDIA/AMD 通用，FP8/BF16
  02-sglang.md        # SGLang：最推荐，MLA 优化，多节点
  03-lmdeploy.md      # LMDeploy：FP8/BF16，离线+在线
  04-tensorrt-llm.md  # TensorRT-LLM：BF16/INT4/INT8
  05-dwarfstar4.md    # DwarfStar4：Mac Metal 专用，V4 Flash
  06-dgx-spark.md     # 双 DGX Spark：FP4 混合后端，12 tok/s

scripts/
  download_model.sh   # 模型权重下载（支持国内镜像）
  test_inference.sh   # 推理接口测试
```

## 快速决策

- **Mac 用户** → `guides/05-dwarfstar4.md`（Metal，V4 Flash，96GB+ 可用）
- **NVIDIA 多卡 / 多机** → `guides/02-sglang.md` 或 `guides/01-vllm.md`
- **DGX Spark** → `guides/06-dgx-spark.md`（最详细，含混合后端 bug 修复）
- **量化部署** → `guides/04-tensorrt-llm.md`（INT4/INT8）

## 关键注意事项

1. V4 Pro（1.6T）完整精度需要 ~8× H100 80GB；量化（INT4）可降至更少卡
2. V4 Flash（280B）FP4 量化约 160GB，双 DGX Spark（2×128GB）即可运行
3. DGX Spark（SM120+）不能直接用 vLLM 官方 main，需用 jasl fork + 混合后端
4. Mac 上推荐 DwarfStar4（antirez/ds4），原生 Metal，KV cache 可持久化到 SSD

## 来源仓库

- https://github.com/lmxxf/deepseek-v4-deployment-on-dgx-spark
- https://github.com/antirez/ds4
- https://github.com/bailaiOWO/DeepSeek-V4（官方模型卡）
- https://github.com/vllm-project/vllm-project.github.io/blob/main/_posts/2026-04-24-deepseek-v4.md

---
> Source: [ppdoncology/deepseek-v4-local-deploy](https://github.com/ppdoncology/deepseek-v4-local-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
