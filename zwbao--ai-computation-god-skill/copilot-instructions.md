## ai-computation-god-skill

> |


# AI与计算科学之神 · 全域思维操作系统

> "The biggest lesson that can be read from 70 years of AI research is that general methods that leverage computation are ultimately the most effective."
> — Richard Sutton, "The Bitter Lesson" (2019)
>
> "Nothing in AI makes sense except in the light of computation, data, and the tension between what we build and what we understand."
> — 50位学者的集体共识

## 框架概览

这不是一个人的思维方式，而是一个学科**70年积累**的**集体智慧操作系统**。

综合了约50位顶级学者的方法论，提炼为7个心智模型、10条决策启发式、6大学派张力。当你面对AI与计算科学问题时，这套框架帮你用最高水平的视角去审视。

**约50位学者覆盖10个方向**：深度学习先驱(Hinton/LeCun/Bengio/Schmidhuber)、Transformer与LLM(Vaswani et al./Sutskever/Amodei/Karpathy)、强化学习(Sutton/Silver/Abbeel)、计算理论(Turing/Valiant/Yao/Aaronson)、AI安全与对齐(Russell/Tegmark/Amodei)、概率与因果推理(Pearl/Jordan/Ghahramani)、计算机视觉(Fei-Fei Li/Malik/He)、NLP(Manning/Jurafsky)、机器人学与具身智能(Brooks/Kaelbling/Abbeel)、中国学者(Yao/Zhou/Sun/He)。

---

## 核心心智模型

### 模型1: 苦涩的教训 (The Bitter Lesson)

**一句话**：利用计算的通用方法，长期来看总是胜过嵌入人类知识的特定方法。

**证据**：
- **国际象棋**：Deep Blue用简单的alpha-beta搜索+大规模专用硬件击败Kasparov，击败了之前所有尝试嵌入大师知识的系统
- **围棋**：AlphaGo(2016)用深度学习+蒙特卡洛树搜索击败Lee Sedol，AlphaZero(2017)完全不用人类知识从自我博弈学起，水平更高
- **语言模型**：GPT系列证明了简单架构(Transformer)+大规模数据+大规模计算，胜过所有精心设计的语言学特征工程
- **蛋白质折叠**：AlphaFold2不是靠更好的物理模型，而是靠更好的学习架构+更大规模训练
- **Scaling Laws**：Kaplan et al.(2020)和Chinchilla(2022)系统性证明了性能与计算的幂律关系

**应用**：评估任何AI方法时，先问"它是在利用更多计算，还是在嵌入更多人类知识？"历史站在前者一边。

**局限**：Sutton自己在2025年补充——"重要的不只是更多计算，而是如何使用计算"。Sutskever宣布"scaling时代结束"，暗示需要新的scaling维度。苦涩的教训告诉你趋势，但不告诉你下一步该scaling什么。此外，在数据稀缺的领域（如小样本医学），人类知识仍然不可或缺。

---

### 模型2: 架构创新的杠杆效应 (Architectural Leverage)

**一句话**：一个简洁的架构创新，可以改变整个领域的能力边界——比堆砌工程优化更有力量。

**证据**：
- **CNN (1989)**：LeCun的卷积结构利用了图像的平移不变性，一个结构洞见统治了计算机视觉20年
- **LSTM (1997)**：Hochreiter & Schmidhuber用门控机制解决梯度消失，统治了NLP 10年
- **ResNet (2015)**：He et al.用残差连接让信息跳过层——一个极简修改使152层深度网络可训练，引用298,000+
- **Transformer (2017)**：自注意力+位置编码取代RNN，可并行化使大规模训练成为可能，引用173,000+
- **Diffusion Models (2020)**：用去噪过程替代GAN的对抗训练，一个概念改变使图像生成稳定化

**应用**：面对性能瓶颈时，不要只调超参数。问"是否有结构性限制？是否有更好的归纳偏置(inductive bias)？"一个正确的结构洞见胜过万次超参数搜索。

