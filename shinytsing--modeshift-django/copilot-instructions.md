## project-structure

> Django项目结构和架构指南


# Django项目结构和架构指南

这是一个复杂的Django项目，名为"modeshift_django"，包含多个应用模块和完整的CI/CD部署流程。

## 项目根目录结构

- **主入口**: [manage.py](mdc:manage.py) - Django管理命令入口
- **配置目录**: [config/](mdc:config/) - 包含不同环境的Django设置
- **应用目录**: [apps/](mdc:apps/) - 包含所有Django应用
- **模板目录**: [templates/](mdc:templates/) - HTML模板文件
- **静态文件**: [static/](mdc:static/) 和 [src/static/](mdc:src/static/) - CSS、JS、图片等静态资源
- **媒体文件**: [media/](mdc:media/) - 用户上传的文件
- **测试目录**: [tests/](mdc:tests/) - 单元测试和集成测试

## 核心应用模块

### 1. 工具应用 (apps/tools/)
- **模型**: [apps/tools/models/](mdc:apps/tools/models/) - 包含多个模型文件，按功能分类
- **视图**: [apps/tools/views/](mdc:apps/tools/views/) - 各种工具功能的视图
- **服务**: [apps/tools/services/](mdc:apps/tools/services/) - 业务逻辑服务层
- **管理命令**: [apps/tools/management/commands/](mdc:apps/tools/management/commands/) - Django管理命令

### 2. 用户应用 (apps/users/)
- **用户管理**: 用户认证、权限管理
- **个人资料**: 用户信息、头像等

### 3. 内容应用 (apps/content/)
- **内容管理**: 文章、页面等内容管理

### 4. 分享应用 (apps/share/, apps/sharing/, apps/social_sharing/)
- **社交分享**: 各种分享功能

## 配置管理

- **开发环境**: [config/settings/development.py](mdc:config/settings/development.py)
- **生产环境**: [config/settings/production.py](mdc:config/settings/production.py)
- **基础配置**: [config/settings/base.py](mdc:config/settings/base.py)

## 部署和CI/CD

- **Docker配置**: [docker-compose.yml](mdc:docker-compose.yml), [Dockerfile](mdc:Dockerfile)
- **CI/CD工作流**: [.github/workflows/ci-cd.yml](mdc:.github/workflows/ci-cd.yml)
- **部署脚本**: [deploy.sh](mdc:deploy.sh), [deploy-ci.sh](mdc:deploy-ci.sh)
- **环境变量**: [env.production](mdc:env.production), [env.example](mdc:env.example)

## 数据库

项目使用PostgreSQL作为主数据库，配置在Django设置中。开发和生产环境都使用PostgreSQL，确保一致性。

## 重要注意事项

1. **API密钥管理**: 所有第三方API密钥都通过环境变量管理，不得硬编码
2. **代码质量**: 项目有完整的代码质量检查工具链（flake8, black, isort, mypy, bandit, safety）
3. **测试覆盖**: 包含单元测试、集成测试和端到端测试
4. **安全扫描**: 使用bandit和safety进行安全漏洞扫描
5. **部署流程**: 有完整的Docker化部署流程和CI/CD自动化

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
