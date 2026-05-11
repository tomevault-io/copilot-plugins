## project-overview

> Gaia RPA 是一个基于"录制-执行-自愈"三阶段架构的Browser-AI RPA Starter项目。

# Gaia RPA 项目概览

## 项目简介
Gaia RPA 是一个基于"录制-执行-自愈"三阶段架构的Browser-AI RPA Starter项目。

## 核心架构
- **录制阶段**: 使用 [Browser-Use](mdc:https:/github.com/browser-use/browser-use) 进行可视化录制
- **执行阶段**: 使用 [Playwright](mdc:https:/playwright.dev) 进行高并发稳定执行
- **自愈阶段**: 使用 Browser-Use 进行智能错误修复

## 项目结构
```
gaia-rpa/
├── .venv/                   # python虚拟环境
├── docs/                    # 📖 项目文档
│   ├── development-plan.md  # 开发规划文档
│   ├── phase-0-bootstrap-plan.md  # 阶段0计划
│   └── todolist.md         # 任务清单
├── src/                    # 💻 源代码目录
│   ├── cli/               # 命令行接口
│   ├── core/              # 核心功能模块
│   ├── models/            # 数据模型定义
│   ├── utils/             # 工具函数集
│   └── web/               # Web界面
├── workflows/             # 🎬 工作流存储
├── contexts/              # 🔧 配置上下文
├── scripts/               # 🔧 辅助脚本
└── tests/                 # 🧪 测试文件
```

## 技术栈
- **后端**: Python 3.11+, FastAPI, Pydantic
- **前端**: HTML5, Bootstrap 5, Vanilla JavaScript
- **自动化**: Browser-Use, Playwright
- **CLI**: Typer
- **日志**: structlog
- **模板**: Jinja2

## 开发阶段
当前项目按照以下阶段进行开发：
- ✅ **Phase 0**: 项目初始化和环境搭建
- ✅ **Phase 1**: 录制和编辑功能实现
- 🔄 **Phase 2**: 执行功能实现 (进行中)
- ⏳ **Phase 3**: Browser-Use自愈功能实现
- ⏳ **Phase 4**: CLI工具和用户界面完善
- ⏳ **Phase 5**: 测试完善和文档编写

## 关键文件
- [pyproject.toml](mdc:pyproject.toml) - 项目配置
- [requirements.txt](mdc:requirements.txt) - 依赖管理
- [src/cli/main.py](mdc:src/cli/main.py) - CLI入口
- [src/web/app.py](mdc:src/web/app.py) - Web应用入口
- [scripts/start_web_ui.py](mdc:scripts/start_web_ui.py) - Web UI启动脚本

---
> Source: [WW-AI-Lab/browser-use-playwright](https://github.com/WW-AI-Lab/browser-use-playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
