## tespat-engine

> TESPAT Engine 是一个基于 Rust 的二维网格模式匹配与替换引擎，用于规则驱动的网格演化、自动机场景模拟和随机内容生成；仓库同时包含规则可视化编辑/回放前端，以及将规则编译为 Rust 代码的编译器，便于按功能定位实现。

# AGENTS.md

## 项目简介

TESPAT Engine 是一个基于 Rust 的二维网格模式匹配与替换引擎，用于规则驱动的网格演化、自动机场景模拟和随机内容生成；仓库同时包含规则可视化编辑/回放前端，以及将规则编译为 Rust 代码的编译器，便于按功能定位实现。

## 项目大结构

代码根目录下的大结构主要分为三部分：
- 前端：`tespat-web`
- 模式替换核心引擎：`tespat-core`
- 编译到 Rust 的 tespat 编译器：`tespat-build`

按功能查找代码时，可优先按职责定位：
- 编辑规则、导入导出、回放与界面交互：看 `tespat-web`
- 模式匹配、替换执行、对称变换、历史帧记录：看 `tespat-core`
- JSON 规则转 Rust 静态代码、编译期生成：看 `tespat-build`

## 提醒事项

- 搜索代码目录时，记得避开 `<workspace_root>/tespat-web/node_modules`、`<workspace_root>/tespat-web/dist` 和 `<workspace_root>/target` 目录。
- 若无特定要求，不要添加测试代码
- 若无特定要求：生成代码时模仿附近代码风格格式化代码，不要格式化你不需要修改的代码
- 生成代码时，如果需要生成新的块级定义（例如函数、接口定义、结构体定义等），给这个新生成的item上方（而不是内容）添加简短注释

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fancyflame)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fancyflame)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
