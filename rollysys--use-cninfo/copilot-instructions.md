## use-cninfo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这个 repo 是什么

`cninfo-cli` — 一个**按需召唤型**的 A 股公告抓取工具:CLI + Python 包,顺带给 Claude Code 提供一个 skill 入口。从巨潮资讯网(cninfo.com.cn)拉公告 PDF + PyMuPDF 提取的 md 全文,落本地缓存,二次访问秒级。

不是后台调度系统,不是数据池,不做增量同步 — 全是一次性命令,跑完即停。

## 常用命令

```bash
# 开发环境
uv venv --python 3.10
uv pip install -e ".[dev]"

# 跑测试(离线,32 个)
uv run pytest                       # 全跑
uv run pytest tests/test_api.py     # 单文件
uv run pytest -k "cache_hit"        # 按名字过滤

# Lint
uv run ruff check src tests
uv run ruff check --fix src tests   # 自动修

# 真机 smoke test(联网)
CNINFO_ALLOW_NETWORK=1 cninfo fetch-report 600519 --year 2024 --kind annual

# 重生成 fixture PDF(几乎不用)
python tests/fixtures/_make_pdf.py
```

## 架构关键点

**单向依赖**(从底到顶):

```
api.py        ← 纯函数,cninfo HTTP 客户端 + 时区/标题清洗
parser.py     ← PyMuPDF 包装
cache.py      ← ~/.cache/cninfo/ 三件套读写 + orgid_map
orgid.py      ← secCode→orgId(走 cninfo topSearch),用 cache.py 持久化
classify.py   ← argus_legal 薄包装(动态 import,缺失时抛 ClassifierUnavailable)
fetcher.py    ← 把上面拼起来:fetch_announcement / iter_* / find_periodic_report
cli.py        ← argparse,6 个子命令
```

**测试隔离**:`conftest.py` autouse fixture 默认禁用真实网络(`requests.{get,post}` 抛错),要联网测时设 `CNINFO_ALLOW_NETWORK=1` 或在测试里 mock。

**缓存命中策略**:`pdf + md + meta` 三件齐全才算命中;任一缺失走未命中路径。`--force` 跳过命中检查。

## 接口约束(继承自 cninfo,不要尝试"修复")

详见 [`docs/gotchas.md`](docs/gotchas.md) 14 条。最关键的 4 条:

1. **`pageSize` 服务端硬限 30** — 必须翻页(`api.py:query_all` 已处理)
2. **`stock` 必须 `<6位>,<orgId>`** — 仅传 6 位返 0 条;本仓库通过 `topSearch` 拉 orgId(`orgid.py`),首次发现的 orgId 也会顺带缓存进 `orgid_map.json`
3. **`announcementTime` 是 UTC epoch ms** — 必须按北京时区转(`api.py:epoch_ms_to_ann_date`),否则跨日公告 ann_date 差一天
4. **同一公告可能有 2 条记录** — announcementId 不同 PDF 内容相同;`is_periodic_report_body` 已排除摘要/审计/内控,但全市场场景下用户需自行去重

## 改动指引

**修接口参数 / 加 category / 加新场景**:
- 改 `api.py`(常量 / 函数)→ 同步更新 `docs/api_reference.md` 那张表
- 加 fixture(`tests/fixtures/`) → 加单测

**新增 CLI 子命令**:
- `cli.py` 的 `build_parser()` 加子 parser,`cmd_xxx` 写动作,默认 JSON 优先 + 表格可选
- 一定要在 `tests/test_cli.py` 加一个走 `main([...])` 的覆盖(可只看返回码 / stderr)

**改缓存布局**:
- 改 `cache.paths_for()` + 同步更新 `verify` / `prune`
- README 与 SKILL.md 的"缓存布局"段都要同步

**改 orgId 来源**:
- cninfo `topSearch` 端点未来若变,只动 `orgid.py:lookup_orgid`,接口签名保持
- `gotchas.md #2` 同步更新

## 不要做的事

- **不要把这工具改造成有后台 / 增量 / 数据池的系统** — 那是另一种产品,本 repo 的定位就是按需召唤
- **不要把 `argus_legal` 写进 `pyproject.toml` 依赖** — 它当前未发布到 PyPI,只能源码装,我们故意保持可选
- **不要在测试里发真实网络请求** — `conftest.py` 会拦截;要测真机用 `CNINFO_ALLOW_NETWORK=1` + 单独跑
- **不要扩展 docs/cookbook.py 当生产代码** — 它是"接口知识沉淀"的 reference,生产代码在 `src/cninfo/`
- **不要为下游 skill(dossier-fin / stock-5views)做特殊适配** — 让它们自己读 `~/.cache/cninfo/md/` 即可

## 文件森林角色

| 路径 | 角色 |
|---|---|
| `src/cninfo/` | Python 包(运行时) |
| `tests/` | pytest 离线测试 |
| `skill/SKILL.md` | Claude Code skill 入口(用户软链到 `~/.claude/skills/cninfo`) |
| `docs/` | 接口实测知识(api_reference / capabilities / gotchas / cookbook),只读参考 |
| `pyproject.toml` | uv / pip 标准 |
| `.github/workflows/ci.yml` | lint + 离线测试矩阵(3.10/3.11/3.12) |

## 项目状态(截至 2026-05-04)

**已开源**:[rollysys/use_cninfo](https://github.com/rollysys/use_cninfo)(public, MIT),`main` 分支 = 唯一发布分支。CI 已绿(3.10/3.11/3.12 矩阵)。

**已上游集成**:[duolongworld/AI_Renaissance#17](https://github.com/duolongworld/AI_Renaissance/pull/17) — 给 AI_Renaissance 项目加了 `data_sources/cninfo.py`(subprocess 调本工具 CLI)+ `skills/data/cninfo/SKILL.md`,PR 等 review。该 fork 在 `rollysys/AI_Renaissance:feat/data-cninfo` 分支上。

**未做的真机验证**(后续接到反馈时再补):
- `search --type ... --sub-type ...` 路径**没跑过真实 announcement_filter**(本机未装 argus_legal,只验证了缺失态报错 graceful)。装好 announcement_filter 后应跑一次 e2e 确认 classifier 在 cninfo 返回的中文 title 上分类合理
- 老 PDF 404(2024-06 之前)的失败重试 / failure 表机制只在 `gotchas.md #7` 提了,代码里**没实现** — 用户碰到时再加,现在不预先抽象
- `search` 命令对每条 ann_id 的 type/sub_type 标签**没缓存** — 全市场切片 + 标签过滤会反复调 classifier,后续若慢可在 `meta/<ann_id>.json` 写入 `tags` 字段

**接到 PR review 反馈时**:`/tmp/AI_Renaissance` 已清理,需要时重新 `git clone https://github.com/rollysys/AI_Renaissance.git && git checkout feat/data-cninfo` 改完 push 即可,branch 已存在。

---
> Source: [rollysys/use_cninfo](https://github.com/rollysys/use_cninfo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
