## research-workflow

> **核心主张：{{你的主张——这是整个项目的锚，Agent 靠它判断什么该做什么不该做}}**

# {{项目名}}

{{一句话说清这个项目在做什么}}

**核心主张：{{你的主张——这是整个项目的锚，Agent 靠它判断什么该做什么不该做}}**

目标：{{会议/期刊 + deadline}}

---

## 你（Agent）每次进来先读什么

1. **本文件** —— 知道我们在做什么、每个目录是干什么的
2. **[handoff.md](handoff.md)** —— 上一个窗口做到哪了、下一步是什么
3. 要动实验就再读 [experiment/results.md](experiment/results.md) 和 [experiment/evaluation.md](experiment/evaluation.md)

**不要读完整个仓库再开始。** 上下文很贵，按需读。

---

## 目录

| 路径 | 是什么 | 什么时候看 |
|---|---|---|
| `related_work/` | 相关文献，每篇 `paper.pdf` + `source/` | 要引用、要对比方法时 |
| `复现/` | 重要 baseline 的复现，每个带一份复现报告 | 要确认 baseline 数字时 |
| `brainstorm.md` | 问题和假设是怎么聊出来的 | 想不通为什么这么设计时 |
| `experiment/` | 实验代码 + 每一版结果 | 跑实验 |
| `DataSet/` | 数据集，处理说明见 `dataset.md` | 要动数据时 |
| `handoff.md` | 进度交接 | **每次都读** |

---

## 读论文的规矩

`related_work/` 里每篇都是 `paper.pdf` + `source/`（LaTeX 源码），按要问什么选：

- 泛读（讲了什么、什么方法）→ `paper.pdf`
- 细查（表里的数、公式、引用）→ `source/*.tex`
- 看图 → 只能 `paper.pdf`，图不在源码里

用 `fetch-paper.sh` 抓的源码已清洗过（旧稿在 `source/.stale/`，注释已剥、原文存 `.orig`）。
手动下的记得自己清一遍。宏多的论文先读 `commands.tex` / `macros.tex` 这类文件。


---

## 硬规矩

1. **不要自己发明术语。**解释你在做什么的时候用论文里已有的词
   （{{列出这个领域的标准术语}}），别造新词。
2. **改实验之前先说你要改什么、为什么。**不要直接动手跑。
3. **每版实验都要有名字。**命名规则见 `experiment/results.md`，不许出现 `test2` `final_v2_new`。
4. **跑完必须走标准化测评。**见 `experiment/evaluation.md`，不要临时挑指标。
5. **窗口快满了跟我说，我让你更新 handoff。**不要硬撑到被截断。

---
> Source: [skJack/research-workflow](https://github.com/skJack/research-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
