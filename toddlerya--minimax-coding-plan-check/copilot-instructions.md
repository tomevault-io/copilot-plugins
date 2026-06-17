## minimax-coding-plan-check

> Use when needing to check MiniMax Token Plan (coding plan) quota usage, remaining credits, or reset countdown. Triggered by queries like "check minimax quota", "how much coding plan left", "minimax额度".


# MiniMax Coding Plan 用量查询

## Overview
查询 MiniMax Token Plan（coding plan）的 5 小时滚动窗口剩余额度，包括已用量、总额度、重置倒计时，以及历史使用率统计。

## 核心要点：API Key 配置（最常出错的地方）

MiniMax 有两种 Key，**不能混用**：

| Key 类型 | 前缀 | 用途 |
|---------|------|------|
| 普通 API Key | `sk-xxxxxxxx` | 通用 API 调用 |
| **Token Plan Key** | `sk-cp-xxxxxxxx` | coding plan 额度查询 |

**错误症状：** API 返回 `status_msg: invalid token` 或 `status_code != 0`。
**解决方法：** 必须使用 `sk-cp-` 开头的 Key，普通 Key 无法调用 coding plan 接口。

## 配置步骤

### Step 1：获取 Token Plan API Key
1. 登录 MiniMax 开放平台
2. 进入「Token Plan」产品页面
3. 复制以 `sk-cp-` 开头的 Key

### Step 2：配置到各 Agent

**Claude Code（当前 Agent）：**
```bash
mkdir -p ~/.minimax-quota
echo '{"api_key": "sk-cp-你的Key"}' > ~/.minimax-quota.json
chmod 600 ~/.minimax-quota.json
```

**OpenClaw / 其他 Agent：**
不同 Agent 读取配置文件的位置可能不同。以下是常见方案：

**方案 A：环境变量（推荐）**
```bash
export MINIMAX_API_KEY="sk-cp-你的Key"
```
在 OpenClaw 启动脚本或 systemd service 中加入此环境变量。

**方案 B：直接在脚本中硬编码（仅临时使用）**
不推荐，但快速验证时可用：
```bash
API_KEY="sk-cp-你的Key" ./minimax-quota.sh
```

**方案 C：统一配置文件路径**
修改脚本 `CONFIG_FILE` 变量指向共享路径：
```bash
CONFIG_FILE="/path/to/shared/minimax-quota.json"
```

### Step 3：验证配置
```bash
./minimax-quota.sh
```
首次运行会交互式询问 Key 并保存；后续无需重复输入。

## API 信息速查

| 项目 | 值 |
|------|-----|
| 端点 | `https://www.minimaxi.com/v1/api/openplatform/coding_plan/remains` |
| 方法 | GET |
| Header | `Authorization: Bearer {API_KEY}` |
| 时间窗口 | 5 小时滚动窗口 |
| 当前模型 | MiniMax-M2.7 |

## 快速参考

```bash
# 标准调用（读取 ~/.minimax-quota.json）
./minimax-quota.sh

# 指定 Key（临时）
API_KEY="sk-cp-xxx" ./minimax-quota.sh

# 指定配置文件路径
CONFIG_FILE="/custom/path/minimax-quota.json" ./minimax-quota.sh
```

## 常见问题

**Q: 报错 `API 请求失败: invalid token`**
A: 使用了普通 API Key（`sk-`）而非 Token Plan Key（`sk-cp-`）。重新获取正确的 Key。

**Q: 报错 `API 请求失败: ...`**
A: Key 过期或无 Token Plan 订阅。请到 MiniMax 控制台确认服务状态。

**Q: 进度条显示乱码**
A: 终端不支持 ANSI 颜色码，不影响数据准确性。可忽略或改用 `TERM=dumb` 运行。

## 历史使用率查询

支持按日/周/月统计和指定时间段查询。

### Web API

```bash
# 获取每日统计（默认最近7天）
curl http://localhost:8080/api/daily
curl http://localhost:8080/api/daily?days=30

# 获取每周统计（默认最近4周）
curl http://localhost:8080/api/weekly
curl http://localhost:8080/api/weekly?weeks=12

# 获取每月统计（默认最近6个月）
curl http://localhost:8080/api/monthly
curl http://localhost:8080/api/monthly?months=12

# 指定时间段查询（格式：YYYY-MM-DD）
curl "http://localhost:8080/api/range?start=2026-03-01&end=2026-03-24"
```

### 触发词示例

- "查看这周的使用率"
- "上周每天用多少"
- "3月份的使用统计"
- "查询3月1日到3月15日的使用率"
- "最近一个月哪天用得最多"

## 脚本位置

当前 Agent 中的脚本：`/Users/evi1/AI-Assistant/scripts/minimax-quota.sh`

---
> Source: [toddlerya/minimax-coding-plan-check](https://github.com/toddlerya/minimax-coding-plan-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
