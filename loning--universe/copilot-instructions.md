## synclanguage

> 同步中英文翻译

执行 [check_updates.py](mdc:tools/check_updates.py) 检查需要更新的文件为[目标文件]
先更新中英文件都存在但是英文版没有与中文版同步的[目标文件]
如果[目标文件]版本小于 [formal_theory_cosmic_ontology.md](mdc:formal_theory/formal_theory_cosmic_ontology.md) 的[核心版本号], 则用[formal_theory_cosmic_ontology.md](mdc:formal_theory/formal_theory_cosmic_ontology.md) 中的理论更新中英两个文件,并更新中英文两个文件的版本号为[核心版本号]
更新英文版时直接清空英文版原文然后把中文翻译过去
不要尝试生成脚本,因为有可能还要更新文件内容.

优先更新根目录根formal_theory*.

---
> Source: [loning/universe](https://github.com/loning/universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
