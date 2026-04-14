## openclaw-clawevo-bsc-testnet

> 本项目使用 conda 虚拟环境，Python 解释器路径：

# Claude Code 项目配置

## Python 环境

本项目使用 conda 虚拟环境，Python 解释器路径：

```
/Users/bolin/miniconda3/envs/develop/bin/python
```

运行 Python 脚本时请使用：
```bash
/Users/bolin/miniconda3/envs/develop/bin/python script.py
```


## 数据库

PostgreSQL 18（Homebrew，trust 模式，无密码）：

```bash
# 启动（需要 LC_ALL=C 避免多线程错误）
LC_ALL="C" /usr/local/opt/postgresql@18/bin/pg_ctl -D /usr/local/var/postgresql@18 -l /usr/local/var/postgresql@18/server.log start

# 停止
/usr/local/opt/postgresql@18/bin/pg_ctl -D /usr/local/var/postgresql@18 stop

# 连接
/usr/local/opt/postgresql@18/bin/psql -U bolin -d news
```

- 用户名：`bolin`
- 生产数据库：`news`
- 测试数据库：`news_test`
- 连接字符串：`postgresql://bolin@localhost:5432/news`
- 异步连接：`postgresql+asyncpg://bolin@localhost:5432/news`

## 安全规范

### 敏感信息管理

1. **禁止在文档中写入真实密钥**
   - API 密钥、AccessKey、Secret 等敏感信息只能存放在 `.env` 文件中
   - 文档中使用占位符表示，如：`<your_access_key>`、`<your_secret_key>`
   - `.env` 文件已加入 `.gitignore`，不会被提交到版本控制

2. **敏感信息示例**
   - 阿里云 AccessKey/Secret（OSS、SMS 等服务）
   - 数据库连接密码
   - JWT Secret
   - 第三方 API 密钥（高德、Google Maps、OpenAI 等）

3. **环境变量文件**
   - `.env` - 真实配置（本地使用，不提交）
   - `.env.example` - 示例模板（可提交，使用占位符）

### 代码安全

1. **避免硬编码敏感信息**
   - 所有密钥通过 `settings` 或环境变量获取
   - 禁止在代码中直接写入密钥字符串

2. **日志安全**
   - 日志中不输出完整的敏感信息
   - 手机号等信息需脱敏处理，如：`158****1686`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coinmini)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/coinmini)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
