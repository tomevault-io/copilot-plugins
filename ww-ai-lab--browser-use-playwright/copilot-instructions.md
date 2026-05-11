## development-workflow

> │   ├── feature/phase-2-executor (功能分支)

# 开发工作流和最佳实践

## 开发流程

### 分支管理策略
```
main (生产分支)
├── develop (开发分支) 
│   ├── feature/phase-2-executor (功能分支)
│   ├── feature/phase-3-healer (功能分支)
│   └── hotfix/bug-fix (热修复分支)
```

### 提交规范
使用 [Conventional Commits](mdc:https:/www.conventionalcommits.org) 规范：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**类型说明**:
- `feat`: 新功能
- `fix`: 修复bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构代码
- `test`: 添加或修改测试
- `chore`: 构建过程或辅助工具的变动

**示例**:
```bash
feat(recorder): 添加Browser-Use工作流录制功能
fix(web): 修复模板文件缺失问题
docs(readme): 更新安装说明
```

### 开发阶段管理
参考 [docs/todolist.md](mdc:docs/todolist.md) 和 [docs/development-plan.md](mdc:docs/development-plan.md)：

1. **Phase 0**: ✅ 项目初始化和环境搭建
2. **Phase 1**: ✅ 录制和编辑功能实现
3. **Phase 2**: 🔄 执行功能实现 (当前阶段)
4. **Phase 3**: ⏳ Browser-Use自愈功能实现
5. **Phase 4**: ⏳ CLI工具和用户界面完善
6. **Phase 5**: ⏳ 测试完善和文档编写

## 环境管理

### 虚拟环境设置
```bash
# 创建虚拟环境
python -m venv .venv

# 激活虚拟环境 (macOS/Linux)
source .venv/bin/activate

# 激活虚拟环境 (Windows)
.venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
```

### 依赖管理
- 使用 [requirements.txt](mdc:requirements.txt) 管理依赖
- 新增依赖时同时更新 requirements.txt
- 定期更新依赖版本，确保安全性

### 配置管理
- 使用 [config.yml](mdc:config.yml) 管理应用配置
- 敏感信息通过环境变量管理
- 不同环境使用不同的配置文件

## 代码质量保证

### 代码检查工具
```bash
# 代码格式化
black src/ tests/

# 导入排序
isort src/ tests/

# 代码检查
flake8 src/ tests/

# 类型检查
mypy src/
```

### 预提交钩子
创建 `.pre-commit-config.yaml`：
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 22.3.0
    hooks:
      - id: black
  - repo: https://github.com/pycqa/isort
    rev: 5.10.1
    hooks:
      - id: isort
  - repo: https://github.com/pycqa/flake8
    rev: 4.0.1
    hooks:
      - id: flake8
```

### 测试策略
```bash
# 运行所有测试
pytest

# 运行特定测试模块
pytest tests/test_recorder.py

# 运行覆盖率测试
pytest --cov=src tests/

# 运行异步测试
pytest -v tests/test_async_functions.py
```

## 文档管理

### 文档结构
```
docs/
├── development-plan.md      # 开发规划
├── phase-0-bootstrap-plan.md # 阶段0计划
├── todolist.md             # 任务清单
├── api-reference.md         # API参考 (待创建)
├── user-guide.md           # 用户指南 (待创建)
└── deployment.md           # 部署指南 (待创建)
```

### 文档更新原则
1. 代码变更时同步更新相关文档
2. API变更时更新API文档
3. 新功能完成时更新用户指南
4. 每个阶段完成时更新开发规划

### 代码注释规范
```python
class WorkflowRecorder:
    """工作流录制器
    
    负责使用Browser-Use进行工作流录制，生成JSON格式的工作流文件。
    支持交互式录制和引导式录制两种模式。
    
    Attributes:
        output_dir: 工作流输出目录
        browser_config: 浏览器配置
        
    Example:
        >>> recorder = WorkflowRecorder()
        >>> workflow = await recorder.record_workflow("my_workflow")
    """
    
    async def record_workflow(self, name: str, mode: str = "interactive") -> Workflow:
        """录制工作流
        
        Args:
            name: 工作流名称
            mode: 录制模式，'interactive' 或 'guided'
            
        Returns:
            Workflow: 录制完成的工作流对象
            
        Raises:
            RecordingError: 录制过程中发生错误
        """
        pass
```

## 调试和日志

### 日志配置
使用 [src/utils/logger.py](mdc:src/utils/logger.py) 进行结构化日志：

```python
import structlog

logger = structlog.get_logger(__name__)

# 不同级别的日志
logger.debug("调试信息", step_id=step.id)
logger.info("操作成功", workflow_name=name)
logger.warning("警告信息", message="某些条件不满足")
logger.error("错误信息", error=str(e), traceback=True)
```

### 调试技巧
```python
# 使用断点调试
import pdb; pdb.set_trace()

# 异步代码调试
import asyncio
import pdb

async def debug_async_function():
    pdb.set_trace()
    result = await some_async_operation()
    return result
```

## 部署和发布

### 本地开发服务器
```bash
# 启动Web UI
python scripts/start_web_ui.py

# 使用CLI工具
python -m src.cli.main --help

# 运行测试
python scripts/test_web_ui.py
```

### 构建和打包
```bash
# 构建Python包
python -m build

# 创建Docker镜像 (待实现)
docker build -t gaia-rpa:latest .

# 运行Docker容器
docker run -p 8000:8000 gaia-rpa:latest
```

### 版本管理
使用语义化版本控制 (SemVer)：
- `MAJOR.MINOR.PATCH`
- 例如: `1.0.0`, `1.1.0`, `1.1.1`

## 协作开发

### 代码审查清单
- [ ] 代码符合项目编码规范
- [ ] 包含适当的错误处理
- [ ] 添加了必要的测试
- [ ] 更新了相关文档
- [ ] 日志记录完整
- [ ] 性能考虑合理

### 问题报告模板
```markdown
## 问题描述
简要描述遇到的问题

## 复现步骤
1. 执行命令 `xxx`
2. 访问页面 `xxx`
3. 点击按钮 `xxx`

## 预期行为
描述期望的正确行为

## 实际行为
描述实际发生的错误行为

## 环境信息
- OS: macOS 12.0
- Python: 3.11.0
- Browser: Chrome 96.0

## 错误日志
```
粘贴相关的错误日志
```

### 功能请求模板
```markdown
## 功能描述
描述需要的新功能

## 使用场景
说明这个功能的使用场景

## 建议实现
提供实现建议（可选）

## 相关文档
链接到相关的文档或issue
```

---
> Source: [WW-AI-Lab/browser-use-playwright](https://github.com/WW-AI-Lab/browser-use-playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
