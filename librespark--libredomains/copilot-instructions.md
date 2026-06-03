## libredomains

> > 本文档专为 AI 助手提供项目技术上下文，包含架构、工作流程、API 参考等技术细节。

# LibreDomains - AI 项目上下文

> 本文档专为 AI 助手提供项目技术上下文，包含架构、工作流程、API 参考等技术细节。

## 项目概述

LibreDomains 是一个基于 GitHub 的二级域名分发服务，用户通过提交 Pull Request 申请和管理域名。

**技术栈：**
- Python 3.x - 验证和部署脚本
- GitHub Actions - 自动化 CI/CD
- Cloudflare API - DNS 记录管理
- Git/GitHub - 配置存储和版本控制

## 项目架构

### 系统架构图

```
┌─────────────┐       ┌──────────────┐       ┌───────────────┐
│ GitHub 用户 │──────▶│ Pull Request │──────▶│ 验证检查脚本  │
└─────────────┘       └──────────────┘       └───────┬───────┘
                                                    │
                                                    ▼
┌─────────────┐       ┌──────────────┐       ┌───────────────┐
│ DNS 记录生效 │◀─────│ Cloudflare   │◀─────│ 部署脚本      │
└─────────────┘       └──────────────┘       └───────────────┘
```

### 分层架构

1. **配置存储层**：GitHub 仓库存储所有域名配置（JSON 格式）
2. **验证层**：Python 脚本 + GitHub Actions 自动验证
3. **部署层**：Python 脚本通过 Cloudflare API 部署 DNS 记录
4. **监控层**：定期健康检查确保域名正常工作

### 关键依赖

- **第三方服务商**：
  - Cloudflare（DNS 解析）
  - GitHub（代码托管、CI/CD）

- **外部 API**：
  - Cloudflare API（DNS 管理）
  - GitHub API（PR 评论）

## 文件结构

```
.
├── .github/
│   ├── ISSUE_TEMPLATE/           # Issue 模板
│   └── workflows/                # GitHub Actions 工作流
│       ├── deploy-domains.yml    # 部署域名
│       ├── health-check.yml      # 健康检查
│       ├── validate-domain.yml   # 验证域名申请
│       └── generate-stats.yml    # 生成统计
├── config/
│   └── domains.json              # 全局配置（域名列表、规则）
├── docs/                         # 自动生成的报告和数据
│   ├── stats.json                # 统计数据（自动生成）
│   └── stats_report.md           # 统计报告（自动生成）
├── domains/                      # 域名配置目录
│   ├── ciao.su/                  # ciao.su 域名的子域名配置
│   │   └── *.json                # 每个文件代表一个子域名
│   └── ciallo.de/                # ciallo.de 域名的子域名配置
├── scripts/                      # 所有脚本
│   ├── admin/                    # 管理员工具
│   ├── bot/                      # PR 机器人
│   │   └── pr_checker.py         # 检查 PR 中的域名申请
│   ├── cloudflare/               # Cloudflare 集成
│   │   └── cloudflare_manager.py # DNS 记录管理
│   ├── dns/                      # DNS 工具
│   ├── health/                   # 健康检查
│   │   └── domain_health.py      # 域名健康检查
│   ├── stats/                    # 统计脚本
│   │   └── domain_stats.py       # 生成域名统计
│   ├── utils/                    # 工具函数
│   └── validation/               # 验证逻辑
│       └── domain_validator.py   # 域名配置验证器
├── CLAUDE.md                     # 本文件（AI 项目上下文）
├── GUIDE.md                      # 用户和管理员完整指南
├── README.md                     # 项目主页
├── TERMS_OF_SERVICE.md           # 服务条款
└── requirements.txt              # Python 依赖
```

## 工作流程

### 域名申请流程（完整）

1. **用户提交**：
   - 用户 Fork 仓库
   - 在 `domains/[域名]/` 创建 JSON 配置文件
   - 提交 Pull Request

