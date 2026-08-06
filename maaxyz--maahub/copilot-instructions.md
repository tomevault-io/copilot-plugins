## maahub

> 以下的 markdown 是为修改此仓库的 AI 看的，如果你正在打算修改 `website`，请务必遵守。

以下的 markdown 是为修改此仓库的 AI 看的，如果你正在打算修改 `website`，请务必遵守。

```md
MaaHub 是 为 MaaFramework(maafw) 开源框架构建的范式参考社区，有关 `pipeline`、`custom` 等为 MaaFramework 专属概念，详情参考 `\dev\instructions\maafw-guide`。

简要而言，各模块指：

- `skills`：用于辅助 AI 理解 maafw 的 agent skill，并非 maafw 的概念
- `pipeline`：maafw 的流水线管线配置文件，一般为 json 或 jsonc 文件，具体规则参考`\dev\instructions\maafw-guide\3.1-任务流水线协议.md`
- `custom`：maafw 的自定义代码模块，一般用于自定义 pipeline 无法达成的功能，支持多语言
- `经验`：maafw 社区开发者撰写的使用框架开发自己项目的经验心得
```

以下的目录是为检索此仓库的 AI 看的。如果你打算搜索与下载某个 skill，或是参考某个 pipeline 等，可参考以下目录：

```tree
.
|-- Storage/
|   |-- customs/        # python、go 等自定义代码
|   |-- experiences/    # 开发经验
|   |-- pipelines/      # pipeline 范式
|   `-- skills/         # 用于 AI 的 skills
```

---
> Source: [MaaXYZ/MaaHub](https://github.com/MaaXYZ/MaaHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
