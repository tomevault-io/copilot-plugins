## kingdeemcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install in development mode
pip install -e .

# Run the MCP server directly
kingdee-mcp

# Run with uvx (no install needed)
uvx kingdee-mcp

# Build distribution package
pip install hatchling
python -m hatchling build

# Install with all deps
pip install "mcp[cli]>=1.0.0" httpx pydantic
```

Run tests:
```bash
# Unit + mock tests (no Kingdee server required)
python -m pytest tests/ -v

# Integration tests (requires Kingdee server + env vars)
KINGDEE_SERVER_URL=http://your-server/k3cloud/ \
KINGDEE_ACCT_ID=... KINGDEE_USERNAME=... \
KINGDEE_APP_ID=... KINGDEE_APP_SEC=... \
KINGDEE_LCID=2052 \
python -m pytest tests/test_integration.py -v

# End-to-end workflow tests (requires real Kingdee env, same vars as above)
# Auto-skipped when KINGDEE_* env vars are absent — never commit credentials in source.
python -m pytest tests/e2e -v -m e2e
```

Run the evaluation layer (`evals/` — agent 行为评估与回归对比，详见 `evals/README.md`）:
```bash
# 离线自检整条评估管线（mock agent + mock 账套状态，无需金蝶、不写数据）
python -m evals.run_eval --dry-run

# 真实评估（连测试账套；需先把 evals/config.yaml 的 test_account.confirmed 设为 true）
python -m evals.run_eval
```

## Architecture

This is a single-file MCP Server (`src/kingdee_mcp/server.py`) that bridges AI clients to Kingdee Cloud K/3 ERP via its WebAPI.

### Request Flow

1. AI client calls a tool → FastMCP dispatches to the decorated async function
2. Function calls `_post(ep_key, payload)` with a `[form_id, {params}]` list
3. `_post()` checks the global `_session_id` cache; calls `_login()` if absent
4. On 401 or session-expired response, auto re-logins and retries once
5. All tools return JSON strings (success or error message from `_err()`)

### Key Internals

- **`_EP` dict** — maps short names (`"query"`, `"save"`, etc.) to full Kingdee WebAPI endpoint paths
- **`_session_id` global** — cached session cookie (`kdservice-sessionid`) from `LoginByAppSecret`
- **`FORM_CATALOG` dict** — maps `form_id` → `{name, alias[], fields}` used by `kingdee_list_forms` and `kingdee_get_fields` to help the AI discover form IDs without hitting the API
- **Pydantic input models** — each tool has a dedicated model (`QueryInput`, `SaveInput`, `BillIdsInput`, etc.) with `extra="forbid"` for strict validation

### Tool Categories

| Category | Tools |
|----------|-------|
| Metadata | `kingdee_list_forms`, `kingdee_get_fields` |
| Read-only queries | `kingdee_query_bills`, `kingdee_view_bill`, `kingdee_query_purchase_orders`, `kingdee_query_sale_orders`, `kingdee_query_stock_bills`, `kingdee_query_inventory`, `kingdee_query_materials`, `kingdee_query_partners` |
| Write operations | `kingdee_save_bill`, `kingdee_submit_bills`, `kingdee_audit_bills`, `kingdee_unaudit_bills`, `kingdee_delete_bills`, `kingdee_push_bill` |
| Workflow | `kingdee_query_pending_approvals`, `kingdee_query_workflow_status`, `kingdee_workflow_approve`, `kingdee_query_expense_reimburse` |
| SQL Server introspection | `kingdee_discover_tables`, `kingdee_discover_columns`, `kingdee_describe_table`, `kingdee_discover_metadata_candidates` |

### Environment Variables (required at runtime)

| Variable | Description |
|----------|-------------|
| `KINGDEE_SERVER_URL` | Server URL ending in `/k3cloud/` |
| `KINGDEE_ACCT_ID` | Account set ID |
| `KINGDEE_USERNAME` | Integration user name |
| `KINGDEE_APP_ID` | App ID from Kingdee admin |
| `KINGDEE_APP_SEC` | App Secret from Kingdee admin |
| `MCP_SQLSERVER_HOST` | SQL Server host (optional, for DB introspection) |
| `MCP_SQLSERVER_PORT` | SQL Server port (default 1433) |
| `MCP_SQLSERVER_DATABASE` | Database name |
| `MCP_SQLSERVER_USER` | SQL Server user (read-only recommended) |
| `MCP_SQLSERVER_PASSWORD` | SQL Server password |

### 自动记忆约定

### 代码注释格式（💡 REMEMBER）
代码中发现值得记忆的内容时，用以下格式标记：
```python
# 💡 REMEMBER: <简短描述>
# 💡 REMEMBER: <简短描述> — <根因/结论>
```
示例：
```python
# 💡 REMEMBER: httpx 0.28+ 默认 HTTP/2，金蝶不支持，必须显式传 http1=True，否则全 502
# 💡 REMEMBER: demo 环境 FLinkQty 不存在 — 用 FReceiveQty+FStockInQty 代替
```

### 提取脚本
```bash
python scripts/extract_remember.py   # 扫描并打印所有记忆条目
python scripts/extract_remember.py --update  # 追加到记忆文件
```

### 会话结束检查
每次长会话结束时，AI 应主动问：
> "这次有哪些发现需要记忆？"

## GitHub Pages (`docs/`)

The website at `https://wahailong.github.io/KingdeeMCP/` is a single static HTML file (`docs/index.html`). The deploy workflow (`.github/workflows/deploy-pages.yml`) triggers on push to `main` branch.