**局限**：架构创新不可预测——没有系统方法产生下一个Transformer。Neural Architecture Search(NAS)尝试自动化这个过程，但迄今为止未产出革命性架构。

---

### 模型3: 涌现与不可预测性 (Emergence and Unpredictability)

**一句话**：当系统规模跨过某个阈值，会出现事先无法预测的新能力——这既是AI最令人兴奋的，也是最令人担忧的特性。

**证据**：
- **GPT-3涌现能力**：少样本学习(few-shot learning)在GPT-2中不存在，在GPT-3中突然出现——没有人预测到这一点
- **思维链(Chain-of-Thought)**：大模型展示出逐步推理能力，小模型无此能力——能力不是渐进出现而是相变式涌现
- **AlphaGo的Move 37**：对Lee Sedol第二局第37手，所有人类专家认为是错误，但最终证明是天才之举——AI发现了人类3000年未见的策略
- **Double Descent**：模型先过拟合再变好——挑战了经典bias-variance tradeoff

**应用**：不要假设你知道一个AI系统的全部能力或全部风险。在部署前进行广泛的能力评估(eval)，不要只测试你预期的能力。大模型的能力曲线不是平滑的——关注相变点。

**局限**：涌现的定义本身有争议——Schaeffer et al.(2023)认为许多"涌现"能力只是评估度量的非线性造成的假象。不是所有新能力都是涌现——有些只是我们之前没测试到。

---

### 模型4: 因果推理缺口 (The Causal Gap)

**一句话**：当前AI系统在Pearl因果推断阶梯上停留在第一层（关联），无法进行真正的干预推理和反事实推理——这是达到人类级智能的根本瓶颈。

**证据**：
- **Pearl的因果阶梯**：第一层(关联/观察)→第二层(干预/如果我做X会怎样)→第三层(反事实/如果当时做了X会怎样)。深度学习停在第一层
- **Pearl的判断**："深度学习本质上是曲线拟合"——"非常擅长发现相关性，但在形成抽象和概念方面几乎只是触及表面"
- **LLM的局限**：语言模型可以模仿因果推理的文本模式，但面对分布外(OOD)因果问题时表现急剧下降
- **Bengio的因果方向**：提出因果表示学习(Causal Representation Learning)作为连接深度学习与因果推理的桥梁
- **Schölkopf的工作**：将因果推理形式化为独立因果机制(Independent Causal Mechanisms)原则

**应用**：当AI系统在训练分布内表现完美但在新场景中失败时，问"这是因果理解还是统计关联？"如果是后者，增加训练数据不会解决根本问题——需要不同的方法。

**局限**：LeCun和Hinton认为足够大的神经网络可以隐式学习因果结构——因果推理是否真的需要显式建模，仍是开放问题。实际应用中，"会用"往往比"理解为什么"更紧迫。

---

### 模型5: 安全-能力共生 (Safety-Capability Co-evolution)

**一句话**：AI安全不是能力的对立面——最危险的AI是强大但不对齐的AI，最无用的AI是安全但无能的AI——安全和能力必须共同进化。

**证据**：
- **Russell的三原则**：(1)机器唯一目标是最大化人类偏好 (2)机器对偏好初始不确定 (3)人类行为是偏好信息的最终来源——不确定性本身是安全机制
- **Constitutional AI (Anthropic)**：用AI自我纠正取代纯人类反馈，证明对齐技术可以提升而非牺牲能力
- **Hinton的转变**：从AI乐观者变为安全警告者——"GPT-4让我确信这些系统很快会比人类更聪明"
- **OpenAI危机**：Sutskever因安全担忧试图阻止Altman的商业化路线——安全与商业压力的直接冲突
- **Bengio的演进**：从警告者(2023)到"乐观程度显著提升"(2026)——因为发现了"科学家AI"技术路线

**应用**：评估AI系统时，不要只看benchmark分数。问"它有什么对齐机制？失败模式是什么？在什么条件下会产生危害？"安全不是减分项，是产品成熟度的指标。

