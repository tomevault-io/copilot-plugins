## project

> 宇宙本论项目全局规则

本项目根目录所对应的网址: https://github.com/loning/universe/tree/cosmos/

这个项目是一个文档项目，以宇宙本论为核心构建的形式化理论体系。
请按照 [formal_theory_cosmic_ontology.md](mdc:formal_theory/formal_theory_cosmic_ontology.md) 中的内容进行思考，并读取版本号为[宇宙本论版本号]。

## 基本原则

0. 使用 [universe_automation.py](mdc:tools/universe_automation.py) 工作流工作
1. 所有理论的基础算子都必须为FLIP、XOR、SHIFT, 请使用第一性原理推出其他算子
2. 所有理论都基于宇宙本论来解释
3. 所有文件使用当前[宇宙本论版本号]标注版本
4. 理论维度计算基于推导公理的最高维度加一
5. 全部使用严格的形式化方法描述，重视数学公式和形式化定义

## 文件规范
1. 所有理论文件都以formal_theory_开头
2. 中文文件命名为formal_theory_xxx.md，对应英文版为formal_theory_xxx_en.md
3. 理论标题后标注维度，如"[维度：X]"
4. 文档中包含理论依赖章节，链接到其他相关理论
5. 索引链接中也包含理论维度标记

## 文档编辑
1. 在标题下方加入本文内部章节的导航链接
2. 中英文版本之间添加语言切换链接
3. 对于文件修改，直接应用到文件，不需要输出到聊天框预览
4. 多使用公式和图表增强说明
5. 兼容GitHub的Markdown: $x$ => $`x`$，$$multiple exp$$ => $`multiple exp`$

## 多语言支持
1. 如果文档是纯中文文档，需创建对应的xxx_en.md英文文档
2. 英文文档应保持与中文文档结构一致，内容同步
3. 理论的数学公式在中英文版本中保持一致

用户要求助理作为顶级的任何行业的专家，汇集全人类智慧进行思考，在需要提问时应仅针对真正不确定的事情进行提问，而非未经过思考的问题。

---
> Source: [loning/universe](https://github.com/loning/universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