2. **自动验证** (`validate-domain.yml` 触发)：
   - `pr_checker.py` 检查 PR 中的文件变更
   - `domain_validator.py` 验证：
     - JSON 格式
     - 子域名命名规则
     - 是否被保留
     - 是否已被占用
     - 用户申请数量限制
     - DNS 记录数量限制
   - 机器人在 PR 中评论验证结果

3. **人工审核**：
   - 管理员检查自动验证结果
   - 审查域名用途合理性
   - 决定是否合并

4. **自动部署** (`deploy-domains.yml` 触发)：
   - PR 合并到主分支
   - `cloudflare_manager.py` 读取配置
   - 通过 Cloudflare API 创建/更新 DNS 记录
   - DNS 记录在几分钟内生效

5. **持续监控**：
   - `health-check.yml` 定期运行
   - `domain_health.py` 检查所有域名状态
   - 生成健康报告

### 域名更新流程

1. 用户编辑已有的 JSON 文件
2. 提交新的 Pull Request
3. 重复验证、审核、部署流程
4. `cloudflare_manager.py` 自动对比差异并更新

### 域名删除流程

1. 用户删除 JSON 文件
2. 提交 Pull Request
3. 审核通过后，部署脚本检测到文件被删除
4. `cloudflare_manager.py` 删除对应的 DNS 记录

## API 参考

### CloudflareManager 类

**位置**: `scripts/cloudflare/cloudflare_manager.py`

**主要方法**:

```python
class CloudflareManager:
    def __init__(api_key, email=None, config_path=None)
        # 初始化 Cloudflare 管理器

    def get_zone_id(domain)
        # 获取域名的 Cloudflare Zone ID
        # 返回: str (Zone ID)

    def list_dns_records(zone_id, type=None, name=None)
        # 列出指定 Zone 的 DNS 记录
        # 返回: list[dict] (记录列表)

    def create_dns_record(zone_id, type, name, content, ttl=3600, proxied=True)
        # 创建新的 DNS 记录
        # 返回: dict (创建的记录)

    def update_dns_record(zone_id, record_id, type, name, content, ttl=3600, proxied=True)
        # 更新已有的 DNS 记录
        # 返回: dict (更新的记录)

    def delete_dns_record(zone_id, record_id)
        # 删除 DNS 记录
        # 返回: bool (成功/失败)

    def sync_domain_records(domain, records)
        # 同步域名的所有记录（自动对比差异）
        # 返回: dict (操作摘要)
```

### DomainValidator 类

**位置**: `scripts/validation/domain_validator.py`

**主要函数**:

```python
def load_config(config_path=None)
    # 加载全局配置 (config/domains.json)
    # 返回: dict

def is_valid_domain_name(domain)
    # 验证子域名格式（RFC 1035）
    # 返回: bool

def is_reserved_subdomain(subdomain, config)
    # 检查是否为保留子域名
    # 返回: bool

def validate_domain_config(file_path, config)
    # 验证单个域名配置文件
    # 返回: dict (验证结果)

def validate_pull_request(pr_files, config)
    # 验证整个 PR 的所有变更
    # 返回: dict (验证报告)
```

## 配置参考

### 全局配置 (`config/domains.json`)

```json
{
  "domains": [
    {
      "name": "ciao.su",
      "enabled": true,
      "description": "主要域名",
      "cloudflare_zone_id": "xxx"
    }
  ],
  "record_types": ["A", "AAAA", "CNAME", "TXT", "MX"],
  "max_records_per_subdomain": 10,
  "max_subdomains_per_user": 3,
  "cloudflare_timeout": 30,
  "reserved_subdomains": [
    "www", "mail", "email", "webmail", "ns", "dns",
    "api", "cdn", "ftp", "sftp",
    "admin", "panel", "dashboard", "control",
    "dev", "test", "staging", "demo",
    "blog", "forum", "wiki", "docs", "tv",
    "app", "mobile", "static", "assets"
  ]
}
```

**字段说明**:
- `domains`: 可用域名列表，每个域名需要 Cloudflare Zone ID
- `record_types`: 支持的 DNS 记录类型
- `max_records_per_subdomain`: 每个子域名最多记录数（防止滥用）
- `max_subdomains_per_user`: 每个用户最多申请数（防止囤积）
- `reserved_subdomains`: 系统保留的子域名（安全和功能保留）