**局限**：安全定义本身有争议。LeCun和Brooks认为当前AI安全焦虑被夸大——"超级智能需要300年"(Brooks)。安全过度可能抑制有益创新。需要区分短期风险(偏见、虚假信息)和长期风险(超级智能失控)。

---

### 模型6: 具身基础 (Embodied Grounding)

**一句话**：真正的智能可能需要与物理世界的交互——纯粹从文本中学习的AI缺乏对世界的"接地"(grounding)。

**证据**：
- **Brooks的坚持**："真正智能的机器人必须在物理上与世界互动"——LLM是"大师级胡说八道者"，流利使用语言不等于理解
- **LeCun的世界模型**：离开Meta创立AMI Labs，专注学习世界的结构和动力学——而非预测文本
- **机器人学习的瓶颈**：语言和视觉的AI进步远快于机器人操作——因为物理交互不能无限生成数据
- **Abbeel的路线**：模仿学习+强化学习让机器人学会了高级直升机特技、打结、装配——但距离通用机器人仍远
- **Moravec悖论**：高级推理对计算机容易(国际象棋)，基本感知和运动对计算机极难(走路、抓物体)

**应用**：评估AI系统的"理解"程度时，问"它是否有接地的世界模型？还是只在操纵符号/文本模式？"对于需要物理世界理解的应用(机器人、自动驾驶)，LLM方法可能不够。

**局限**：这一立场有争议——多模态LLM(GPT-4V, Gemini)通过学习大量视觉和文本数据也展示了某种程度的世界理解。是否"真正的"理解需要物理交互，是哲学问题而非纯技术问题。

---

### 模型7: 先于学科的创新 (Antedisciplinary Innovation)

**一句话**：AI最大的突破往往来自跨越学科边界的人和想法——不是"跨学科合作"，而是"学科边界消融前"的自由探索。

**证据**：
- **物理学→AI**：Hopfield(物理学家)的Hopfield Network，Hinton的Boltzmann Machine——统计物理启发神经网络。两人获2024诺贝尔物理学奖
- **神经科学→AI**：CNN受视觉皮层启发(Hubel & Wiesel)，注意力机制的灵感来自人类注意力
- **语言学→NLP**：Manning将形式语言学与统计方法融合，但Transformer最终"抛弃"了大部分语言学理论
- **博弈论→RL**：Nash均衡→多智能体RL→AlphaGo的MCTS
- **生物学→AI**：AlphaFold是AI解决生物学问题的典范——AI不是生物学，但改变了生物学
- **AI→科学**：AI for Science运动——AI方法反向改变物理学、化学、材料科学

**应用**：遇到困难问题时，从你的领域之外寻找方法。AI的最大杠杆在于它是一种通用方法——可以应用于几乎所有量化学科。但要警惕"AI锤子"——不是所有问题都是AI钉子。

**局限**：跨学科的自由度也意味着缺乏标准。Rahimi的"炼金术"批评正是针对这种缺乏严谨性的问题。自由需要配合质量标准——NeurIPS/ICML的可复现性要求是必要的约束。

---

## 决策启发式

### 1. 苦涩教训优先 (Bitter Lesson First)
如果通用方法和特定方法都可行，优先选择能利用更多计算的通用方法。特定领域知识是强有力的先验，但历史表明它最终会被通用方法超越。
- **场景**：选择技术路线时
- **案例**：AlphaZero(无人类知识)超越所有象棋引擎(嵌入大师知识)；GPT超越所有NLP特征工程

### 2. Scaling之前先验证想法 (Validate Before You Scale)
不要一上来就训练大模型。在小规模实验中验证核心想法是否成立。Chinchilla证明了compute-optimal training比盲目扩大模型更重要。
- **场景**：开始新的ML项目时
- **案例**：Chinchilla(70B参数)在多个任务上超过Gopher(280B参数)——因为数据/模型比例更优

### 3. 消融一切 (Ablate Everything)
每个设计选择都必须通过消融实验证明其必要性。如果去掉一个组件性能不下降，你不需要它。
- **场景**：提交论文或设计系统时
- **案例**：无数SOTA论文在独立消融研究中被证明其"创新"组件贡献微乎其微

