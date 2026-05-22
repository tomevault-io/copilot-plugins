## testing-standards

> 测试标准和最佳实践


# 测试标准和最佳实践

## 测试目录结构

- **主测试目录**: [tests/](mdc:tests/) - 包含所有测试文件
- **单元测试**: [tests/unit/](mdc:tests/unit/) - 单元测试
- **集成测试**: [tests/integration/](mdc:tests/integration/) - 集成测试
- **端到端测试**: [tests/e2e/](mdc:tests/e2e/) - 端到端测试
- **性能测试**: [tests/performance/](mdc:tests/performance/) - 性能测试

## 测试框架配置

- **主框架**: pytest
- **Django集成**: pytest-django
- **覆盖率**: pytest-cov
- **并行执行**: pytest-xdist
- **HTML报告**: pytest-html
- **配置文件**: [pytest.ini](mdc:pytest.ini)

## 测试类型

### 1. 单元测试
- 测试单个函数或方法
- 使用mock隔离外部依赖
- 快速执行，无外部依赖

### 2. 集成测试
- 测试多个组件协作
- 使用真实数据库和Redis
- 验证数据流和API调用

### 3. 端到端测试
- 测试完整用户流程
- 使用Selenium或类似工具
- 验证前端和后端集成

### 4. 性能测试
- 测试响应时间和吞吐量
- 内存使用和CPU使用率
- 数据库查询性能

## 测试数据管理

### 测试数据库
- 使用PostgreSQL（与生产环境一致）
- 测试前清理数据
- 使用事务回滚

### 测试数据创建
```python
# 使用工厂模式创建测试数据
from django.test import TestCase
from apps.tools.models import SomeModel

class TestSomeFeature(TestCase):
    def setUp(self):
        self.test_data = SomeModel.objects.create(
            field1='test_value',
            field2='another_value'
        )
```

## 测试覆盖率

- **目标覆盖率**: 80%以上
- **关键路径**: 100%覆盖
- **报告生成**: 使用pytest-cov生成HTML报告
- **覆盖率文件**: [coverage.xml](mdc:coverage.xml)

## 测试命令

### 运行所有测试
```bash
pytest
```

### 运行特定测试
```bash
# 运行单元测试
pytest tests/unit/

# 运行集成测试
pytest tests/integration/

# 运行特定测试文件
pytest tests/unit/test_models.py
```

### 生成覆盖率报告
```bash
pytest --cov=apps --cov-report=html --cov-report=xml
```

## 测试最佳实践

### 1. 测试命名
- 使用描述性的测试方法名
- 遵循 `test_<功能>_<场景>_<期望结果>` 格式

### 2. 测试结构
- 使用AAA模式：Arrange, Act, Assert
- 每个测试只验证一个行为
- 保持测试独立和可重复

### 3. 测试数据
- 使用工厂模式创建测试数据
- 避免硬编码测试数据
- 清理测试后的数据

### 4. 断言
- 使用具体的断言消息
- 验证所有相关状态
- 测试边界条件

### 5. 模拟和存根
- 模拟外部API调用
- 使用Django的测试客户端
- 模拟文件上传和下载

## 持续集成测试

### GitHub Actions集成
- 每次推送都运行测试
- 测试失败阻止部署
- 生成测试报告和覆盖率

### 本地测试流程
1. 运行代码质量检查
2. 运行单元测试
3. 运行集成测试
4. 检查测试覆盖率
5. 修复失败的测试

## 测试维护

### 定期审查
- 删除过时的测试
- 更新测试数据
- 优化测试性能

### 测试文档
- 为复杂测试添加注释
- 维护测试用例文档
- 记录测试环境要求

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
