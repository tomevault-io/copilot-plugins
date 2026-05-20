## theory-maintenance

> 理论体系自动维护

此规则用于自动维护整个理论体系的一致性，确保所有理论文件都符合格式要求并且保持同步。

执行以下维护任务：
1. 检查未索引文件并添加到索引（中英文）
2. 更新所有理论文件中的维度信息
3. 检查并修复中英文文档之间的链接关系
4. 更新版本号到最新的宇宙本论版本号
5. 重新排序索引文件中的理论列表（按维度从高到低）
6. 生成或更新依赖关系图

使用脚本：
- [check_unindexed_files.py](mdc:tools/check_unindexed_files.py) - 检查未索引文件
- [update_dimensions.py](mdc:tools/update_dimensions.py) - 更新维度信息
- [fix_critical_links.py](mdc:tools/fix_critical_links.py) - 修复中英文链接
- [reorder_theories.py](mdc:tools/reorder_theories.py) - 重排索引文件
- [generate_theory_graph.py](mdc:tools/generate_theory_graph.py) - 生成理论依赖图

维护周期：每次理论体系有重大更新时执行一次完整维护。

---
> Source: [loning/universe](https://github.com/loning/universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
