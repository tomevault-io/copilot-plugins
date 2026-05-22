## restaurant-agent

> 所有对话、回复、代码注释、文档输出均使用简体中文。与用户的交流必须使用中文。

# Restaurant-Agent

## 语言要求

所有对话、回复、代码注释、文档输出均使用简体中文。与用户的交流必须使用中文。

智能美食推荐系统，基于 AI Agent 架构，接收用户的自然语言美食查询，通过多步骤编排流程调用高德地图 API 搜索餐厅，并返回筛选排序后的推荐结果。

## 技术栈

- **Web 框架**: FastAPI (端口 9000)，提供 REST API 和静态前端页面
- **Agent 编排**: LangGraph (`langgraph`) — 状态图驱动的多步骤工作流
- **LLM**: 阿里云 DashScope (`qwen-plus`)，兼容 OpenAI SDK，通过 `langchain-openai` 调用
- **地图 API**: 高德地图 — 地理编码 (`/v3/geocode/geo`) 和周边搜索 (`/v3/place/around`)
- **LLM 编排模式**: Plan-and-Execute — Planner 先制定计划，Executor 逐步执行子图
- **部署**: Docker + docker-compose，Python 3.11-slim 镜像

## 项目结构

```
Food/
├── main.py                          # FastAPI 入口，定义 /api/recommend、/health、/api/info、/
├── config.py                        # 环境变量加载（.env），阿里云+高德 API Key 配置
├── .env                             # API Key 和默认城市/坐标
├── index.html                       # 前端单页面（原生 HTML/CSS/JS，调用 /api/recommend）
├── Dockerfile / docker-compose.yml  # 容器化部署
├── requirements.txt                 # Python 依赖
│
├── plann_and_execute/               # Plan-and-Execute 编排器核心
│   ├── state.py                     # OrchestratorState、Plan、PlanStep 数据模型
│   ├── agent.py                     # LangGraph 状态图构建 + 条件路由 (replan/continue/end/error)
│   └── node.py                      # 四个节点实现：planner_node、executor_node、subgraph_node、formatter_node
│
├── sub_agents/                      # 四个子 Agent（每个是独立的 LangGraph 状态图）
│   ├── parse_query/parse_query.py           # 城市/位置解析 Agent — LLM 提取 city + 高德地理编码
│   ├── scenario_classifier/
│   │   ├── scenario_classifier.py           # 场景分类 Agent — 识别就餐场景并映射高德 POI type
│   │   ├── restaurant_taxonomy.json         # 高德餐饮 POI 类型映射表（中类-小类 → type 码）
│   │   └── parse_xlsx.py                    # 从高德 Excel 生成 restaurant_taxonomy.json 的工具脚本
│   ├── filter_criteria/
│   │   ├── filter_criteria.py               # 筛选条件提取 Agent — LLM 提取价格/评分/距离/排序
│   │   └── apply_filters.py                 # 对搜索结果应用筛选和排序逻辑
│   └── food_search/food_search.py           # 高德 POI 搜索 Agent — 分页查询 + 结果清洗
│
├── prompt/                          # LLM 提示词模板
│   ├── planner.py                   # Planner 的 System/User/Replan prompt
│   ├── parse_query.py               # 参数解析 prompt
│   ├── scenario_classifier.py       # 场景分类 prompt
│   └── filter_criteria.py           # 筛选条件提取 prompt
│
├── tests/                           # 单元测试
├── tools/estimate_poi_tokens.py     # 估算高德 POI 返回 token 占用
└── test.py                          # 编排器流程集成测试入口
```

## 核心架构：Plan-and-Execute 编排流程

### 状态图 (LangGraph)

```
planner → executor → subgraph → [路由判断] → executor (continue)
                                           → planner (replan)
                                           → formatter (end)
                                           → END (error)
```

### 四个节点职责

1. **planner_node** — 调用 LLM 将用户 query 分解为 `List[PlanStep]`，每个步骤指定调用的子图名和 `input_mapping`（步骤间数据依赖）。重规划时附带 `error_info` 和 `past_plans` 上下文。

2. **executor_node** — 只负责将 `current_step` 递增 1，指向下一步。

3. **subgraph_node** — 核心执行节点。根据 `current_step` 从 Plan 中取出对应的 `PlanStep`，通过 `input_mapping` 从之前步骤的结果中提取输入参数，然后调用注册的子图执行器。结果存入 `step_results`。

4. **formatter_node** — 汇总所有步骤结果，从 `step_results` 中提取 scenario/city/location/search_results/filters，调用 `apply_filters()` 过滤排序，取前 5 条，输出 JSON。

### 路由逻辑 (`route_after_subgraph_execution`)

- 有错误 → `replan`（回到 planner 重试，最多 3 次；不可恢复错误直接 `error`）
- 所有步骤完成 → `end` → formatter
- 否则 → `continue` → executor 执行下一步

### 数据依赖传递

步骤间通过 `input_mapping` 声明数据依赖，格式为 `"param_name": "step_N.field_name"`。例如 food_search 步骤依赖 step_1 的 `city`、`location` 和 step_2 的 `types`。`_prepare_step_input()` 解析这些映射从 `step_results` 中取值。

## 四个子 Agent

| 子图 | 功能 | 关键输出 |
|------|------|---------|
| `parse_query` | LLM 提取城市 → 高德地理编码获取经纬度；缺失时回退到默认位置 | `city`, `location` |
| `scenario_classifier` | LLM 识别就餐场景/菜系 → 查 `restaurant_taxonomy.json` 映射为高德 type 码 | `scenario`, `types` |
| `filter_criteria` | LLM 从 query 中提取价格范围、最低评分、距离、排序方式 | `filters` |
| `food_search` | 调用高德 `/v3/place/around`，分页查询 POI，清洗字段后返回 | `search_results` |

## API 端点

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/` | 返回 `index.html` 前端页面 |
| GET | `/health` | 健康检查 |
| GET | `/api/info` | Agent 元信息 |
| POST | `/api/recommend` | 核心推荐接口，请求体 `{"query": "..."}` |

POST `/api/recommend` 的流程：接收用户 query → 构建 `OrchestratorState` → `graph.invoke()` → 解析 `final_result` JSON → 返回 `RecommendResponse`。

## 关键配置 (.env)

- `ALIYUN_API_KEY` — DashScope API Key（格式 `sk-...`）
- `ALIYUN_MODEL` — 默认 `qwen-plus`
- `GAODE_API_KEY` — 高德地图 Web 服务 Key
- `DEFAULT_CITY` / `DEFAULT_LOCATION` — 用户未指定城市时的回退值

## 部署

```bash
docker-compose up -d   # 构建并启动，映射端口 9000
```

容器使用 Python 标准库 `urllib.request` 做健康检查，无额外系统依赖。

---
> Source: [bba70/Restaurant-Agent](https://github.com/bba70/Restaurant-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