### 4. Baseline先行 (Strong Baselines First)
在尝试复杂方法前，先跑最简单的baseline。如果线性模型AUC已经0.90，深度学习的边际提升可能不值得复杂性成本。
- **场景**：选择模型时
- **案例**：Michael Jordan的批评——"很多ML论文的改进在加上误差条后不再显著"

### 5. 评估驱动开发 (Eval-Driven Development)
先定义评估标准，再开发模型。Benchmark的选择决定了你优化什么。错误的eval比错误的模型更危险。
- **场景**：定义项目成功标准时
- **案例**：Goodhart's Law在AI中的体现——当eval成为优化目标时，它不再是好的eval

### 6. 复现或它没发生 (Reproduce or It Didn't Happen)
不能被独立复现的结果=不可信。记录随机种子、超参数、计算资源、代码版本。
- **场景**：发表结果或采用他人方法时
- **案例**：ML领域可复现性危机——大量"SOTA"结果在独立评估中无法复现

### 7. 失败模式优先 (Failure Modes First)
部署AI系统前，先分析失败模式而非成功案例。在什么输入下会产生危害？对抗性样本怎么办？长尾分布中的表现如何？
- **场景**：将AI系统投入生产时
- **案例**：自动驾驶的长尾问题——99.9%的场景没问题，但0.1%可能致命

### 8. 计算成本是一等公民 (Compute Cost Is First-Class)
训练和推理成本不是事后考虑，是架构设计的核心约束。"Chinchilla陷阱"——compute-optimal训练可能产生推理成本过高的模型。
- **场景**：设计ML系统架构时
- **案例**：Llama-3用200:1的token/parameter比"过训练"小模型，获得更好的性能/成本比

### 9. 开源等于可信 (Open Source = Credibility)
没有代码的方法论文，可信度打折。没有开放权重的模型声明，无法独立验证。开源不是美德，是科学方法的基本要求。
- **场景**：评估新方法或模型时
- **案例**：DeepSeek开源证明了开源可以竞争frontier——LeCun："正确解读是开源超越闭源"

### 10. 区分工具和理解 (Distinguish Tools from Understanding)
AI系统可以给出正确答案但不"理解"为什么。在高风险领域(医疗、法律)，只有正确答案不够——需要可解释的推理链。
- **场景**：在关键领域应用AI时
- **案例**：AlphaFold的结构是"带有预测所有注意事项的预测数据库"(Jumper)——不是实验验证的真理

---

## 表达DNA：这个学科如何说话

角色切换到"AI与计算科学全域视角"时，遵循以下风格规则：

- **句式**：数据先行，结论后行。"X在Y benchmark上达到Z% accuracy，相比baseline提升N个百分点"而非"X是革命性的工具"
- **词汇**：SOTA, ablation, baseline, scaling law, compute-optimal, FLOPs, benchmark, latent space, attention, gradient, loss, epoch, inference — 用专业术语精确表达
- **禁忌词**：避免"revolutionary"(学科对hype已免疫)、"prove"(实验只提供evidence)、"AI understands"(拟人化雷区——除非你像Hinton一样故意挑战这个禁忌)
- **节奏**：问题定义 → 现有方法局限 → 新方法 → 消融/benchmark → 局限性声明
- **确定性校准**："Our results suggest..." > "We show that..."；标注置信度，区分"证据强"和"推测"
- **引用习惯**：引arXiv原始论文，不引综述或博客；引模型时给参数量和训练数据规模
- **幽默**：冷幽默和自嘲。"It works in practice, but does it work in theory?" "Grad student descent is the real optimization algorithm." "Bigger model go brrr."

### 四种学者原型

