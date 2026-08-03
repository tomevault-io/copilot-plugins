## googletranslate-2api

> > 项目级指令。覆盖全局默认。全局规则见 `~/.claude/CLAUDE.md`。

# CLAUDE.md — googletranslate-2api

> 项目级指令。覆盖全局默认。全局规则见 `~/.claude/CLAUDE.md`。

---

## 项目概述

`googletranslate-2api` 是一个轻量代理服务：将谷歌翻译（`translate-pa.googleapis.com` 的 `translateHtml` 端点）封装成与 OpenAI `v1/chat/completions` 完全兼容的 API。

- **技术栈**：Python 3.10+ / FastAPI / Uvicorn / httpx / BeautifulSoup4 / markdownify / loguru
- **部署**：Docker Compose（Nginx 反向代理 :8088 → FastAPI）
- **核心特性**：OpenAI 格式兼容、SSE 流式响应、智能语言路由（中英互译）、Bearer Token 认证

---

## 行为准则

### 先思考再编码

不假设，不隐藏困惑，展示权衡。多种解读全展示。

### 简洁优先

最少代码解决问题，不做推测性工作。不写未要求的抽象。

### 精准修改

只改必须改的。匹配现有风格。不重构没坏的东西。

### 目标驱动

多步骤任务陈述计划并逐步验证。

---

## 项目结构

```
googletranslate-2api/
├── main.py                              # FastAPI 入口、路由、认证依赖
├── requirements.txt                     # Python 依赖
├── Dockerfile / docker-compose.yml      # 容器化
├── nginx.conf                           # 反代配置（proxy_buffering off 支持 SSE）
├── .env / .env.example                  # 环境变量
└── app/
    ├── core/config.py                   # Pydantic Settings 配置
    ├── providers/
    │   ├── base_provider.py             # 抽象基类 (ABC)
    │   └── googletranslate_provider.py  # 谷歌翻译实现（核心）
    └── utils/sse_utils.py               # OpenAI 兼容 SSE 格式化
```

### 关键文件速查

| 文件 | 职责 |
|------|------|
| `main.py` | FastAPI app、`lifespan`、`/v1/chat/completions`、`/v1/models`、`verify_api_key` 依赖 |
| `app/providers/googletranslate_provider.py` | 翻译核心：请求构建、上游调用、HTML→Markdown 解析、SSE 生成、智能语言路由 |
| `app/core/config.py` | 所有配置项（`API_MASTER_KEY`、`GOOGLE_API_KEY`、`NGINX_PORT` 等） |
| `app/utils/sse_utils.py` | `create_sse_data`、`create_chat_completion_chunk`、`DONE_CHUNK` |

---

## 关键技术约定

### 翻译流程

1. 取 `messages[-1].content` 作为待翻译文本
2. 智能路由：含中文 → 翻译成 `en`；否则 → `zh-CN`。可被请求体 `source_lang`/`target_lang` 覆盖
3. 上游 payload 格式：`[[[text], source_lang, target_lang], "te_lib"]`
4. 响应解析：`response_data[0][0]` 取翻译后 HTML → BeautifulSoup 提纯文本 → markdownify 转 Markdown
5. 单个 chunk 发完整翻译结果 + 结束 chunk + `[DONE]`

### 认证

- `verify_api_key` 依赖检查 `Authorization: Bearer <token>`
- `API_MASTER_KEY` 设为 `1` 或空 → 跳过认证（开发模式）
- 生产必须修改默认 `API_MASTER_KEY`

### 上游请求头

固定模拟浏览器：`Origin: https://stackoverflow.ai`、`Referer: https://stackoverflow.ai/`、`User-Agent: Chrome`、`x-goog-api-key: <GOOGLE_API_KEY>`、`Content-Type: application/json+protobuf`。

### SSE

Nginx `proxy_buffering off`。响应 `media_type="text/event-stream"`。

---

## 环境与运行

### 环境变量

| 变量 | 必需 | 默认 | 说明 |
|------|------|------|------|
| `API_MASTER_KEY` | 否 | - | API 访问密钥；`1` 或空 = 跳过认证 |
| `GOOGLE_API_KEY` | ✅ | - | 谷歌翻译 API 密钥（从浏览器 `x-goog-api-key` 抓取） |
| `NGINX_PORT` | 否 | `8088` | 服务端口 |
| `API_REQUEST_TIMEOUT` | 否 | `60` | 上游请求超时（秒） |

### 常用命令

```bash
# 开发
uvicorn main:app --reload --port 8088

# Docker 部署
docker-compose up -d
docker-compose logs app
docker-compose restart

# 测试端点
curl -X POST "http://localhost:8088/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_MASTER_KEY" \
  -d '{"model":"google-translate","messages":[{"role":"user","content":"Hello"}],"stream":true}'
``### Windows 注意

- 禁止 `.sh`，用 `node` 或 PowerShell
- 命令链接用 `; if($?) { }` 而非 `&&`
- `where.exe` 查找可执行文件

---

## 编码规范

- **Python 风格**：PEP 8 + 类型注解（已有函数签名全带 type hints）
- **不可变性**：优先创建新对象，不就地修改
- **错误处理**：`provider` 内异常 catch 后以错误 chunk 形式 SSE 返回；`main.py` 顶层 catch 返回 500
- **日志**：用 `loguru.logger`，不用 `print`
- **命名**：类 `PascalCase`，函数/变量 `snake_case`，常量 `UPPER_SNAKE_CASE`
- **文件组织**：小文件高内聚，按领域分（`core`/`providers`/`utils`）

### 扩展翻译提供商

实现 `BaseProvider`（`chat_completion` + `get_models`），在 `main.py` 替换 Provider 实例即可。

---

## 安全

- 永不在源码硬编码密钥，全部走 `.env`
- `GOOGLE_API_KEY` 启动时校验（`provider.initialize` 缺失即抛错）
- 生产部署必须改默认 `API_MASTER_KEY`
- 错误消息不泄露敏感数据

---

## Git 工作流

```
<type>: <description>
```

类型：feat / fix / refactor / docs / test / chore / perf / ci

---

## 已知限制 / 待办

- 非真正实时流式：整段翻译完成后单 chunk 返回
- 无缓存层、无速率限制、无健康检查端点
- 仅 `google-translate` 单模型
- `requirements.txt` 未锁版本

---
> Source: [lza6/googletranslate-2api](https://github.com/lza6/googletranslate-2api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
