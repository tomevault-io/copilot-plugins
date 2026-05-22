## coding-standards

> Python代码标准和最佳实践


# Python代码标准和最佳实践

## 代码风格

- **格式化工具**: 使用 `black` 进行代码格式化，配置在 [pyproject.toml](mdc:pyproject.toml)
- **导入排序**: 使用 `isort` 进行导入语句排序
- **代码检查**: 使用 `flake8` 进行代码风格检查
- **类型检查**: 使用 `mypy` 进行静态类型检查

## 安全要求

- **API密钥**: 必须使用环境变量，通过 `os.getenv()` 获取，不得硬编码
- **安全扫描**: 使用 `bandit` 和 `safety` 进行安全漏洞扫描
- **敏感信息**: 不得在代码中暴露密码、密钥等敏感信息

## Django最佳实践

### 模型设计
- 模型文件按功能分类，位于 [apps/tools/models/](mdc:apps/tools/models/)
- 使用适当的字段类型和约束
- 添加 `__str__` 方法用于调试
- 使用 `Meta` 类定义表名和索引

### 视图设计
- 视图文件按功能分类，位于 [apps/tools/views/](mdc:apps/tools/views/)
- 使用类视图优先于函数视图
- 添加适当的权限检查
- 使用 `@login_required` 装饰器保护需要认证的视图

### 服务层
- 业务逻辑放在服务层，位于 [apps/tools/services/](mdc:apps/tools/services/)
- 服务类应该有清晰的职责划分
- 使用依赖注入模式

## 错误处理

- 使用适当的异常类型
- 添加日志记录
- 提供有意义的错误消息
- 使用 `try-except` 块处理可能的异常

## 测试要求

- 为所有新功能编写测试
- 测试文件位于 [tests/](mdc:tests/) 目录
- 使用 `pytest` 作为测试框架
- 保持测试覆盖率在合理水平

## 文档要求

- 为所有公共函数和类添加文档字符串
- 使用Google风格的文档字符串格式
- 添加类型注解
- 保持注释的及时更新

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
