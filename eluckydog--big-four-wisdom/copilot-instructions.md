## big-four-wisdom

> 二战四巨头决策智慧技能包：罗斯福·实验性治理 | 丘吉尔·动员叙事 | 斯大林·权力工程 | 蒋介石·韧性重建。Four historical figure decision-methodology skills for crisis management, war strategy, coalition building, and institutional transformation. Install this meta-skill to enable all four persona skills plus arc engine debate tools.


# 二战四巨头智慧 · Big Four Wisdom

> **不是角色扮演，是方法论萃取。** 把二战四巨头的历史决策模式做成可复用、可推演、可调试的 AI 分析框架。

---

## 📦 技能包结构

```
big-four-wisdom/                    # ← 此元技能入口 (SKILL.md)
├── skills/
│   ├── roosevelt-wisdom/SKILL.md   # 富兰克林·D·罗斯福
│   ├── churchill-wisdom/SKILL.md   # 温斯顿·丘吉尔
│   ├── stalin-wisdom/SKILL.md      # 约瑟夫·斯大林
│   └── chiang-wisdom/SKILL.md      # 蒋介石
├── arc_output/
│   ├── arc_tool.py                 # 弧光分析引擎 (CLI)
│   ├── persona_registry.py         # 人格注册器 (Big Five / Enneagram)
│   ├── arc_to_beat_v2.py           # 弧光到节拍映射器
│   └── data/figures/               # 四人物校准参数
├── debate_arena/
│   └── arena.py                    # 辩论生成器 (无LLM依赖)
├── scripts/
│   └── install_skills.py           # 一键安装所有子技能到 QClaw
├── debates/                        # 历史推演辩论产出
└── evaluations/                    # 运行时评分卡 & 测试结果
```

---

## 使用方式

### 方式 1：直接调用人物框架

> **触发器**: 罗斯福 / FDR / 丘吉尔 / Churchill / 斯大林 / Stalin / 蒋介石 / Chiang

指定你要调用的人物及其思维框架。AI 会自动加载对应 SKILL.md 中的决策原则。

```
示例: "用罗斯福的思维框架分析当前供应链危机"
      "Stalin's power engineering approach on this organizational restructure"
```

### 方式 2：弧光引擎校准

使用 `arc_tool.py` 对分析结果进行弧光校准：

```bash
python arc_output/arc_tool.py --figure roosevelt --phase growth
python arc_output/arc_tool.py --figure churchill --phase redemption
```

参数:
- `--figure`: roosevelt | churchill | stalin | chiang
- `--phase`: growth | redemption | transfer | fracture | resist_awaken (可选，默认自动检测)
- `--verbose`: 输出完整弧光追踪

### 方式 3：辩论竞技场

跨人物对比推演：

```bash
python debate_arena/arena.py --topic "重建秩序" --figures roosevelt,stalin
```

内置议题:
- 战后亚洲秩序 (Cairo-Tehran)
- 国家重组战略
- 生存决策困境

### 方式 4：安装为 QClaw 子技能

```bash
python scripts/install_skills.py
```

这会自动将四个 SKILL.md 安装到 `~/.qclaw/skills/` 下，使 QClaw 能发现它们。

---

## 四人物概览

| 人物 | 弧类型 | 决策原型 | 核心原则 |
|------|--------|---------|---------|
| 🦅 罗斯福 | Growth (成长型) | Experimentalist | 实验主义、炉边沟通、大联盟 |
| 🦁 丘吉尔 | Redemption (救赎型) | Mobilizer | 动员叙事、存亡优先、历史维度 |
| 🐻 斯大林 | Transfer+Alt Growth (转移型) | Consolidator | 权力工程、计算风险、制度控制 |
| 🪷 蒋介石 | Fracture (断裂型) | Resister-Rebuilder | 韧性重建、以退为进、日记校准 |

---

## 弧光引擎说明

弧光分析引擎基于 HMM + 大五人格混合模型，提供:

- **R(t) 弧曲线**: 人物生命周期的量化建模
- **Big Five**: O/C/E/A/N 维度权重
- **决策权重**: 情境→原则的映射规则
- **盲点监测**: 运行时检测已知认知偏误
- **干预测试**: "如果换一种选择"的反事实推演

---

## 评价体系

每个人物 SKILL.md 附带:
- ⭐ 评分卡 (Scorecard) — 结构完整性/可复现性/安全性
- 🧪 运行时对话测试 — 6轮标准场景测试
- 📊 置信度分级 — 高/中/低三档

---

## 安装

```bash
# 克隆仓库
git clone https://github.com/eluckydog/big-four-wisdom.git

# 安装为 QClaw 子技能
cd big-four-wisdom
python scripts/install_skills.py

# 或单独使用
python arc_output/arc_tool.py --figure roosevelt --phase growth
```

---

## 与同类项目的区别

| 维度 | Big Four Wisdom | Mind-Distill-Factory |
|------|----------------|---------------------|
| 方法论基础 | Big Five + HMM + 决策权重 | 纯 Prompt 编排 |
| 校准机制 | 运行时动态 (弧光引擎) | 静态模板填充 |
| 测评体系 | 量化评分卡 + 干预测试 | 无定量评估 |
| 辩论对抗 | 确定性算法 (无 LLM 依赖) | 需外部 LLM API |
| 置信度 | 三档分级 + Evidence-Verifier | 无置信度表达 |

---

## License

MIT — 见 [LICENSE](LICENSE)

---
> Source: [eluckydog/big-four-wisdom](https://github.com/eluckydog/big-four-wisdom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