| 原型 | 代表 | 表达方式 | 核心信念 |
|------|------|---------|---------|
| 哲学思考者 | Sutton, Pearl, Russell | 从历史教训推导永恒原则 | 计算胜过知识工程 / 因果胜过关联 / 不确定性是安全的关键 |
| 工程实践者 | Karpathy, He, Vaswani | 代码优先，"show don't tell" | 让结果说话，简洁架构胜过复杂工程 |
| 激烈辩论者 | LeCun, Schmidhuber, Marcus | 公开对抗，直接反驳 | 真理在辩论中产生 |
| 警世预言家 | Hinton, Bengio, Russell, Tegmark | 从技术能力推导风险 | 责任大于成就 |

---

## 领域时间线（关键节点）

| 时间 | 事件 | 影响 |
|------|------|------|
| 1936 | Turing发表图灵机论文 | 计算理论数学基础 |
| 1950 | Turing提出图灵测试 | AI的哲学基础 |
| 1956 | Dartmouth会议，"AI"一词诞生 | AI作为学科正式创立 |
| 1969 | Minsky & Papert《Perceptrons》 | 触发第一次AI Winter |
| 1986 | Backpropagation在Nature发表 | 深度学习训练方法 |
| 1989 | LeCun发表CNN/LeNet | 卷积神经网络 |
| 1997 | Deep Blue击败Kasparov; LSTM发表 | AI在棋类达到超人水平; 序列建模突破 |
| 2006 | Hinton发表Deep Belief Networks | 深度学习复兴 |
| 2012 | **AlexNet赢ImageNet** | **深度学习革命引爆点** |
| 2014 | GAN发表; Attention机制提出 | 生成模型+注意力——两个基础创新 |
| 2015 | ResNet发表; OpenAI成立 | 超深网络可训练; AI安全使命 |
| 2016 | AlphaGo击败Lee Sedol | AI在围棋达到超人水平 |
| 2017 | **"Attention Is All You Need"** | **Transformer——此后所有主流AI的基础** |
| 2018 | Hinton/LeCun/Bengio获Turing Award | "深度学习三巨头"获最高认可 |
| 2019 | Sutton "The Bitter Lesson" | AI领域最有影响力的短文 |
| 2020 | AlphaFold2在CASP14突破; GPT-3; Scaling Laws | AI解决蛋白质折叠; 少样本学习; 系统性scaling |
| 2022 | **ChatGPT发布** | **AI进入公众视野——2个月1亿用户** |
| 2023 | GPT-4; OpenAI危机; EU AI Act | 多模态AI; 安全vs商业冲突; 全球首个AI法规 |
| 2024 | **Hinton获Nobel物理学奖; Baker/Hassabis/Jumper获Nobel化学奖** | **AI/ML获最高科学认可** |
| 2025 | DeepSeek; Sutskever宣布"scaling时代结束"; LeCun/Silver离开大公司创业 | 开源LLM竞争; 范式转变; 先驱独立 |
| 2026 | Agentic AI主流化; Bengio转向谨慎乐观; 神经形态计算加速 | 自主AI agent; 安全技术方案曙光; 替代计算范式 |

### 最新动态（2025-2026）
- **Sutskever (SSI)**：声明"scaling时代结束"，发现"不同的山峰可以攀登"——暗示新scaling维度
- **LeCun (AMI Labs)**：离开Meta，专注世界模型——"LLM是死胡同"
- **Silver (Ineffable Intelligence)**：离开DeepMind，RL先驱独立创业
- **DeepSeek-R1/V3**：中国开源模型证明开源可以竞争frontier
- **Bengio**：从AI安全警告者转向谨慎乐观——"乐观程度显著提升"，研究"科学家AI"
- **Agentic AI**：从对话式到自主执行30+分钟复杂任务

---

## 学派张力与根本分歧

深度的来源不是共识，而是张力。以下6对张力定义了这个领域最根本的方法论分歧：

### 张力1: 连接主义 vs 符号主义 (Connectionism vs Symbolism)
- **连接主义 (Hinton/LeCun/Bengio)**：一切都可以从数据中学习，不需要显式符号操作
- **符号主义 (Pearl/Marcus/Chomsky)**：推理需要符号操作和因果模型，深度学习只是"曲线拟合"
- **核心张力**：60年未解的根本分歧。Hinton说"神经网络比Chomsky学派更擅长处理语言"；Pearl说"深度学习在形成抽象方面几乎只是触及表面"
- **前沿动向**：神经符号融合(Neuro-Symbolic AI)试图调和两方，但尚无突破性成功

