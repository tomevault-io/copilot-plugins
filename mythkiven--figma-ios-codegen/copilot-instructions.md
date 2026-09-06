## figma-ios-codegen

> > 本文件是**跨客户端**的 Agent 入口（Cursor / Claude Code / Codex / 其他读 `AGENTS.md` 的工具）。

# Project Agent Rules

> 本文件是**跨客户端**的 Agent 入口（Cursor / Claude Code / Codex / 其他读 `AGENTS.md` 的工具）。  
> Cursor 细则：`.cursor/rules/figma-ios-codegen-workflow.mdc`（**按描述触发**，`alwaysApply: false`；命中 Figma→代码意图时加载）。  
> 新增 / 删除 / 重命名 `.cursor/rules/*.mdc` 时，必须同步更新本文件。  
> **给人读的对外文档**（安装 / 架构 / 设计约定 / FAQ）：[`docs/`](docs/README.zh.md) — Agent 生成代码时仍以本文件 + skills 为准，不必整份加载 `docs/`。

## 必读规则文件

| 文件 | 适用场景 |
|------|----------|
| `.cursor/rules/figma-ios-codegen-workflow.mdc` | 用户给出 Figma 链接并要求转/生成 iOS 代码时：**强制**按该 workflow 执行 |

详细 skill：`.cursor/skills/`（总入口 [`figma-ios-playbook`](.cursor/skills/figma-ios-playbook/SKILL.md)）。  
宿主定制：`.cursor/bindings/`（[`bindings/README.md`](.cursor/bindings/README.md)）。  
当前示例前缀 **MK**；换宿主只改 JSON，不改 skill。

---

## 何时启用本流程

**同时**满足才启用：

1. 提到 `figma` 或给出 `figma.com/...` 链接  
2. 表达「转/生成代码」意图

不触发：只问 Figma 用法、或未提 Figma 的「写个登录页」。

当前只支持 **iOS Swift（UIKit）**。

---

## 铁律（最高优先级）

1. **先数据包，后写码** — 无成功的 `{data_dir}/manifest.json` → 禁止阶段 2。已有 `data_dir` 可跳过重拉，仍须校验 `schema_version`。  
2. **阶段 2 唯一数据源** — 只读数据包 + **`.cursor/bindings/`**；禁止再调 Figma REST / MCP。  
3. **全量还原** — 应实现节点全部生成；未完成须按 mdc「未完成处置模板」开头声明。  
4. **TODO** — 仅 `// TODO(阶段3):` / `// TODO(阶段2):`。  
5. **不要改 preload 源码「自愈」** — 先 `python3 -m src.cli --self-check-only`；禁止 LLM 重写 `src/**`。

---

## 架构（1 → 2 → 4；阶段 3 仓外）

| 阶段 | 做什么 | 谁做 |
|------|--------|------|
| **1** | Figma URL → 规范化数据包（零 LLM） | 本地 CLI（或可选自建 MCP） |
| **2** | 数据包 + bindings → **基础 UI** Swift | Agent + `.cursor/skills` |
| **4** | 节点覆盖率 + 截图视觉验收 | Agent（阶段 2 之后强制） |
| **3** | PRD + 接口 → 商用业务代码 | **不在本仓库流程内** |

---

## 阶段 1：生成数据包

### 路径探测

```bash
WORKSPACE_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
PRELOAD_DIR_ENV="${FIGMA_PRELOAD_DIR:-$FIGMA_DATA_MCP_DIR}"   # 新名优先；旧名兼容

if [ -n "$PRELOAD_DIR_ENV" ] && [ -f "$PRELOAD_DIR_ENV/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$PRELOAD_DIR_ENV"
elif [ -f "$WORKSPACE_ROOT/.cursor/figma-ios-preload-data/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$WORKSPACE_ROOT/.cursor/figma-ios-preload-data"
elif [ -f "$WORKSPACE_ROOT/figma-ios-preload-data/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$WORKSPACE_ROOT/figma-ios-preload-data"
fi
# 有 DATA_PRELOAD_DIR → MODE=local；否则需已配置 fetch_figma_design（MODE=mcp）
```

**MODE=local（默认）：**

```bash
cd "$DATA_PRELOAD_DIR"
python3 -m src.cli "<figma_url>" --platforms ios
```

**MODE=mcp：** 本仓不附带 server；契约见 [`docs/mcp-interface.md`](docs/mcp-interface.md)。  
两者皆无 → 停止，禁止手写调 Figma API。

成功判据：输出含 `data_dir` 且 `{data_dir}/manifest.json` 存在。

---

## 阶段 2：读包 + bindings 生成基础 UI

| 信息 | 路径 |
|------|------|
| 高风险清单（**必读**） | `{data_dir}/audit.json` |
| 节点 / 索引 / 评论 / 切图 / 截图 | `design.json` / `index.json` / `comments.json` / `assets/ios/` / `screenshot.png` |
| 警告 | `manifest.json`（`schema_version` 须为 `1.0.0`） |
| **宿主** | `.cursor/bindings/host.json` + `*_map.json` |

推荐顺序：manifest → audit → **bindings** → index → 按需 design/comments → playbook/skills → `rg` 对账（commercial-delivery）→ 阶段 4。

- 类前缀 `<P>`：用户声明 → **可选外部 driver** → `host.class_prefix` → `class_prefix_fallback`  
- 交互 / 布局：按 `host.interaction` / `host.layout_engine`（勿默认强行 RxSwift）  
- 色/字/icon：查 map；未命中用 fallback

---

## 阶段 4：验收

1. 节点覆盖率先于视觉；< 100% 且无豁免 → 回阶段 2  
2. `screenshot.png` 视觉比对  
3. 输出覆盖率报告 + 视觉报告  

「无需比对」只豁免视觉，不豁免覆盖率。

---

## 验收清单（摘要）

- [ ] 阶段 1：`data_dir` + `manifest.json`  
- [ ] 阶段 2：host 基类；bindings；相对布局；`// Figma node:`；audit 对账  
- [ ] 阶段 4：覆盖率 100%（或合法豁免）+ 视觉报告  
- [ ] 无裸 `TODO:`  

---

## 客户端怎么加载

| 客户端 | 方式 |
|--------|------|
| **Cursor** | 描述触发 `.cursor/rules/figma-ios-codegen-workflow.mdc`；同时遵守本 `AGENTS.md` |
| **Claude Code** | 根目录 `CLAUDE.md`（`@AGENTS.md`）；**须显式 Read** mdc + `figma-ios-playbook`（不会自动套 Cursor skills） |
| **Codex / 其他** | 根目录 `AGENTS.md`；同样显式 Read mdc + playbook |

给人读的分端说明：[`docs/using-with-agents.zh.md`](docs/using-with-agents.zh.md) · [EN](docs/using-with-agents.md)。

冲突时：以 **全量还原 / 先数据包后写码 / 唯一数据源（含 bindings）** 为准；细则以 mdc 正文为准。

---
> Source: [mythkiven/figma-ios-codegen](https://github.com/mythkiven/figma-ios-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
