## task-config-md-prototype

> 所有代码、配置、脚本、文档和 Markdown 数据文件均使用 UTF-8 编码。

# Repository Guidelines

所有代码、配置、脚本、文档和 Markdown 数据文件均使用 UTF-8 编码。

## Markdown 清册编辑

修改 `任务项运行配置工具.md` 前，先阅读并遵循：

- `skills/task-config-md-editor/SKILL.md`

编辑后运行：

```bash
node skills/task-config-md-editor/scripts/validate-task-config-md.js 任务项运行配置工具.md
```

该 Markdown 是页面的数据源；删除清册行时必须同时删除对应的 `1.1/1.2` 二级参数行，并重新编号。

---
> Source: [xiaoguan521/task-config-md-prototype](https://github.com/xiaoguan521/task-config-md-prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
