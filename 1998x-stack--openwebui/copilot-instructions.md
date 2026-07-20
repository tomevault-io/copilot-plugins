## openwebui

> OpenWebUI Docker 部署，连接 iAgent Gateway (OpenAI 兼容) + 2 台 Ollama 服务器 + OpenCode Gateway。

# OpenWebUI 本地部署

OpenWebUI Docker 部署，连接 iAgent Gateway (OpenAI 兼容) + 2 台 Ollama 服务器 + OpenCode Gateway。

## 文件结构

```
~/openwebui/
├── docker-compose.yml        # 容器编排（镜像、端口、环境变量）
├── .env                      # 密钥（WEBUI_SECRET_KEY, OPENAI_API_KEY）
├── .gitignore                # .env
├── fix_models_v4.py          # 模型 patch 脚本（容器重建后必须运行）
├── CLAUDE.md                 # 本文件
├── gateway/                  # OpenCode Gateway 服务代码
│   ├── server.py             # FastAPI 服务端（spawn opencode CLI, JSONL→SSE）
│   ├── pipe.py               # OpenWebUI Pipe 函数（chat → Gateway bridge）
│   ├── register.py           # Pipe 函数注册到 OpenWebUI
│   └── start.sh              # 启动 Gateway 脚本
├── tools/                    # OpenWebUI 插件工具集
│   ├── _plugin.py            # PluginBase + PluginRegistry (factory)
│   ├── rag_search.py         # RAG 知识库搜索工具
│   ├── can_query.py          # CAN DB SQL 查询工具
│   ├── query_server.py       # CAN DB HTTP 代理服务
│   ├── can-text2sql-skill.md # CAN Text2SQL Skill 系统提示词
│   ├── register.py           # 一键注册所有工具
│   └── deploy.sh             # 一键部署所有工具
├── workspace/                # OpenCode 运行时工作区
│   ├── projects/             # nano-gpt, sticky-notes
│   ├── tutorials/            # 8 个主题的教程 + 测试
│   └── data/                 # 共享数据集 (shakespeare.txt)
└── docs/
    ├── README.md             # 文档索引
    ├── architecture.md       # 服务拓扑、数据流、网络
    ├── configuration.md      # 环境变量、DB 配置详解
    ├── models.md             # 模型来源、白名单、Embedding
    ├── adding-models.md      # 新增模型指南（含 vLLM 示例）
    ├── opencode-gateway.md   # OpenCode Gateway 架构、API、Pipe 机制
    ├── opencode-gateway-reference.md  # Gateway 原始 README
    ├── troubleshooting.md    # 故障记录
    ├── operations.md         # 启停、备份、升级、监控
    └── access-control.md     # 多用户权限：Tool/Skill/Function/Pipe 可见性机制
```

## 服务拓扑

```
浏览器 :3000 → OpenWebUI Docker (v0.9.6)
                  ├── iAgent Gateway (https://iagent.iauto.com/v1)  — chat + embedding
                  ├── Ollama #1 (192.168.89.37:11434)                — 本地 LLM
                  ├── Ollama #2 (192.168.146.137:11434)              — 本地 LLM (主)
                  └── OpenCode Gateway (host :14099)                  — Agent 工具调用
```

## 关键端口

| 端口 | 服务 | 说明 |
|------|------|------|
| 3000 | OpenWebUI | Web UI |
| 14099 | OpenCode Gateway | FastAPI, 需要手动启动 |

## 模型白名单（5 个聊天 + 1 个 Embedding）

通过 patch `utils/models.py` 和 `routers/openai.py` 实现：

| 模型 | 来源 |
|------|------|
| `iagent/standard` | iAgent (DeepSeek-V4-Flash) |
| `qwen3.5-b300/Qwen3.5-122B` | iAgent (最快 134tps) |
| `deepseek.v.4-b300/DeepSeek-V4-Flash` | iAgent |
| `qwen3-coder:30b` | Ollama |
| `qwen3:14b` | Ollama |

Embedding: `bge-m3-b300/bge-m3` (iAgent, 仅 RAG 后台使用，不在聊天模型列表)

