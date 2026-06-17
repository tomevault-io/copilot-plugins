## soul-s-travel

> Soul's Travel 的编码守则与项目规范。AI 工具必读，每次任务开始前对照执行。

# CLAUDE.md — 项目说明书

Soul's Travel 的编码守则与项目规范。AI 工具必读，每次任务开始前对照执行。

---

## 项目概述

**Soul's Travel** 是个人旅行记录与分析工具。AI 辅助录入行程数据（PDF / 自然语言 / 图片 / 穷游链接），多维度统计分析，生成旅行评价，支持分享。

技术栈：React 19 + Vite + Ant Design + Recharts（前端）/ Flask + SQLAlchemy + SQLite（后端）/ Claude API + NLU（AI 解析）

详细架构见 [ARCHITECTURE.md](ARCHITECTURE.md)，接口与测试见 [TECHNICAL.md](TECHNICAL.md)。

---

## 项目风格

- **视觉风格**：参考圆周旅迹（PiTravel），清新/轻盈/大留白；主色天蓝 `#0099ff`，背景极浅灰 `#f5f7fa`
- **语言**：UI 文字、注释、文档均用**简体中文**；代码标识符用英文
- **数据偏好**：金额存总价，人数/数量写 description；轻量列表字段存 JSON 字符串（`activities`、`transport`），不建子表

---

## 命名规范

| 层次 | 规范 | 示例 |
|------|------|------|
| Python 函数/变量/模块 | `snake_case` | `parse_text`, `trip_id` |
| Python 类（ORM Model） | `PascalCase` | `Trip`, `TripEvaluation` |
| Flask Blueprint | `*_bp` | `trips_bp`, `parse_bp` |
| React 组件/页面/文件名 | `PascalCase` | `TripCard.jsx`, `ChatPanel` |
| JS 变量/函数/API 方法 | `camelCase` | `tripApi.createShare()` |
| CSS 类名 | `kebab-case` | `.trip-card`, `.day-anchor` |
| 常量（Python/JS 全局） | `UPPER_SNAKE_CASE` | `DEST_CITIES`, `COUNTRY_MAP` |

---

## 禁止事项

- **禁止**在测试文件内用 `sys.path.insert` hack，路径通过 `pyproject.toml` 的 `pythonpath` 配置
- **禁止**硬编码端口号（`5002` / `5000`）在逻辑代码内；端口仅在 `app.py` 和 `vite.config.js` 各出现一次
- **禁止**浅断言（只验结构不验值）：`assert result['type'] == 'compound'` 不够，必须断言到叶子字段
- **禁止**单侧验证跨层 bug：后端+前端同时涉及的问题，后端测试通过 ≠ bug 已修复
- **禁止**正则写死单一货币/格式（如 `CNY`），必须枚举或通配
- **禁止**新增城市时只改一张表，解析层 `_EN_TO_ZH_CITY`、`DEST_CITIES`、`COUNTRY_MAP` 与评价层 `city_benchmarks.json` 四张表必须同步（前 3 个让解析器认得这座城，第 4 个让评价器有正确的住宿/日均参考价；漏掉第 4 个会让该城回退到国家平均，野生动物保护区被按城市经济档评判）
- **禁止**未经调研就"改进"相邻代码，外科手术式修改，只碰必须碰的

---

## 必须事项

- **后端 response 新增/重命名字段**时，同一次 PR 必须包含前端对应处理，检查方法：
  ```bash
  rg "result\.type|result\[.type.\]|\.type ===" frontend/src/
  ```
- **新增解析 bug 修复**时，Phase 1 必须先数源数据条数，再数输出条数，差值 > 0 = 不完整修复
- **新增 `parse_text()` 返回 type** 时，必须同步更新 extractor / `_nlu_parse` 路由 / 前端 `applyAction` / `ChatPanel` 提示文案 / 测试

---

## 常用命令

```bash
# 后端
cd backend
pip3 install -r requirements.txt     # 安装依赖
python3 seed.py                      # 导入样例数据（首次运行）
python3 app.py                       # 启动后端 → http://localhost:5002

# 后端测试
cd backend
python3 -m pytest tests/ -v          # 全量测试
python3 -m pytest tests/test_trips.py -v   # 单文件测试

# 前端
cd frontend
npm install                          # 安装依赖
npm run dev                          # 启动前端 → http://localhost:5000
npm run build                        # 构建生产包
npm run lint                         # ESLint 检查

# 一键启动（前后端同时）
./start.sh
```

