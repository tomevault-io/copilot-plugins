## pycmdb

> **易用** - 让用户（运维人员和 AI Agent）能够轻松使用

# pycmdb - Git-based CMDB

## 项目目标

**易用** - 让用户（运维人员和 AI Agent）能够轻松使用
**安全** - 配置变更可追溯、可审计、可回滚

### 设计原则

1. **对运维人员**: 清晰的 CLI 输出、直观的配置格式、友好的错误提示
2. **对 AI Agent**: 结构化的输出、确定性的行为、完整的上下文信息

### AI Agent 支持

所有命令支持 `--json` 输出，便于程序解析：

```bash
# 获取变更列表（JSON）
pixi run detect --json

# 校验配置（JSON，含错误详情）
pixi run validate --all --json
```

所有功能迭代都应围绕这两个目标和两类用户的需求进行权衡。

## 项目概述

Git-based CMDB with change detection and local deployment. 使用 Git 作为配置管理数据库，通过文件变更检测实现配置变更追踪和发布。

## 环境配置

- **包管理**: pixi
- **Python**: >=3.10
- **依赖**: pyyaml, jsonschema, jinja2

### 常用命令

```bash
# 安装依赖
pixi install

# Web 界面（浏览器访问 http://localhost:5000）
pixi run web

# 检测变更
pixi run detect

# 按类型过滤检测 (hosts/host_groups/services)
pixi run detect --type hosts

# 指定目标文件检测
pixi run detect --targets web-01,web-02

# JSON 格式输出（便于 AI 解析）
pixi run detect --json

# 校验变更（只校验检测到的变更）
pixi run validate

# 校验所有配置
pixi run validate --all

# 校验所有配置（JSON 格式）
pixi run validate --all --json

# 部署变更
pixi run deploy

# 按类型部署
pixi run deploy --type services

# 指定目标部署
pixi run deploy --targets api-gateway
```

## 目录结构

```
pycmdb/
├── publish/           # 用户可编辑的配置目录
│   ├── hosts/        # 主机配置
│   │   ├── config/   # 主机配置文件
│   │   ├── _schema.json
│   │   └── _defaults.yaml
│   ├── host_groups/  # 主机组配置
│   │   ├── config/
│   │   ├── _schema.json
│   │   └── _defaults.yaml
│   └── services/     # 服务配置
│       ├── config/
│       ├── _schema.json
│       └── _defaults.yaml
├── scripts/          # CLI 核心代码
│   ├── cli.py        # CLI 入口 (detect/validate/deploy 命令)
│   ├── detector.py  # 变更检测
│   ├── executor.py   # 变更执行
│   ├── validator.py  # 配置校验
│   └── setup_hooks.py
├── hooks/            # Git hooks (pre-commit)
│   ├── hosts_*.py
│   ├── hostgroups_*.py
│   └── services_*.py
└── pixi.toml         # pixi 配置
```

## 配置类型

每种配置类型 (hosts/host_groups/services) 都在 `publish/` 目录下:
- **config/**: 配置文件（无扩展名）
- **_schema.json**: JSON Schema 验证规则
- **_defaults.yaml**: 默认值
- **config/*.yaml**: 配置文件
- **_schema.json**: JSON Schema 验证规则
- **_defaults.yaml**: 默认值

### 命名规范

所有配置的 `name` 字段必须与文件名一致（无扩展名）：

```yaml
# hosts/web-01
name: web-01        # 必须与文件名一致
ip: 10.0.1.1

# host_groups/web
name: web           # 必须与文件名一致

# services/api-gateway
name: api-gateway   # 必须与文件名一致
```

- **name/hostname**: 小写字母开头，只含 `a-z0-9-`
- **version**: 语义化版本 `x.y.z`

### 部署锁

deploy 命令使用文件锁防止并发执行，避免多人同时操作导致冲突：

- 锁文件: `.deploy.lock`
- 自动清理: 进程崩溃后锁会自动释放
- 超时: 锁超过 30 分钟自动过期

当另一人正在部署时，会提示：
```
[ERROR] 部署正在进行中 (PID: 12345, 开始时间: 2026-05-06T10:30:00)
```

### services.hosts 字段格式

独立 host 直接写名称，host_group 使用 `group:` 前缀：

```yaml
hosts:
  - web-01                    # 独立 host
  - web-02                    # 独立 host
  - group:web                 # host_group，会展开为所有成员
```

## CLI 架构

- `scripts.cli`: Click 入口，提供 detect/validate/deploy 命令
- `scripts.detector`: 检测 git 变更，返回 Change 对象列表
- `scripts.validator`: 校验配置完整性和关联关系
- `scripts.executor`: 执行变更，调用 hooks/

## hooks 机制

变更执行时会调用对应类型的 hook 脚本:
- `hooks/{type}_new.py` - 新增配置
- `hooks/{type}_update.py` - 更新配置
- `hooks/{type}_delete.py` - 删除配置

Git pre-commit hook 位于 `hooks/pre-commit`，通过 `scripts/setup_hooks.py` 安装。

## 业务校验规则

配置校验除了 JSON Schema 验证外，还有业务规则校验:

| 配置类型 | 规则 | 错误信息示例 |
|---------|------|-------------|
| hosts | 文件名（无后缀）== name | `文件名 web-02 与 name web-01 不匹配` |
| host_groups | 文件名（无后缀）== name | `文件名 db-servers 与 name web-servers 不匹配` |
| services | 文件名（无后缀）== name | `文件名 gateway 与 name api-gateway 不匹配` |

## 自动提交规则

`pixi run deploy` 执行成功后，会自动将变更文件 git add 并 commit，然后 push 到远程仓库。

commit 信息格式: `{新增|更新|删除} {hosts|host_groups|services}: {文件名}`

---
> Source: [jwl93/pycmdb](https://github.com/jwl93/pycmdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
