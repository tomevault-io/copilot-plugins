## control-tutorial

> （交流可以用英文，所有文档中文，保留这句）

（交流可以用英文，所有文档中文，保留这句）

## 项目目标
编写一份控制理论的中文教程markdown，要包含大量的习题和参考答案（答案默认折叠）。
重点关注飞机导弹、生产线自动化、自动驾驶和具身智能方向应用
每章尽量包含一个案例，如 spacex 的可重复使用火箭的凸优化和控制率，如 deepmind 核聚变中的控制问题
RL 本身不作重点，主要关注控制问题的提炼、分析和解，以及 lyapunov再设计、代数riccati、滑模控制、MPC渐近稳定性、BSDE用于控制等话题
文件组织是 index.md + chapter1.md + ...
如有数学公式，用 latex.

## Audience
verteran programmer and AI scientists

## 章节结构要求
每个章节应包含：
1. **开篇段落**：简要介绍本章内容和学习目标
2. **文字论述**：以文字论述为主，适当配上公式和 ASCII 图说明。
3. **本章小结**：总结关键概念和公式
4. **练习题**：
   - 每章包含6-8道练习题
   - 50%基础题（帮助熟悉材料）
   - 50%挑战题（包括开放性思考题）
   - 每题提供提示（Hint）
   - 答案默认折叠，不包含代码
5. **常见陷阱与错误** (Gotchas)：每章包含该主题的常见错误和调试技巧
6. **最佳实践检查清单**：每章末尾提供设计审查要点

---
> Source: [zsc/control_tutorial](https://github.com/zsc/control_tutorial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
