## openmanus

> - Google Style docstrings


## 项目特定规则


说中文


### 代码风格
- 使用 Python 3.13+ 特性
- 使用 uv 管理环境
- 单个文件不超过 1000 行
- 使用完整的类型提示
- Google Style docstrings
- 通过 ruff 和 mypy 检查


### 组件创建规范
创建任何新组件（app/script/package）时必须包含：
1. README.md - 完整的使用文档
2. .ai.json - AI 可解析的元数据（符合 .ai/schemas/component.schema.json）
3. examples/ - 至少一个可运行的示例
4. 完整的 Google Style docstrings
5. 类型提示


### AI Agent 友好设计
- 所有 CLI 使用 Typer 框架
- 输入输出优先使用 JSON/YAML
- 配置使用 Pydantic Settings
- 接口标准化且可预测
- 提供清晰的错误消息


### 文档要求
- 每个函数和类都要有 docstring
- 包含 Args, Returns, Raises, Examples
- README 包含安装、使用、配置、示例
- .ai.json 包含 interface、examples、ai_hints


### 工具使用
- 格式化: `ruff format`
- Lint: `ruff check --fix`
- 类型检查: `mypy`
- 测试: `pytest`


### Git 提交规范
- feat: 新功能
- fix: 修复 bug
- docs: 文档更新
- refactor: 重构
- test: 测试
- chore: 构建/工具更新


### 开发流程
1. 小步快跑，步步验证
2. 经常使用 git 保存进度
3. 创建组件后立即更新 AI 上下文: `python scripts/tools/update_ai_context.py`
4. 使用 pre-commit hooks 确保代码质量


### AI 提示模板
使用 .ai/prompts/ 中的模板来指导 AI：
- create_component.md - 创建新组件
- code_review.md - 代码审查


### 工程化思维

- 不要做玩具，使用 SOTA 模式
- 考虑可维护性和可扩展性
- 遵循 SOLID 原则
- 优先使用已有的 packages/common 组件
- 避免许多冗余的说明文件，如果确实需要多个说明文件，请你在 docs 文件夹内新建文件夹来装载对应的说明文件


### 其他

- 测试覆盖率应该大于 80%

### 文件元信息协议（File Metadata Protocol）


所有源代码文件必须在文件最前方包含元信息注释块，用于描述文件的核心信息。


#### 强制字段
- `@PURPOSE:` 文件的核心作用和功能（必填）
- `@OUTLINE:` 文件的结构大纲，包括主要类/函数/模块（必填）


#### 可选字段
- `@GOTCHAS:` 易出错点、注意事项、常见陷阱
- `@TECH_DEBT:` 已知的技术债务和待优化项
- `@DEPENDENCIES:` 关键依赖关系（内部/外部模块）
- `@CHANGELOG:` 重要修改历史记录
- `@AUTHOR:` 作者信息
- `@RELATED:` 相关文件引用


#### 格式规范


**Python 文件示例：**
```python
"""
@PURPOSE: 实现用户认证和授权功能
@OUTLINE:
  - class AuthService: 主认证服务类
  - def login(username, password): 用户登录
  - def verify_token(token): 验证JWT令牌
@GOTCHAS:
  - 密码必须在存储前进行哈希处理
  - Token过期时间为24小时，需要定期刷新
@TECH_DEBT:
  - TODO: 添加多因素认证支持
  - TODO: 实现OAuth2.0集成
@DEPENDENCIES:
  - 内部: packages.common.logger, packages.common.config
  - 外部: jwt, bcrypt
@RELATED: user_service.py, permission_manager.py
"""
```


**TypeScript/JavaScript 文件示例：**
```typescript
/**
 * @PURPOSE: 实现前端路由管理和导航守卫
 * @OUTLINE:
 *   - class Router: 核心路由管理器
 *   - function setupGuards(): 配置导航守卫
 *   - function handleNavigation(): 处理路由跳转
 * @GOTCHAS:
 *   - 路由守卫必须返回boolean或Promise<boolean>
 *   - 避免在守卫中进行重定向循环
 * @DEPENDENCIES:
 *   - 外部: vue-router, pinia
 */
```


#### 规则说明


1. **位置要求**：元信息注释必须位于文件最开始（在任何import语句之前）
2. **格式要求**：
   - 使用对应语言的多行注释语法
   - 每个字段以 `@KEYWORD:` 开头（关键字后跟冒号）
   - 多行内容使用缩进延续
3. **内容要求**：
   - @PURPOSE 应简洁明了，1-2句话说明核心功能
   - @OUTLINE 列出主要的类、函数、导出内容
   - 其他字段根据实际情况填写，不强制
4. **维护要求**：
   - 文件结构发生重大变化时必须更新 @OUTLINE
   - 添加重要功能时更新 @CHANGELOG
   - 发现新的易错点时补充到 @GOTCHAS


**注意：**
- 所有源代码文件都已符合元信息协议规范（合规率100%）
- 优先使用MCP工具获取元信息，比读取整个文件更高效
- 如果需要查看具体实现代码，再使用 read_file 工具


- 执行优化、编写、重构等代码工作时，徐娅仔细思考是否回产生其他bug或问题。


- 你需要有全局思考能力，思考、解决问题的时候尽量考虑到全局范围，不要只盯着用户指定的问题，需要思考该问题会不会牵扯到其他问题，优化该问题时需要思考有没有其他有关联性的代码或者问题需要同步优化、解决。


- 使用.env文件加载变量


- 我不许单个代码文件


- 超过1000 行小步快跑，布布验证


- 不要频繁建立 md 文件报告总结进度如果确实需要，请你写在项目对应的文件中，而不是新建文件


- 时刻记住使用 git 保存进度


- 如果遇到类似于.env的文件使用 shell 的 cat 命令查看


- 使用 SOTA 模式，不要做玩具（mvp 版本），使用工程化思维


- Always respond in Chinese-simplified

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GouBuliya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GouBuliya)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