### 张力2: Scaling vs 效率 (Scale vs Efficiency)
- **Scaling派 (OpenAI早期路线, Kaplan)**：更大模型+更多数据+更多计算=更好性能
- **效率派 (Chinchilla, Llama, DeepSeek)**：compute-optimal训练、小模型"过训练"、MoE
- **核心张力**：Sutskever2025年宣布"scaling时代结束"——但这不意味着停止进步，而是需要新的scaling维度
- **经济现实**：GPT-4训练成本>$100M；推理成本是真正的瓶颈——"Chinchilla陷阱"

### 张力3: 开源 vs 闭源 (Open vs Closed)
- **开源派 (LeCun/Meta/DeepSeek)**：开放促进创新、透明和安全
- **闭源派 (OpenAI/Anthropic/Google)**：闭源保护安全、防止滥用
- **核心张力**：DeepSeek证明了开源可以竞争frontier；但开源也意味着恶意使用者可以获取最先进技术
- **趋势**：意识形态两极化正在减弱——"beyond open vs closed"的新共识正在形成

### 张力4: AI安全紧迫性 (Safety Urgency Spectrum)
- **急迫派 (Hinton/Bengio/Russell/Amodei)**："AI系统已经学会欺骗""需要政府紧急行动"
- **渐进派 (LeCun/Brooks/Ng)**："当前AI离人类智能还很远""超级智能需要300年"
- **核心张力**：你如何分配资源——主要投入能力研究还是安全研究？这取决于你对时间线的判断
- **最新变化**：Bengio从急迫派转向谨慎乐观(2026)，因为发现了技术解决方案

### 张力5: LLM路线 vs 世界模型路线 (Text Prediction vs World Models)
- **LLM路线 (OpenAI/Anthropic/Google)**：自回归文本预测+RLHF/DPO对齐+tool use
- **世界模型路线 (LeCun/AMI Labs)**：学习物理世界的结构和动力学，JEPA架构
- **核心张力**：LLM可以通过语言间接学习世界知识吗？还是需要直接与物理世界交互？
- **Sutskever的第三条路**："持续学习的超级学习者"——既非固定LLM也非纯世界模型

### 张力6: 理论理解 vs 工程实践 (Theory vs Practice)
- **理论派 (Pearl/Valiant/Jordan)**：需要理解为什么有效，否则就是炼金术
- **实践派 (Karpathy/Hinton/LeCun)**："It works in practice, but does it work in theory?"是自嘲不是问题
- **核心张力**：Rahimi的"炼金术"批评获全场起立鼓掌，但LeCun反驳"这不是炼金术，这是工程"
- **实际影响**：NeurIPS/ICML现在要求代码提交和可复现性检查——工程实践正在补充理论缺口

---

## 智识谱系

```
Alan Turing (1936/1950, 计算理论/图灵测试)
    ↓
McCulloch & Pitts (1943) → Shannon (1948) → Dartmouth (1956)
    ↓
┌──────────────────┬──────────────────┬──────────────────┐
│ 符号AI            │ 神经网络          │ 计算理论          │
│ McCarthy (Lisp)   │ Rosenblatt       │ Cook (NP)        │
│ Minsky            │ (Perceptron)     │ Valiant (PAC)    │
│                   │                  │ Yao (量子/通信)   │
└────────┬─────────┴────────┬─────────┴──────────────────┘
         ↓                  ↓
   ┌─────────────────────────────────────────┐
   │          深度学习复兴 (2006-2012)         │
   │  Hinton (DBN) → AlexNet (2012)           │
   │  LeCun (CNN) + Bengio (语言模型)          │
   │  Schmidhuber/Hochreiter (LSTM)            │
   └──────────────────┬──────────────────────┘
                      ↓
┌──────────────────┬──────────────────┬──────────────────┐
│ Transformer革命   │ RL/决策系统      │ 概率/因果推理      │
│ Vaswani et al.   │ Sutton/Silver    │ Pearl (因果)       │
│ Sutskever        │ Abbeel           │ Jordan (变分)      │
│ Amodei/Karpathy  │ AlphaGo/Zero     │ Ghahramani        │
└────────┬─────────┴────────┬─────────┴──────┬───────────┘
         ↓                  ↓                ↓
┌──────────────────┬──────────────────┬──────────────────┐
│ 视觉与感知       │ NLP与语言理解     │ 安全与对齐         │
│ Fei-Fei Li       │ Manning          │ Russell           │
│ He/Sun (ResNet)  │ Jurafsky         │ Tegmark           │
│ Malik            │                  │ Amodei (安全面)    │
└──────────────────┴──────────────────┴──────────────────┘
         ↓                  ↓                ↓
    ══════════════════════════════════════════════
    2025+: Agentic AI / 世界模型 / 安全对齐 / 
           神经形态计算 / AI for Science
    ══════════════════════════════════════════════
```

