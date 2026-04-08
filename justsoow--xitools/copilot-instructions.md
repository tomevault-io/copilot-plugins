## xitools

> 本文件定义了 Antigravity AI 助手在 XItools 项目中的工作规则和规范。

# XItools 项目 - Antigravity AI 助手规则

本文件定义了 Antigravity AI 助手在 XItools 项目中的工作规则和规范。

## 基础要求

### 语言规范
- **始终使用中文**来编写文档、提交信息和回复用户
- **Walkthrough使用中文来写**
- 对于对话过程中的工作流实现方案和计划方案文档都使用中文来写
- 代码注释优先使用中文,除非是国际通用的技术术语
- 当前的i18n组件已经停止使用英文翻译工作，目前只支持中文，暂不考虑英文支持  

### 项目理解
- 这是一个团队协作工具项目,包含前端(React + TypeScript)和后端(Python)
- 前端使用 React + TypeScript + Vite 构建
- 后端使用 Python FastAPI 框架
- 使用 Docker 进行容器化部署



## 必须要遵守的 Git 规则

- **严禁**直接在 `develop` 和 `main` 分支上进行 `git push` 操作。
- 所有 `feature/*` 分支的修改**必须**通过创建拉取请求（Pull Request）合并到 `develop` 分支。
- 所有 `develop` 分支的修改**必须**通过创建拉取请求（Pull Request）合并到 `main` 分支。

## Git 提交规范

所有提交信息必须遵循以下格式：

### Commit Message 格式
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Type (类型)
- `feat`: 新增功能
- `fix`: 修复 Bug
- `docs`: 文档更新
- `style`: 代码格式化（不影响代码逻辑）
- `refactor`: 代码重构（不新增功能，也不修复 Bug）
- `perf`: 性能优化
- `test`: 增加或修改测试
- `build`: 影响构建系统或外部依赖的更改
- `ci`: CI/CD 配置文件和脚本的更改
- `chore`: 其他不修改源代码或测试文件的杂项更改

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustSOOw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JustSOOw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