> **Makefile（规划中）**：`make test`、`make lint`、`make coverage`、`make check-all` — 详见 [TECHNICAL.md](TECHNICAL.md) Harness 章节。

---

## 环境与端口

| 服务 | 地址 | 备注 |
|------|------|------|
| 后端 Flask | http://localhost:5002 | `backend/app.py` |
| 前端 Vite | http://localhost:5000 | `frontend/vite.config.js` |
| API 代理 | `/api/*` → `:5002` | Vite proxy，开发时透明 |

> **注意**：macOS 5000 端口理论上被 AirPlay 占用，但本项目前端固定使用 5000（`strictPort: true`）。若冲突，在系统设置中关闭「隔空播放接收器」。

### 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `ANTHROPIC_API_KEY` | AI 解析时必填 | Claude API 密钥，用于 `ai_parser.py` |
| `AI_API_KEY` | 图片解析时必填 | OpenAI 兼容 Vision API 密钥，用于 `image_parser.py` |
| `AI_MODEL` | 否 | Vision 模型名，默认由 `image_parser.py` 内定 |
| `AI_BASE_URL` | 否 | Vision API base URL，用于自定义端点 |

环境变量通过 `.env` 文件加载（`python-dotenv`），示例见 `.env.example`。

---

## 编码偏好

1. **最小改动原则**：每行改动必须能追溯到用户需求。不"顺手重构"，不"改进"无关代码。

2. **确定性测试优先**：解析器测试用固定字符串常量驱动，无需 mock；避免引入 `pytest-mock` 仅为绕过副作用——优先将副作用提取为可替换参数。

3. **内联类型优于子表**：`TripDay.activities`、`TripDay.transport` 存 JSON 字符串数组，不建独立表。适用场景：列表内容无需独立查询时。

4. **Lint**（规划中）：后端引入 `ruff` 替代 flake8/black，配置写入 `pyproject.toml`；前端已有 ESLint flat config（`eslint.config.js`）。

5. **变更验证**：涉及多层（前端 + 后端）的改动，验证必须走完完整数据流（用户输入 → 后端 → API 响应 → 前端渲染 → UI 更新）。

---

## CHANGELOG 书写规范

`CHANGELOG.md` 面向最终用户，不是开发日志。每次新增条目前对照下面四条铁律：

1. **每条都有粗体子标题**：`- **标题**：1–3 句描述`，标题一眼概括「用户看到的变化」。
2. **1–3 句精炼描述**：先说之前的现象/痛点，再说现在的行为；避免展开实现细节。
3. **无代码标识符**：不写文件名（`ai_parser.py`）、函数名（`parse_text()`）、类名（`TripCard`）、配置键、库名（`pdfplumber` / `jieba` / `Playwright` 等）；用功能性语言替代。
4. **无敏感/内部信息**：不暴露技术栈选型、算法名（TF-IDF / LinearSVC）、测试数量、DB 类型、接口路径、枚举到的代码常量等。

**版本头**：`## [x.y.z] - YYYY-MM-DD - 一句话主题摘要`。主题摘要是纯文本，不带粗体、不带全角冒号，也不能是某条 bullet 的半句截断。

**参考范本**：本仓库 `CHANGELOG.md` 里的 `[0.1.0]` 段，以及 `../us-stock-trading-assistant/CHANGELOG.md`、`../trading-calculate/CHANGELOG.md` 的任意已发布版本。

**自动化守护**：

- `.githooks/pre-push` + `.githooks/bump_changelog.py`：推送时把 `[Unreleased]` 晋升为版本块，并剥离 `feat:` / `> Theme:` 等技术遗留。**新 clone 或换机器后**务必执行一次激活：

  ```bash
  git config core.hooksPath .githooks
  ```

- `backend/tests/test_bump_changelog.py`：锁住晋升脚本的清洗逻辑，防止 hook 倒退。
- `backend/tests/test_changelog_format.py`：以 pytest 扫描 `CHANGELOG.md`，若 bullet 缺失粗体子标题、或出现 `*.py` / `*.jsx` / `func()` 这类代码标识符反引号、或版本头摘要残留 bullet 粗体，会直接失败。

---

## LLM 编码准则

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- Remove imports/variables/functions that YOUR changes made unused.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

**这份文档生效的标志**：diff 里没有无关改动，bug 修复前先写能复现的测试，澄清问题早于实现问题。

---
> Source: [xiulonnn-crypto/Soul-s-Travel](https://github.com/xiulonnn-crypto/Soul-s-Travel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
