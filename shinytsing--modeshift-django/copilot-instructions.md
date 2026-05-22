## cicd-deployment

> CI/CD和部署流程规范


# CI/CD和部署流程规范

## GitHub Actions工作流

主工作流文件：[.github/workflows/ci-cd.yml](mdc:.github/workflows/ci-cd.yml)

### 工作流阶段

1. **代码质量检查**: flake8, black, isort, mypy, bandit, safety
2. **单元测试**: pytest with coverage
3. **集成测试**: 数据库和Redis集成测试
4. **安全扫描**: bandit和safety漏洞扫描
5. **部署**: 自动部署到生产环境

### 部署脚本

- **生产部署**: [deploy.sh](mdc:deploy.sh) - 完整生产环境部署
- **CI/CD部署**: [deploy-ci.sh](mdc:deploy-ci.sh) - CI/CD环境专用部署
- **快速修复**: [quick-fix-deploy.sh](mdc:quick-fix-deploy.sh) - 紧急修复部署

### Docker配置

- **主配置**: [docker-compose.yml](mdc:docker-compose.yml) - 服务编排
- **Dockerfile**: [Dockerfile](mdc:Dockerfile) - 应用镜像构建
- **中国镜像**: [Dockerfile.china](mdc:Dockerfile.china) - 国内网络优化

### 环境配置

- **生产环境变量**: [env.production](mdc:env.production)
- **Nginx配置**: [nginx.production.conf](mdc:nginx.production.conf)
- **SSL证书**: [ssl_certs/](mdc:ssl_certs/)

## 部署流程

### 1. 本地CI/CD测试

```bash
# 运行本地CI/CD测试（无Docker）
./run-local-cicd-no-docker.sh

# 运行完整本地CI/CD测试（需要Docker）
./run-local-cicd.sh
```

### 2. 代码质量修复

```bash
# 自动修复代码质量问题
./fix-code-quality.sh
```

### 3. 部署到生产环境

```bash
# 使用CI/CD部署脚本
./deploy-ci.sh

# 或使用完整部署脚本
./deploy.sh
```

## 健康检查

- **本地健康检查**: `http://localhost/health/`
- **生产健康检查**: `http://47.103.143.152/health/`
- **健康检查脚本**: [monitoring/health_check.py](mdc:monitoring/health_check.py)

## 监控和日志

- **应用日志**: [logs/django.log](mdc:logs/django.log)
- **Gunicorn日志**: [logs/gunicorn_*.log](mdc:logs/)
- **监控脚本**: [monitoring/](mdc:monitoring/)

## 故障排除

### SSH连接问题
- 检查SSH密钥配置
- 验证GitHub Secrets设置
- 使用 [verify-ssh-config.sh](mdc:verify-ssh-config.sh) 生成新密钥

### 部署失败
- 检查环境变量配置
- 验证Docker服务状态
- 查看部署日志

### 代码质量检查失败
- 运行 [fix-code-quality.sh](mdc:fix-code-quality.sh)
- 检查依赖版本兼容性
- 修复安全漏洞

## 最佳实践

1. **本地测试优先**: 在本地通过所有CI/CD检查后再推送
2. **小步提交**: 频繁提交小的、可测试的更改
3. **监控部署**: 部署后检查健康状态和日志
4. **回滚准备**: 保持部署脚本的回滚能力
5. **环境隔离**: 开发、测试、生产环境完全隔离

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
