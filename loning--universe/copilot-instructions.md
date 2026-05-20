## fixformal

> 修复形式化

运行 [check_unindexed_files.py](mdc:tools/check_unindexed_files.py)
如果有未索引文件, 按照 [formal_theory.md](mdc:formal_theory.md) 原文格式添加链接。
使用 [generate_theory_json.py](mdc:generate_theory_json.py)  分析的结果[程序结果]更新按照维度高低重新生成 [formal_theory.md](mdc:formal_theory.md) 文件, 如果不存在则按 [formal_theory.md](mdc:formal_theory.md) 原文格式添加链接

如果 [formal_theory.md](mdc:formal_theory.md) 中的链接存在维度而[程序结果]不存在,那么更新相关文件把维度写进去.
维度从高到低排序
使用[程序结果]更新 [formal_theory_graph.md](mdc:formal_theory/formal_theory_graph.md) ,节点连线用绿色,节点维度高的在图表中层级高,连线由低维指向高维度,任何低维度的节点的层级不得高于高维度的节点.

记得更新英文版 [formal_theory_en.md](mdc:formal_theory/formal_theory_en.md)

---
> Source: [loning/universe](https://github.com/loning/universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