### Examples (`examples/`)

Business scenario examples showing how to use each tool. See `examples/README.md` for the full list. Useful as reference when helping users with Kingdee MCP queries.

### Workflow Hints (`examples/workflow-hints.md`)

> **核心参考文件，AI 执行具体任务时按需检索。**

关键内容：
- **单据生命周期状态机**：Save(草稿) → Submit(待审核) → Audit(已审核)
- **高层复合工具**：`kingdee_create_and_audit`、`kingdee_push_and_audit` 一站式操作
- **写操作完整流程**：Save → Submit → Audit（含 next_action 字段说明）
- **操作返回结构化字段**：success/next_action/errors/tip
- **常见错误处理**：关联数量已达上限、字段不存在、权限不足等

### 测试指南 (`tests/TEST_GUIDE.md`)

生产模块测试参考，包含：
- 生产订单、计划管理、资产管理、成本管理、调拨管理、质量管理、审计合规
- 各模块的 form_id 和可用字段
- 已知问题列表（Demo 环境未启用的模块）

---

## Harness 约束层

`tests/test_harness.py` 定义了操作链约束和反馈循环机制：

### 结构化操作结果

每次写操作返回：
```json
{
  "success": true,
  "bill_no": "PO001",
  "fid": "12345",
  "next_action": "submit",
  "errors": [{"message": "...", "reason": "...", "suggestion": "..."}]
}
```

**判断完成**：检查 `next_action == null` 且 `success == true`。

### 错误模式库 (`KNOWN_ERROR_PATTERNS`)

位于 `server.py` 顶部，自动匹配常见错误：

| 错误关键词 | 原因 | 建议操作 |
|-----------|------|---------|
| 502/Bad Gateway | 金蝶不支持 HTTP/2 | 确保 httpx 传 `http1=True` |
| 会话/session | Session 过期 | 调用 `_login()` 重新登录 |
| 关联数量 | 累计已达上限 | 检查 FReceiveQty+FStockInQty |
| 字段不存在 | 字段在当前账套未启用 | 用 `kingdee_get_fields` 确认 |
| 已被其他用户修改 | 乐观锁冲突 | 重新 `view_bill` 拉最新数据 |

错误命中后会携带 `matched.next_action_tool` 建议下一步工具。

### 单据状态机 (`DOC_LIFECYCLE`)

定义写操作后的状态流转：
- `save` → 草稿，下一步建议 `submit`
- `submit` → 待审核，下一步建议 `audit`
- `audit` → 已审核，操作完成
- `push` → 目标单草稿，下一步建议 `submit+audit`

---

## 测试问题排查流程

1. **运行测试查看错误**
   ```bash
   python -m pytest tests/ -v
   ```

2. **检查测试指南** (`tests/TEST_GUIDE.md`)
   - 查看已知问题列表（Demo 环境未启用的模块）
   - 确认 form_id 和字段是否正确

3. **查阅官方 API 文档**
   - 使用 MCP 浏览器访问 https://openapi.open.kingdee.com/ApiDoc
   - 搜索对应业务领域的单据类型
   - 查看请求参数、响应格式、代码示例

4. **常见 Demo 环境限制**
   - FLinkQty 不存在 → 用 FReceiveQty+FStockInQty 代替
   - 某些模块未启用 → form_id 存在但字段不存在
   - 权限限制 → 联系管理员开通

---

## 使用日志系统

详细文档见 `docs/usage_logging.md`。

### MCP 工具

| 工具 | 说明 |
|------|------|
| `kingdee_usage_report` | 查看详细使用报告（text/markdown/json） |
| `kingdee_usage_stats` | 获取当前会话统计摘要（JSON） |

### 命令行工具

```bash
# 生成使用报告
python scripts/usage_report.py

# 输出 Markdown 格式
python scripts/usage_report.py --format markdown -o report.md

# 分析指定日志文件
python scripts/usage_report.py --file /path/to/log.jsonl
```

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `MCP_USAGE_LOG` | `usage_log.jsonl` | 日志文件名 |
| `MCP_USAGE_LOG_DIR` | `.` | 日志目录 |

### 日志分析改进

基于日志可以分析：
- **高频工具** → 优先优化核心流程
- **高频错误** → 归类到 `KNOWN_ERROR_PATTERNS`
- **耗时分布** → 定位慢查询
- **未覆盖场景** → 发现新 API 需求

---
> Source: [WaHaiLong/KingdeeMCP](https://github.com/WaHaiLong/KingdeeMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
