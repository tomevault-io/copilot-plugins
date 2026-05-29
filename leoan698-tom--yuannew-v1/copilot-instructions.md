## yuannew-v1

> - 默认使用中文回复、分析、写文档和提交说明。

# AGENTS.md

- 默认使用中文回复、分析、写文档和提交说明。
- 当前仓库是元聚邻 GitHub 执行仓的本地准备目录。
- 先读 `操作系统/00_总入口.md`。
- 完整工作流以 `操作系统/07_完整工作流/README.md` 为准。
- 采用页面级闭环：测完一个页面，处理确认改动，修复、验证、提交后，再测下一个页面。
- 不要先测完整个企业端再集中修复。
- 同一页面内两个测试 skill 必须先后执行：先 `project-issue-pattern-audit`，再 `yuanjulin-prototype-issue-detectors`。
- 没有进入 `确认改动/` 的问题，不允许修。
- 一个问题一个 DEV 修复闭环，不允许顺手修其他问题。
- 每个问题必须完成 `01_问题证据.md`、`02_产品裁决.md`、`03_改动方案.md`、`04_开发记录.md`、`05_文档同步.md`、`06_验证证据.md`、`07_提交记录.md`。
- `YuanJuLin_New/` 在 GitHub 执行仓中作为普通目录处理；上传副本不得保留内部 `.git`。
- 新写入路径优先使用相对路径或 `${PROJECT_ROOT}`，不要新增依赖 `D:\WGKJ\yuanxing` 的硬编码。

---
> Source: [leoan698-tom/yuannew_v1](https://github.com/leoan698-tom/yuannew_v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
