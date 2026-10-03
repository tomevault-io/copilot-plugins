## cnki-deepsearch

> - 项目名：cnki-deepsearch (0.2.0)

# CNKI MCP — 知网学术搜索 MCP 服务器

## 项目信息

- 项目名：cnki-deepsearch (0.2.0)
- GitHub：weileyao2005/cnki-deepsearch
- 安装方式：`pip install -e .`（editable 开发模式）
- MCP 入口命令：`cnki-search-mcp` / `cnki-deepsearch` → `cnki_mcp.server:run`
- 环境变量：`CNKI_HEADED=true` 使用有头浏览器（推荐，便于人工验证）

## 代码结构

```
cnki_mcp/
├── server.py         # MCP 工具定义 + handler + 看门狗/自愈
├── browser.py        # Playwright 浏览器自动化（串行锁/自适应超时/拦截识别）
├── query_builder.py  # CNKI 专业检索表达式构造
│   ├── build_expression()    # 结构化参数 → 专业检索表达式
│   └── parse_natural_query() # 自然语言 → 结构化参数
└── parser.py         # HTML/JSON 解析（搜索结果/文章详情/引用数据）
version1.0/           # GitHub 原版备份（2026-08-14 存档，无修复）
```

## 2026-08-14 排查出的关键事实（改代码前必读）

1. **`YE BETWEEN ('a','b')` 被知网风控拦截**（2026-08-12 知网改版后），返回 HTTP 200 + "抱歉，暂无数据，请稍后重试"。年份范围必须用 `YE >= 'a' AND YE <= 'b'`。已做交替对照实验 100% 复现。
2. **知网正常页面正文永久含有"拼图"字样**——任何按页面文本检测滑块验证码的逻辑都是误报，会让流程在填搜索词之前就等待不存在的验证码。不要恢复此类检测。
3. **串行锁必须"中断即释放"**：用户中断工具调用后，被中断的任务不能占着 `_lock`（曾导致后续所有调用"没有反应"）。`_run_locked` 用 fire-and-forget cancel + 立即释放锁实现；被中断/超时的浏览器标记 `_dirty`，下次调用前重建。
4. **基础搜索流程（URL/选择器/填词/点击）与 GitHub 原版一致，未改动**——原版与修复版实测均 17.x 秒返回 20 条。故障全部来自上述风控/锁/误报问题，不是流程本身。
5. 自适应超时：基础 60 秒；仅 URL 跳转验证页（真实验证码）时放宽（`_extend_deadline`）。看门狗：有头 330 秒/无头 100 秒兜底，保证 MCP 必有响应。

## MCP 搜索注意事项

- `cnki_search` 的 `field` 参数控制搜索字段（SU/TI/KY/AU/AF/LY 等）；与 `author`/`organization` 等筛选参数同时使用时走专业检索
- `cnki_professional_search` 语法：AU 用 `=`（精确），AF 用 `%`（模糊），SU 用 `%=`（相关）
- 每次搜索成功后 cookies 自动保存到 `~/.cnki_cookies.json`
- 浏览器单例：同一 MCP session 内浏览器只启动一次；连续失败自动重建
- 检索频率过高会触发知网风控（间歇性"暂无数据"），重试间隔建议 ≥30 秒

---
> Source: [weileyao2005/cnki-deepsearch](https://github.com/weileyao2005/cnki-deepsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
