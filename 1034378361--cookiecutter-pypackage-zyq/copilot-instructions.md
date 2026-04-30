## cookiecutter-pypackage-zyq

> description: Expert guidance for data analysis workflows using pandas, matplotlib, seaborn, and numpy in Jupyter notebooks.

# 规则文件配置信息

## 数据分析规则
```yaml
---
description: Expert guidance for data analysis workflows using pandas, matplotlib, seaborn, and numpy in Jupyter notebooks.
globs: ["**/*.py", "**/*.ipynb"]
alwaysApply: false
---
```

## 依赖管理规则
```yaml
---
description: 项目依赖管理策略和最佳实践，基于pyproject.toml配置。
globs: ["pyproject.toml", "requirements*.txt"]
alwaysApply: true
---
```

## 测试规则
```yaml
---
description: 项目测试策略和实践，基于pytest框架和项目配置。
globs: ["tests/**/*.*", "pyproject.toml"]
alwaysApply: true
---
```

## 项目结构规则
```yaml
---
description: 项目结构和工作流规范，基于当前模板的目录结构、Makefile命令和开发流程。
globs: ["**/*.*"]
alwaysApply: true
---
```

## 代码标准规则
```yaml
---
description: 综合的Python代码标准，包含编码风格、命名、格式和质量标准，基于项目配置的工具链。
globs: ["**/*.py"]
alwaysApply: true
---
```

## 文档规则
```yaml
---
description: Python项目文档最佳实践，包括代码文档、项目文档和API文档的标准和示例。
globs: ["**/*.py", "**/*.rst", "docs/**/*.*"]
alwaysApply: true
---
```

## 技术实践规则
```yaml
---
description: 定义Python项目中的技术最佳实践，包括错误处理、日志记录和配置管理，确保代码的健壮性和可维护性。
globs: ["**/*.py"]
alwaysApply: true
---
```

## CI/CD规则
```yaml
---
description: 项目CI/CD流程实施指南，基于项目中配置的GitHub Actions工作流。
globs: [".github/workflows/**/*.*", "Makefile"]
alwaysApply: true
---
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1034378361) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