> **注**：Ollama 端实际有 12 个模型在白名单中（见 `fix_models_v4.py`），但 `DEFAULT_MODELS` 环境变量只预注册了 2 个 Ollama 模型。其余模型可通过 OpenWebUI Admin 手动添加。

## 日常操作

### 启动

```bash
cd ~/openwebui && docker compose up -d
cd ~/openwebui/gateway && bash start.sh 14099 /workspace/data/xieming/openwebui/workspace &
```

### 停止

```bash
docker compose down   # OpenWebUI
fuser -k 14099/tcp    # Gateway
```

### 重启

```bash
docker restart open-webui
```

### 重建容器（应用 docker-compose.yml 变更）

```bash
cd ~/openwebui && docker compose up -d

# ⚠️ 重建后必须重新应用模型 patch！
docker cp ~/openwebui/fix_models_v4.py open-webui:/tmp/
docker exec open-webui python3 /tmp/fix_models_v4.py
docker restart open-webui
```

### 部署工具更新

```bash
cd ~/openwebui/tools && bash deploy.sh
```

### 部署 Pipe 更新

```bash
docker cp ~/openwebui/gateway/pipe.py open-webui:/tmp/gateway/pipe.py
docker cp ~/openwebui/gateway/register.py open-webui:/tmp/gateway/register.py
docker exec open-webui python3 /tmp/gateway/register.py
```

### 查看日志

```bash
docker logs open-webui --tail 50 -f
```

## 重要警告

1. **容器重建会丢失所有 patch**。`docker compose up -d` 后必须运行 `fix_models_v4.py` 并重启。
2. **镜像升级会丢失 patch**。`docker compose pull && docker compose up -d` 同理。
3. **`.env` 包含 API 密钥**，已在 `.gitignore`，不要提交。
4. **OpenCode Gateway 不会自动启动**，需要手动运行 `start.sh`。
5. **数据持久化**：`webui.db` 在 Docker volume `open-webui` 中（外部卷），容器销毁不会丢失对话数据。

## Patch 详情

### 为什么需要 patch

iAgent Gateway `/v1/models` 返回数字 `id`（102, 101, 36...）而非模型名（如 `iagent/standard`）。OpenWebUI 有两处独立代码读取模型列表：

| 位置 | 作用 | 不 patch 的后果 |
|------|------|----------------|
| `utils/models.py:fetch_openai_models` | UI 模型列表展示 | 显示 "102" 等数字 |
| `routers/openai.py:get_merged_models` | Chat 路由查找 | `Model '' was not found` |

### Patch 内容

1. **`utils/models.py`** — `fetch_openai_models`: 用 `identifier` 替换 `id`，白名单过滤 iAgent 模型；`fetch_ollama_models`: 白名单过滤 Ollama 模型
2. **`routers/openai.py`** — `get_merged_models`: 用 `identifier` 替换 `id`
3. **`webui.db`** — `model` 表注册 iAgent 模型元数据

### Re-apply after container rebuild

```bash
docker cp ~/openwebui/fix_models_v4.py open-webui:/tmp/
docker exec open-webui python3 /tmp/fix_models_v4.py
docker restart open-webui
```

## 相关项目

- `~/openwebui/gateway/` — Gateway 源码
- `~/openwebui/workspace/` — OpenCode 工作目录
   - `projects/nano-gpt/` — GPT 训练 + Shakespeare 生成
   - `tutorials/sklearn/` — sklearn 示例 + 可视化
   - `data/` — 共享数据集
   - `scratch/` — 临时尝试（.gitignore 排除）
- `~/iauto-codes/iagent/docs/MODELS.md` — 模型参考手册
- `~/.config/opencode/opencode.jsonc` — OpenCode CLI 配置
- `~/iauto-codes/iknow-xm/openwebui-tool/` — OpenWebUI Function 工具（RAG 知识库搜索 + CAN DB 查询）

---
> Source: [1998x-stack/openwebui](https://github.com/1998x-stack/openwebui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