### 子域名配置格式

**位置**: `domains/[域名]/[子域名].json`

```json
{
  "description": "网站或服务描述",
  "owner": {
    "name": "用户名称",
    "github": "GitHub 用户名",
    "email": "联系邮箱"
  },
  "records": [
    {
      "type": "A",
      "name": "@",
      "content": "185.199.108.153",
      "ttl": 3600,
      "proxied": false
    },
    {
      "type": "CNAME",
      "name": "www",
      "content": "username.github.io",
      "ttl": 3600,
      "proxied": false
    }
  ]
}
```

**记录字段**:
- `type`: DNS 记录类型（A/AAAA/CNAME/TXT/MX）
- `name`: 子域名（`@` 表示根域名本身）
- `content`: 记录内容（IP/域名/文本）
- `ttl`: 生存时间（秒）
- `proxied`: 是否通过 Cloudflare 代理（CDN）

## 环境变量

### GitHub Actions Secrets

需要在仓库设置中配置：

- `CLOUDFLARE_API_TOKEN`: Cloudflare API 令牌（需要 DNS 编辑权限）
- `GITHUB_TOKEN`: GitHub 自动提供，用于 PR 评论

## 设计决策

### 为什么用 JSON 而不是 YAML？

- JSON 更严格，减少格式错误
- 易于编程语言解析
- GitHub 有更好的 JSON 语法高亮

### 为什么用 PR 而不是 Web 表单？

- 透明：所有变更可追溯
- 安全：利用 Git 版本控制
- 免费：无需维护服务器
- 自动化：GitHub Actions 集成

### 为什么限制每用户 3 个子域名？

- 资源公平分配（Cloudflare 免费额度有限）
- 防止恶意囤积
- 鼓励合理使用

### 为什么保留某些子域名？

- 安全：防止冒充管理界面（`admin`、`panel`）
- 标准：遵循互联网最佳实践（`www`、`mail`）
- 功能：为未来官方功能预留空间（`api`、`cdn`）

## 常见技术问题

### DNS 记录为什么不立即生效？

- Cloudflare API 通常 1-5 分钟同步
- DNS 缓存（TTL）可能延迟 1-24 小时
- 建议设置较短的 TTL（如 3600 秒）

### 如何调试验证脚本失败？

```bash
# 本地运行验证
python scripts/validation/domain_validator.py

# 检查 JSON 格式
cat domains/ciao.su/example.json | python -m json.tool
```

### 如何手动部署 DNS 记录？

```bash
# 使用 cloudflare_manager.py
python scripts/cloudflare/cloudflare_manager.py \
  --domain ciao.su \
  --config domains/ciao.su/example.json
```

## 扩展开发

### 添加新的 DNS 记录类型

1. 更新 `config/domains.json` 中的 `record_types`
2. 在 `domain_validator.py` 中添加验证逻辑
3. 测试 Cloudflare API 支持
4. 更新文档

### 添加新域名

1. 在 Cloudflare 中添加域名并获取 Zone ID
2. 更新 `config/domains.json`
3. 创建 `domains/[新域名]/` 目录
4. 更新 README.md

### 修改限制规则

编辑 `config/domains.json`:
- `max_records_per_subdomain`: 记录数限制
- `max_subdomains_per_user`: 用户申请数限制
- `reserved_subdomains`: 保留子域名列表

## 监控和维护

### 健康检查运行频率

- 每日自动运行（`health-check.yml`）
- 检查所有子域名的 DNS 解析状态
- 生成报告并提交到仓库

### 统计报告生成

- 每周自动运行（`generate-stats.yml`）
- 统计用户数、域名数、记录类型分布
- 更新 `docs/stats_report.md`

### 日志查看

- GitHub Actions 工作流日志
- 每次运行的详细输出
- 失败时的错误信息

---

**最后更新**: 2025-11-01
**维护者**: 查看 GitHub 仓库的 Contributors

---
> Source: [LibreSpark/LibreDomains](https://github.com/LibreSpark/LibreDomains) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
