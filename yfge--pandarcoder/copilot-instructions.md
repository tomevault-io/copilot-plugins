## pandarcoder

> 这个文件记录了Claude开发过程中需要了解的关键信息和常用命令。

# Claude 开发备忘录

这个文件记录了Claude开发过程中需要了解的关键信息和常用命令。

## 项目概述

Claude Web是一个远程命令执行平台，允许用户通过Web界面操控Claude CLI，支持移动端访问和无人化托管。

## 核心技术栈

- **后端**: FastAPI + SQLAlchemy + Alembic + MySQL
- **前端**: Next.js + Radix UI + Tailwind CSS  
- **环境**: conda (Python) + nvm (Node.js)
- **代码规范**: pre-commit + black + prettier

## 常用命令

### 开发环境启动

**后端启动:**
```bash
cd backend
conda activate claude-web
uvicorn app.main:app --reload --port 8000
```

**前端启动:**
```bash
cd frontend  
nvm use 20
npm run dev
```

### 数据库操作

**连接数据库:**
```bash
mysql -h127.0.0.1 -uroot -P13306 -pPa88word
```

**创建迁移:**
```bash
cd backend
alembic revision --autogenerate -m "迁移描述"
```

**应用迁移:**
```bash
cd backend
alembic upgrade head
```

### 测试命令

**后端测试:**
```bash
cd backend
pytest -v
```

**代码格式化:**
```bash
# 后端
cd backend
black .
isort .

# 前端
cd frontend
npm run lint --fix
prettier --write .
```

## 项目结构说明

### 后端结构
```
backend/
├── app/
│   ├── api/          # API路由
│   ├── core/         # 核心配置
│   ├── db/           # 数据库配置
│   ├── models/       # 数据模型
│   ├── schemas/      # Pydantic模式
│   └── services/     # 业务逻辑
├── tests/            # 测试文件
├── alembic/          # 数据库迁移
└── requirements.txt  # Python依赖
```

### 前端结构
```
frontend/
├── src/
│   ├── app/          # Next.js App Router
│   ├── components/   # React组件
│   ├── lib/          # 工具函数
│   └── types/        # TypeScript类型
└── package.json      # Node依赖
```

## 数据库设计

### 主要表结构
- `users`: 用户信息
- `projects`: Git项目配置
- `tasks`: 命令执行任务
- `notifications`: 通知记录

## 开发注意事项

### 数据库连接
- 地址: `127.0.0.1:13306`
- 用户: `root`
- 密码: `Pa88word`
- 开发库: `claude_web`
- 测试库: `claude_web_test`

### 端口分配
- 后端API: `http://localhost:8000`
- 前端Web: `http://localhost:3000`  
- 数据库: `127.0.0.1:13306`

### 环境变量
后端环境变量在 `backend/.env` 中配置，包括：
- `DATABASE_URL`: 数据库连接字符串
- `SECRET_KEY`: JWT密钥
- `BACKEND_CORS_ORIGINS`: 前端域名

## 部署相关

### Docker化
项目支持Docker部署，相关配置：
- `docker-compose.yml`: 多服务编排
- `Dockerfile`: 镜像构建

### 环境管理
- 开发环境: 本地开发
- 测试环境: Docker容器
- 生产环境: Kubernetes集群

## 安全考虑

### 主要风险
1. **代码执行安全**: Claude CLI可执行任意命令
2. **数据泄露**: 项目代码和SSH密钥
3. **资源滥用**: 无限制命令执行

### 安全措施  
1. 容器化隔离执行环境
2. 命令白名单和权限控制
3. 敏感信息加密存储
4. 审计日志记录

## 故障排查

### 常见问题
1. **数据库连接失败**: 检查MySQL服务和端口
2. **迁移失败**: 检查数据库权限和表结构
3. **前端构建失败**: 检查Node版本和依赖
4. **API连接错误**: CORS预检请求失败，检查后端允许的请求头配置
5. **端口冲突**: 前端(3100)和后端(8100)端口配置不匹配

### 故障排查步骤

**API连接问题排查**:
```bash
# 1. 检查后端服务状态
ps aux | grep "uvicorn.*app.main" | grep -v grep
lsof -i :8100

# 2. 测试API健康检查
curl -v http://localhost:8100/health

# 3. 测试CORS预检请求
curl -X OPTIONS \
  -H "Origin: http://localhost:3100" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: X-Request-ID" \
  http://localhost:8100/health

# 4. 重启服务(如果需要)
cd backend && python -m uvicorn app.main:app --reload --port 8100
```

**前端错误诊断**:
- 打开浏览器开发者工具
- 查看Network面板中的API请求状态
- 查看Console面板中的错误日志
- 确认没有CORS相关错误

### 调试工具
- 后端: FastAPI自带的 `/docs` Swagger UI
- 前端: Next.js开发者工具 
- 数据库: MySQL Workbench或命令行
- 网络调试: curl + 浏览器开发者工具
- 详细文档: `docs/troubleshooting/api-connection-errors.md`

## 扩展功能规划

### MVP阶段 (1-2个月)
- 用户认证和项目管理
- 基础Git操作命令
- 简单Web界面

### 扩展阶段 (3-4个月)  
- 完整Claude CLI集成
- 实时状态监控
- 通知系统和移动端优化

### 优化阶段 (5-6个月)
- 性能优化和安全加固
- 监控告警系统
- 自动化运维工具

## 技术债务

### 当前已知问题
- [ ] 需要完善错误处理机制
- [ ] 需要添加更多单元测试
- [ ] 需要优化数据库查询性能
- [ ] 需要添加API限流机制

### 优化计划
1. **代码质量**: 提升测试覆盖率到80%+
2. **性能优化**: 添加缓存和查询优化
3. **安全加固**: 完善权限控制和审计
4. **文档完善**: 补充API文档和部署指南

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yfge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
