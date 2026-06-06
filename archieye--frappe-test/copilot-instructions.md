## frappe-test

> 本文档为 Claude Code（claude.ai/code）在此仓库中工作时提供指引。

# CLAUDE.md

本文档为 Claude Code（claude.ai/code）在此仓库中工作时提供指引。

## 仓库当前状态

该仓库目前是一个初始化工作区，尚未包含业务源码，也没有项目级的构建/测试配置。

当前根目录可见内容：
- `.venv/`：Python 虚拟环境
- `.idea/`：IDE 元数据

当前未发现以下文件：`README.md`、`.cursorrules`、`.cursor/rules/`、`.github/copilot-instructions.md`。

## 开发命令（当前可用）

由于尚无项目清单或构建配置，当前仅能确认以下环境级 Python 命令可用：

- 查看虚拟环境 Python 版本：
  - `./.venv/Scripts/python.exe --version`
- 查看已安装依赖：
  - `./.venv/Scripts/python.exe -m pip list`

## 构建、Lint 与测试命令

当前仓库尚未定义（未发现项目清单文件与相关工具配置）。

当后续加入项目文件后，请补充以下命令：
- 构建命令
- Lint 命令
- 全量测试命令
- 单测（单个测试）运行命令

## 高层架构

当前尚无应用模块或服务代码，因此暂未形成可描述的代码架构。

当后续加入源码后，请在本文件补充：
- 主要运行时与入口
- 关键模块/包边界
- 后端/前端/服务间数据流（如适用）
- 测试目录组织方式

---
> Source: [archieYe/frappe-test](https://github.com/archieYe/frappe-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