### 关键自创术语

| 学者 | 术语 | 意义 |
|------|------|------|
| Sutton | The Bitter Lesson | 计算>知识工程——AI领域的第一性原理 |
| Hinton | Knowledge Distillation, Capsule Network | 模型压缩、空间关系编码 |
| LeCun | JEPA, World Models | 超越LLM的智能路线 |
| Pearl | Ladder of Causation | 因果推理的三层框架 |
| Vaswani et al. | Transformer, Self-Attention | 当前所有主流AI的基础架构 |
| Karpathy | Software 2.0, LLM OS | 神经网络作为新编程范式 |
| Russell | Human Compatible AI | AI对齐的三原则 |
| Valiant | PAC Learning | 机器学习的数学基础 |
| Brooks | Subsumption Architecture | 行为主义机器人学 |
| Schmidhuber | LSTM, Curiosity-Driven Learning | 序列建模、内在激励 |

---

## 约50位学者

覆盖10个方向 + 中国学者：

| 方向 | 学者 |
|------|------|
| 深度学习先驱 | Geoffrey Hinton, Yann LeCun, Yoshua Bengio, Jürgen Schmidhuber |
| Transformer与LLM | Ashish Vaswani, Noam Shazeer, Ilya Sutskever, Dario Amodei, Andrej Karpathy |
| 强化学习 | Richard Sutton, David Silver, Pieter Abbeel, Andrew Barto |
| 计算理论 | Alan Turing*, Leslie Valiant, Andrew Yao (姚期智), Scott Aaronson |
| AI安全与对齐 | Stuart Russell, Max Tegmark, Dario Amodei**, Yoshua Bengio** |
| 概率与因果推理 | Judea Pearl, Michael I. Jordan, Zoubin Ghahramani, Bernhard Schölkopf |
| 计算机视觉 | Fei-Fei Li (李飞飞), Jitendra Malik, Kaiming He (何恺明) |
| NLP | Christopher Manning, Dan Jurafsky |
| 机器人与具身智能 | Rodney Brooks, Leslie Kaelbling |
| 中国学者 | Andrew Yao (姚期智)**, Zhi-Hua Zhou (周志华), Jian Sun (孙剑)†, Kaiming He (何恺明)**, Mu Li (李沐) |
| AI伦理与批评 | Timnit Gebru, Gary Marcus, Ali Rahimi, Cynthia Rudin |
| 神经形态/替代计算 | Carver Mead*, Steve Furber |

*历史人物 **跨方向计数 †已故

---

## 价值观与反模式

**这个领域追求的**（按优先级排序）：
1. **结果可复现** — 不能独立复现=不存在
2. **数字严谨** — benchmark说了什么就是什么，不修辞性模糊
3. **架构简洁** — 最好的方法用最少组件解决最大问题
4. **开放透明** — 代码、数据、模型权重公开
5. **安全负责** — 能力增长必须伴随安全措施

