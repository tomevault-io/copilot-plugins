## py-tools

> - 仓库定位：Py-Tools 是一个实用的 Python 工具集和可复用组件库，旨在简化常见任务，提高 Python 项目的开发效率。

# AGENTS.md

- 仓库定位：Py-Tools 是一个实用的 Python 工具集和可复用组件库，旨在简化常见任务，提高 Python 项目的开发效率。
- 面向对象：主要面向 Python 开发人员；编写代码、设计 API、补充示例和文档时，优先考虑开发者使用体验、接入成本、可复用性与一致性。
- 仓库类型：Python 工具库；源码在 `py_tools/`，发布名 `huidevkit`，使用 `setup.py` 打包，Python `>=3.9`。
- 主要目录：`tests/` 是测试，`demo/` 是示例，`setup.py` 管依赖/extras/CLI，`py_tools/utils/project_templates/` 是脚手架模板，非明确需求不要改。

## 定位约束

- 默认优先服务“开发提效”和“通用复用”场景，避免引入只适配单一业务、单一项目的实现。
- 设计接口、默认参数、目录结构和示例时，优先保证易理解、低心智负担、低接入成本，不为局部灵活性牺牲整体一致性。
- 新增能力或调整行为时，优先沿用现有模式，保持 API 稳定，避免让开发者迁移成本过高。
- 涉及公共导出、默认行为、脚手架模板或示例代码时，除了“能否工作”，还要判断是否有助于开发者快速上手和正确使用。

## `py_tools/` 模块地图

- `connections/`：连接类能力。
- `connections/http/`：同步/异步 HTTP 客户端。
- `connections/db/mysql/`：SQLAlchemy ORM 封装。
- `connections/db/redis_client.py`：Redis 客户端。
- `connections/oss/`：Minio 客户端等。
- `connections/mq/`：RabbitMQ / Kafka 客户端，当前偏实验性，改动前先确认影响。
- `utils/`：通用工具；包含时间、文件、正则、JWT、树结构、序列化、异步互转、Excel、脱敏等。
- `decorators/`：装饰器能力；含缓存、重试/超时等基础装饰器。
- `logging/`：loguru 日志封装与默认配置。
- `chatbot/`：聊天机器人发送与封装。
- `enums/`：公共枚举。
- `exceptions/`：公共异常。
- `data_schemas/`：公共数据结构。
- `meta_cls/`：元类工具。
- `constants/`：常量定义。
- `__init__.py`：公共导出；改这里要特别注意兼容性。

## 开发提示

- 常用命令：`pip install -e .`、`pip install -e .[test]`、`pytest`、`ruff check .`
- 可选依赖按需安装：`.[db-orm]`、`.[db-redis]`、`.[excel-tools]`、`.[all]`
- 改公共 API、默认参数、返回结构、导出路径时，优先考虑兼容性。
- 改用法相关内容时，同步检查 `README.md`、`demo/`、`tests/`。
- 改依赖、extras、版本、包信息、CLI 入口时，直接检查 `setup.py`。

## 文档约束

- `README.md`、`demo/README.md` 和示例文档只写面向仓库读者的稳定信息，不写当前对话、协作过程、修改原因或过程性表述。
- 不要把用户与智能体的对话原句、临时决策或提示词要求直接写入对外文档。
- 文档默认面向未来读者，避免使用“这次改成”“按刚才讨论”“我们认为”这类对话式措辞。

## 注释与 Docstring 约定

- 默认使用 Google 风格 docstring。
- 注释与 docstring 内容默认使用中文，除非外部协议或专有名词更适合保留英文。
- 公共类、公共方法、工具函数、跨模块复用能力，优先补充 docstring。
- docstring 重点说明职责、参数、返回值、副作用和使用边界，不要重复代码字面含义。
- 行内注释只在解释设计原因、约束条件或容易误用的行为时添加。
- 私有且语义清晰的简单函数，可以不写 docstring。

---
> Source: [HuiDBK/py-tools](https://github.com/HuiDBK/py-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
