## mechcad-mllm

> 我们将这一新模型暂定名为 **"MechCAD-MLLM"**（面向机械领域的CAD多模态大模型）。

# MechCAD-MLLM

### **总体研究思路：构建基于 MLLM + 双解码器 (Dual-Decoder) 的智能CAD生成框架**

我们将这一新模型暂定名为 **"MechCAD-MLLM"**（面向机械领域的CAD多模态大模型）。

#### **1. 模型架构设计 (Architecture Design)**

任务书建议采用统一的多模态大模型底座（如LLaVA），并结合Drawing2CAD的双解码器和CAD-GPT的空间Token。

  * **核心底座 (Backbone):**

      * **Encoder (多模态理解):** 参考 **CAD-MLLM** 的做法，使用一个预训练的 ViT (如 CLIP 或 SigLIP) 处理图像，以及一个 LLM (如 Llama 3 或 Vicuna) 处理文本。
      * **Fusion Strategy:** 不要只做简单的特征拼接。你可以参考 **Img2CAD** 中的 *VLM-Assisted Conditional Factorization* 思想，先让 LLM 理解意图（例如：“生成一个带有四个孔的法兰盘”），输出一个高维的语义向量 (Latent Semantic Vector, $Z$)。

  * **解码端创新 (The Innovation - 结合 Drawing2CAD 与 CAD-GPT):**

      * 不要直接让 LLM 输出所有文本 Token（这容易导致长序列错误）。
      * **引入双解码器 (Dual-Decoder):** 这是对 **Drawing2CAD** (`model/model.py` 中的 `SVG2CADTransformer`) 的核心迁移。
          * **解码器1 (Command Decoder):** 接收语义向量 $Z$，专门预测操作命令序列 (如 `Sketch`, `Extrude`, `Cut`)。这部分词表较小，更容易学习结构拓扑。
          * **解码器2 (Args Decoder):** 接收 $Z$ 以及解码器1输出的 **Guidance** (即命令的Hidden States)，专门预测具体的几何参数。
      * **空间感知增强 (Spatial Reasoning):** 在 **Args Decoder** 的预测头（Prediction Head）中，废弃传统的归一化浮点数预测，改为分类预测 **CAD-GPT** 提出的 **空间定位Token** (Spatial Tokens)。
          * 例如：将坐标轴位置预测改为预测 `<P_1024>` 这样的离散 Token。
          * 将旋转角度预测改为预测 `<A_285>` (CAD-GPT中的角度Token)。

#### **2. 数据集构建与处理 (Data Engineering)**

任务书要求针对“机械零件”（含孔、凸台），需整合 DeepCAD 和 Omni-CAD。

  * **数据筛选:** 从 **Omni-CAD** (CAD-MLLM的数据集) 中筛选出标记为 "Mechanical", "Part", "Flange" 等标签的数据。重点关注包含 `Cut` (切除) 操作的序列。
  * **数据增强 (Data Augmentation):**
      * 参考 **Drawing2CAD** 的数据处理方式，它的输入是SVG。你可以编写脚本，将 DeepCAD/Omni-CAD 的3D模型投影为2D工程图（SVG/Image），作为多模态输入的图像部分。
      * **关键点:** 任务书提到“覆盖拉伸、切除”。你需要编写脚本统计 DeepCAD 数据集中 `Extrude` 操作中 `boolean_type == 2` (通常代表 Cut/Difference) 的样本，确保训练集中这类样本比例足够（建议 30% 以上），否则模型很难学会“打孔”。

#### **3. 训练策略 (Training Strategy)**

参考 **CAD-MLLM** 的渐进式训练：

1.  **阶段一 (Unimodal - Text/Code Alignment):** 冻结 Vision Encoder，只训练 LLM + Dual-Decoder。输入文本描述，输出符合 CAD-GPT 格式的 Command/Args 序列。
      * *目的:* 让模型学会 CAD 的语法规则和空间 Token 的物理意义。
2.  **阶段二 (Multimodal Alignment):** 解冻 Vision Encoder，加入图像数据。
      * *目的:* 让模型学会看图建模（Image-to-CAD）。
3.  **阶段三 (Fine-tuning for Mechanical Features):** 在筛选出的机械零件数据集上进行微调，使用 **Drawing2CAD** 中的 **Soft Target Distribution Loss** (参考 `Drawing2CAD/trainer/loss.py` 中的 `gumbel_loss`)。
      * *原因:* 机械零件对精度要求高，Drawing2CAD 的 Soft Loss 允许参数在一定公差范围内波动，这比纯粹的 Cross-Entropy 更符合工程实际。

#### **4. 代码复现与迁移路线 (Implementation Roadmap)**

既然你已了解 **Img2CAD**，现在的重点是提取 **Drawing2CAD** 的核心逻辑：

1.  **阅读重点:**

      * 打开 `lllssc/drawing2cad/.../model/model.py`。
      * 重点看 `SVG2CADTransformer` 类中的 `forward` 函数：
        ```python
        # 伪代码逻辑
        command_logits, guidance = self.command_decoder(z)
        args_logits = self.args_decoder(z, guidance) # 注意这里的 guidance
        ```
      * 你需要将这个 `SVG2CADTransformer` 修改为对接 LLM 的输出，而不是它原有的 SVG Encoder。

2.  **具体修改:**

      * 保留 `CommandDecoder` 和 `ArgsDecoder` 类。
      * 将 `Encoder` 替换为你微调好的 LLaMA/Vicuna 提取出的 Embedding 层。

#### **5. 评估指标 (Evaluation)**

任务书明确要求引入 **CAD-MLLM** 的拓扑指标。你需要在验证代码中实现：

  * **SegE (分段错误):** 检查生成的 B-Rep 边的连接数。
  * **DangEL (悬边长度):** 尤其重要，用于检测生成的机械零件是否“漏水”（非封闭流形）。
  * 这些指标在 **CAD-MLLM** 论文中有公式，需要在 `post-processing` 阶段对生成的 Step/Obj 文件进行网格分析计算。

-----

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WHT222)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WHT222)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
