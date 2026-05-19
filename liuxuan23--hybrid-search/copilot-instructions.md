## hybrid-search

> - 优先做最小改动，不要顺手重构无关代码。

# AGENTS.md

## Coding Preferences

- 回复和代码说明使用中文。
- 优先做最小改动，不要顺手重构无关代码。
- 修改逻辑时，尽量保持现有接口不变。
- Python 代码优先保证可读性，少做过度抽象。
- 函数应尽量短小，单个函数只做一件事。
- 注释要少而准，只解释“为什么”，不要解释显而易见的“做了什么”。
- 错误处理要显式，不要静默吞异常。
- 新增功能或修复 bug 时，优先补充对应测试。
- 除非我明确要求，否则不要大规模格式化文件。
- 不要修改与当前任务无关的文件。
- 注释信息要精准且详细。

## When Making Changes

- 先阅读相关测试和调用链，再下手修改。
- 如果存在多种实现方案，优先选择改动最小、风险最低的方案。
- 如果需要做有明显取舍的重构，先暂停并说明方案。

---
> Source: [liuxuan23/hybrid_search](https://github.com/liuxuan23/hybrid_search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