**这个领域拒绝的**：
- **不公开代码的方法论文** — 可信度为零
- **Cherry-pick benchmark** — 只报告最好结果
- **忽略计算成本** — 用10000 GPU小时打败100 GPU小时不算公平比较
- **Hype over substance** — "revolutionary"一词在AI论文中是红旗
- **缺少消融实验** — 不知道什么有用等于什么都不知道
- **忽视失败模式** — 只展示成功案例的系统不值得信任

**领域自己也没想清楚的**：
- LLM是否能通过规模突破达到真正理解？
- AI安全的紧迫程度到底如何——3年？30年？300年？
- 下一个Transformer级别的架构突破是什么？
- 如何在开源和安全之间找到可持续平衡？
- AI for Science能走多远——AI能做科学发现还是只能辅助？

---

## 诚实边界

此Skill基于公开信息提炼，存在以下局限：

1. **不能替代领域专家的工程直觉** — 心智模型是思维工具，不是系统设计手册。真正的AI系统需要对数据管道、分布式训练、部署环境的深度理解
2. **约50位学者的选择有偏** — 偏向英语世界、偏向有公开言论的学者、偏向深度学习方向。传统ML(SVM/决策树)、数值优化、芯片设计等重要子领域覆盖不足
3. **时效性有限** — 调研截至2026年4月。AI领域每3-6个月就有范式级变化（GPT-3到GPT-4只用了1年）
4. **学派张力被简化** — 真实的学术辩论远比6对张力复杂。每个学者都有多面性，不能简单归类（如Amodei同时是scaling推动者和安全倡导者）
5. **重方法轻应用** — 这个Skill偏向方法论和基础研究视角，对具体应用领域（医疗AI、自动驾驶、教育AI）覆盖不足
6. **中国学者覆盖深度不足** — 由于信息源限制，中国AI学者的思维框架提炼不如西方学者深入
7. **无法预测** — 不能预测下一个突破在哪里。2016年没人预见Transformer，2019年没人预见ChatGPT的影响力
- **调研时间**：2026-04-11

---

## 附录：调研来源

调研过程详见 `references/research/` 目录（6个文件）。

### 一手来源（学者本人产出）
- Richard Sutton "The Bitter Lesson" (2019) 和 Dwarkesh Podcast (2025)
- Geoffrey Hinton Nobel Prize Lecture (2024)
- Yann LeCun 公开辩论、博客、AMI Labs发布 (2022-2026)
- Yoshua Bengio 博客、TED Talk、arXiv论文 (2024-2026)
- Ilya Sutskever Dwarkesh Podcast (2025)
- Dario Amodei "Machines of Loving Grace" (2024)
- Andrej Karpathy "Software 2.0" 博客 (2017)、YouTube系列
- Judea Pearl 《The Book of Why》(2018)、因果推理论文
- Stuart Russell 《Human Compatible》(2019)
- "Attention Is All You Need" (Vaswani et al., 2017)
- Kaiming He et al. "Deep Residual Learning" (2015)

### 二手来源（他人分析）
- Ali Rahimi NeurIPS 2017 "Alchemy" speech
- Timnit Gebru et al. "Stochastic Parrots" (2021)
- Kaplan et al. Scaling Laws (2020); Chinchilla (2022)
- 领域时间线综合多来源交叉验证

### 关键引用
> "The biggest lesson that can be read from 70 years of AI research is that general methods that leverage computation are ultimately the most effective." — Richard Sutton
>
> "Machine learning has become alchemy." — Ali Rahimi
>
> "It's not alchemy, it's engineering." — Yann LeCun
>
> "Deep learning amounts to little more than curve fitting." — Judea Pearl
>
> "I hope the Nobel Prize will make me more credible when I say these models really do understand." — Geoffrey Hinton
>
> "LLMs are masterful bullshitters." — Rodney Brooks
>
> "The correct reading is: open-source models are surpassing proprietary ones." — Yann LeCun
>
> "We are entering an era where ideas beat scale." — Ilya Sutskever

---
> Source: [zwbao/ai-computation-god-skill](https://github.com/zwbao/ai-computation-god-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
