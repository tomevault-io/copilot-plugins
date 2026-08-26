## matchaflow

> 多智能体项目管理模拟系统（Multi-Agent Project Management Simulation System）。基于 LLM 的多 Agent 系统，模拟完整软件项目管理流程：项目发起人、项目经理、项目组成员三个角色，经历预启动、启动、计划、执行、控制、结束六个阶段，产出项目章程、WBS、代码、挣值分析等交付物。

## 项目概述

多智能体项目管理模拟系统（Multi-Agent Project Management Simulation System）。基于 LLM 的多 Agent 系统，模拟完整软件项目管理流程：项目发起人、项目经理、项目组成员三个角色，经历预启动、启动、计划、执行、控制、结束六个阶段，产出项目章程、WBS、代码、挣值分析等交付物。

## 技术栈

- **语言**：Python 3.12
- **包管理**：uv（项目级虚拟环境）
- **依赖**：openai>=1.0.0, requests>=2.28.0, python-dateutil>=2.8.0
- **LLM 接口**：OpenAI 兼容 API（通过 config.py 配置 base_url / api_key / model）

## 目录结构

```
├── agents/              # Agent 模块（base_agent, sponsor, manager, team_member）
├── database/            # 共享数据库（shared_db.py）
├── workflow/            # 工作流引擎（engine.py）
├── utils/               # 工具模块（llm_client.py, document_generator.py）
├── tests/               # 测试套件（test_all.py，86 个测试）
├── simulation/          # 模拟结果存储目录
├── config.py            # 配置（LLM、阶段定义、Agent 提示词）
├── main.py              # 主程序入口
├── test_simple.py       # 快速冒烟测试
├── requirements.txt     # 依赖声明
├── README.md            # 英文文档（主入口）
├── README.zh-CN.md      # 中文文档
└── TEST_REPORT.md       # 完整测试报告
```

## 关键入口 / 核心模块

- **main.py**：CLI 入口，支持 `--project-idea` 和 `--project-code` 参数
- **workflow/engine.py**：工作流引擎，驱动六阶段流转
- **agents/**：三个角色 Agent 的实现
- **config.py**：LLM 配置与 Agent 提示词模板（需配置真实 API 信息后方可运行）

## 运行与预览

- 本项目为 CLI 工具，非预览型产物，`preview_enable = "disabled"`
- 运行方式：`python main.py --project-idea "项目描述"`
- 需先在 config.py 中配置有效的 LLM API 信息

## 用户偏好与长期约束

- 使用 uv 管理 Python 虚拟环境和依赖
- 禁止使用 npm/yarn

## 常见问题和预防

- config.py 中 LLM_CONFIG 默认为占位值，运行前需替换为真实 API 配置
- MAX_EXECUTION_CYCLES 建议 ≥5，前两次循环为 review-only 模式

---
> Source: [NoahIsARider/MatchaFlow](https://github.com/NoahIsARider/MatchaFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
