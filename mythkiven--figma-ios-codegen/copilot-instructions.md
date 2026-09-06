## figma-ios-codegen-workflow

> >-


# ⛔ 全量还原原则（最高优先级，覆盖所有阶段）

> 本节优先级**高于**下文所有阶段细节。任何阶段产物若与本节冲突，以本节为准。

> 📌 **适用范围**：本原则管的是**阶段 2 的 UI 节点覆盖率**（视觉/布局/样式还原）。
> 业务逻辑、接口对接、数据流、埋点等属于**阶段 3**（仓外：PRD + 接口文档），不在本原则约束范围内——阶段 2 遇到这类需求，用 mock + `// TODO(阶段3): ...` 即可。

**默认承诺**：把数据包 `{data_dir}/design.json` 中**所有应实现的节点**（按 `figma-ios-to-code-conventions` 跳过 Home Indicator / Status Bar / Device Frame / 设计标注后的剩余集合）**全部生成对应代码**。**禁止**「先做主体后补细节」式的渐进交付。

## 禁止行为

| ❌ 禁止 | ✅ 正确 |
|---|---|
| 把缺失节点伪装成「可选增量」 | 列出缺失节点 ID，声明违反全量还原，继续补齐 |
| 「为简化示例省略了 …」 | 直接生成完整代码；上下文不足用「未完成模板」 |
| 用截图比对替代节点对账 | 阶段 4 **之前**先做节点 ID 对账（`figma-ios-screenshot-verification`） |
| 把完成度当「偏好」征求意见 | 完成度是默认值，无需征求 |

## 唯一允许的简化场景（须显式列出，并标 TODO）

1. 节点对应**后端动态数据** → mock + `// TODO(阶段3): 接入 xxx 接口`
2. 用户在**本轮请求**中显式说「只做某某区块」/「快速验证某某」
3. 节点已被 `figma-ios-to-code-conventions` 明确列入跳过规则

> **TODO 格式（强制）**仅允许：
> - `// TODO(阶段3): …` — 业务 / 数据 / 接口
> - `// TODO(阶段2): …` — 本轮未决（map 未命中、VECTOR 未拆等）
>
> 禁止裸 `// TODO:` / `FIXME:`。交付前不得残留无限定语 TODO。

## 未完成处置模板（强制）

若无法一次性完成全量代码，**回复开头**声明：

> ⚠️ 本轮未完成全量还原。  
> 已实现节点：…  
> **缺失节点**：…（节点 ID + name）  
> 缺失原因：上下文/时间限制（**不是**设计选择）  
> 下一步：继续补齐，除非用户喊停。

## 与「视觉差异」的边界

- 颜色稍偏 / 间距 2pt → 视觉差异，阶段 4 报告
- 整块未生成 / 只画示意 → **节点缺失**，必须补全

跨客户端摘要见根目录 [`AGENTS.md`](../../AGENTS.md)；Skill 总入口见 [`figma-ios-playbook`](../skills/figma-ios-playbook/SKILL.md)。

---

# 适用场景（按描述触发）

**当且仅当**同时满足：

1. 提到 `figma` 或给出 `figma.com/...` 链接，且
2. 表达「转/生成代码」意图（figma → iOS / 根据设计稿生成 Swift 等）

❌ 不触发：只问 Figma 用法；「写个登录页」但未提 Figma。

当前版本**只支持 iOS Swift（UIKit）**。扩展其它端须另起 workflow，禁止往本文件叠多端逻辑。

> `alwaysApply: false`：非 Figma 会话不灌入本规则。命中上述意图时 Agent 应主动加载本 rule。

---

# 整体架构（1 → 2 → 4；阶段 3 在仓外）

```
┌──────────────────────────────────────────────────────────────────┐
│  阶段 1：数据包（本地 CLI；可选自建 MCP）· 无 LLM                   │
│  Figma URL → design.json / audit.json / assets / screenshot …    │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  阶段 2：基础 UI（LLM + skills + .cursor/bindings）               │
│  包 → Swift UIKit；mock + TODO(阶段3)；禁止再调 Figma API         │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  阶段 4：验收（本 workflow 内强制）                                │
│  节点覆盖率对账 → screenshot 视觉比对                               │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│  阶段 3：商用二次加工（PRD + 接口 · **不在本仓流程**）              │
│  grep TODO(阶段3) → 换真实数据 / 业务 / 路由 / 埋点                 │
└──────────────────────────────────────────────────────────────────┘
```

| 阶段 | 一句话 |
|------|--------|
| 1 | REST → 规范化事实包 |
| 2 | 包 → **能跑能看的基础 UI**（业务留 TODO） |
| 4 | 覆盖率 100% + 视觉报告 |
| 3 | 仓外把 TODO 换成商用逻辑 |

「基础 UI」= 视觉/布局/样式/交互骨架齐全；数据 mock；接口未接。

同步本目录到多工程时：重命名/删除 skill 或 preload 后清理下游旧路径。**单仓可忽略。**

---

# 阶段 1：数据包获取（强制第一步）

## 1.1 路径探测（双路径）

收到 Figma 链接后，先探测 preload CLI：

```bash
WORKSPACE_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"

# 优先新名；兼容旧名 FIGMA_DATA_MCP_DIR（历史命名，实为本地 CLI 路径）
PRELOAD_DIR_ENV="${FIGMA_PRELOAD_DIR:-$FIGMA_DATA_MCP_DIR}"

if [ -n "$PRELOAD_DIR_ENV" ] && [ -f "$PRELOAD_DIR_ENV/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$PRELOAD_DIR_ENV"
elif [ -f "$WORKSPACE_ROOT/.cursor/figma-ios-preload-data/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$WORKSPACE_ROOT/.cursor/figma-ios-preload-data"
elif [ -f "$WORKSPACE_ROOT/figma-ios-preload-data/src/cli.py" ]; then
    DATA_PRELOAD_DIR="$WORKSPACE_ROOT/figma-ios-preload-data"
fi

if [ -n "$DATA_PRELOAD_DIR" ]; then
    echo "MODE=local DATA_PRELOAD_DIR=$DATA_PRELOAD_DIR"
else
    echo "MODE=mcp"
fi
```

### MODE=local（本仓默认）

```bash
cd "$DATA_PRELOAD_DIR"
# 首次：cp config.example.json config.json，填 figma_tokens
python3 -m src.cli "<figma_url>" --platforms ios
```

前置：系统 `python3`（零第三方依赖）。

### MODE=mcp（可选，本仓不附带 server）

仅当本地无 preload，且已自建/注册 `fetch_figma_design` 时使用。  
契约：`docs/mcp-interface.md`。

```jsonc
{ "url": "<figma_url>", "platforms": ["ios"], "with_screenshot": true, "with_assets": true }
```

返回与本地 CLI 同构（`data_dir` + summary + warnings）。

**既无 preload、又无 MCP**：停止并提示安装本仓 preload 或设置 `FIGMA_PRELOAD_DIR`（旧名 `FIGMA_DATA_MCP_DIR`）。**禁止** Agent 直接调 Figma REST。

阶段 2 与路径无关，只认数据包文件。`schema_version` 当前为 `1.0.0`。

## 1.2 数据包目录

```
{data_dir}/
├── manifest.json
├── design.json
├── index.json
├── comments.json
├── audit.json              ← 阶段 2 必读
├── variables.json
├── screenshot.png
├── assets/ios/…
└── raw/                    ← 阶段 2 不读
```

字段语义节选与完整 schema：`docs/data-package-schema.md`。
常用：`frame.relative` 优先布局；`iconfont`；`_role`；`_layout_hint`。

## 1.3 失败处置

| 失败 | 处置 |
|---|---|
| 无 preload 且无 MCP | 提示安装 / `FIGMA_PRELOAD_DIR` / 自建 MCP；禁止手写 REST |
| URL 缺 `node-id` | 选中 Frame 后 Share → Copy link |
| FigJam / Make | 仅支持 `design` / `file` |
| Token 未配置 / 401 | 改 `config.json` 的 `figma_tokens` |
| 429 | token 池 failover；持续失败换 IP / 加 token |
| 节点不存在 | 检查 `node-id` |

**数据包未成功 → 禁止进入阶段 2。** 用户已给现成 `data_dir` 时可跳过重拉，仍须校验 `manifest.json` + `schema_version`。

## 1.4 诊断纪律

**禁止** CLI 报错后用 LLM「补写」`src/**`。

1. 先：`cd .cursor/figma-ios-preload-data && python3 -m src.cli --self-check-only`
2. 自检通过 → 查 token / 网络 / cwd，**不改源码**
3. 自检失败 → 检查 `.gitignore` 是否误伤 `src/cache/`；从本仓完整拷贝；缺失模块只允许整文件 `cp`，禁止 LLM 重写

---

# 阶段 2：基础 UI 代码生成

## 2.0 必读 audit.json

**阶段 2 第一个必须读的文件**；不读 = 交付不算完成。

| 字段 | 漏读后果（摘要） |
|---|---|
| `non_default_opacity` | 控件过亮过重 |
| `non_solid_fills` / `non_solid_strokes` | 渐变/图填充退化 |
| `with_effects` | 缺阴影 |
| `non_standard_fonts` / `letter_spacing` | 字形/字距错 |
| `list_containers` / `chip_group_hits` | 列表语义丢失 → 须 UICollectionView |
| `sticky_keyword_hits` | 吸顶/吸底错误 |
| `export_asset_with_semantic_children` | 整组切图吃掉子节点 |
| `placeholder_rectangles` | D9D9D9 占位当真色 |

每条 entry 的 `must_apply` / `must_use` 必须落地。  
**对账**：按 `figma-ios-commercial-delivery` 用 `rg` 等对照 `audit.json` / `// Figma node:`（本开源包**不附带** coverage CLI driver）。

## 2.1 唯一数据来源 + bindings

| 信息 | 路径 |
|---|---|
| 高风险清单 | `{data_dir}/audit.json` |
| 节点 | `{data_dir}/design.json` |
| 索引 / 树 | `{data_dir}/index.json` |
| 评论 | `{data_dir}/comments.json` |
| 切图 | `{data_dir}/assets/ios/manifest.json` |
| 警告 | `{data_dir}/manifest.json` |
| 截图 | `{data_dir}/screenshot.png` |
| **宿主定制（必读）** | `.cursor/bindings/host.json` + `color_map.json` / `font_map.json` / `iconfont_map.json` |

- 禁止再调 Figma MCP / REST
- 色/字/icon：**查 bindings**；命中原样输出；未命中用 map 内 `fallback`（UIKit）；禁止臆造未登记的宿主 API
- 阶段 1 **不再**写 `lib_hint` / `objc_property` / `font_call`

## 2.2 生成顺序

```
1. manifest.json → 校验 schema_version == "1.0.0"；读 warnings
2. audit.json
3. .cursor/bindings/（host + maps）
4. index.json → 按需 design.json / comments.json
5. 按下方 skill 树生成 Swift（先 playbook 映射表，再写码）
6. 按 figma-ios-commercial-delivery：rg 对账节点覆盖率 + audit 条目
7. 阶段 4
```

`schema_version` 不一致 → **立即停止**，不要静默适配。

### 宿主基类与前缀（写码前）

探测 `host.bases.view_controller` 的 `open` / `public` / internal（见 `figma-ios-commercial-delivery`）。

类前缀 `<P>` 优先级：

1. 用户消息显式声明  
2. **可选外部 driver** 注入的 `CLASS_PREFIX`（本仓不附带 driver）  
3. `host.class_prefix`（非空）  
4. `host.class_prefix_fallback`

不要强制 `import` 宿主 SDK。`@objcMembers` 仅当 `host.base_is_objc` 或用户要求。

布局 / 交互偏好读 `host.json`：

- `layout_engine`（如 `snapkit`）
- `interaction`（如 `rxswift`）→ **仅当**配置为对应值时才套该 skill；改成其它/清空则不要强行 RxSwift

### Skill 依赖树（与 playbook 对齐）

```
总入口
└─ figma-ios-playbook

基础规范
├─ figma-ios-minimum-deployment-12
└─ figma-ios-commercial-delivery

数据消费
├─ figma-ios-comments-integration
├─ figma-ios-image-assets-download   # 引用规范；下载已由阶段 1 完成
└─ figma-ios-variables-detection

节点识别
├─ figma-ios-to-code-conventions
├─ figma-ios-hierarchy-preservation
├─ figma-ios-iconfont-mapping
├─ figma-ios-navigation              # 按 host.navigation
├─ figma-ios-vector-vs-code
├─ figma-ios-listview-recognition
├─ figma-ios-uicollectionview-codegen  # list 命中后必须加载；勿只识别不写 CV 骨架
├─ figma-ios-standard-components
├─ figma-ios-component-state-recognition
└─ figma-ios-selection-interaction     # UICollectionView didSelectItemAt；禁 UITableView / 禁再调 MCP

布局与样式
├─ figma-ios-vertical-layout-safearea
├─ figma-ios-horizontal-layout
├─ figma-ios-snapkit-layout          # 当 host.layout_engine=snapkit
└─ figma-ios-design-token-mapping

交互
└─ figma-ios-rxswift-interaction-pattern  # 当 host.interaction=rxswift

验收
├─ figma-ios-commercial-delivery     # 覆盖率
└─ figma-ios-screenshot-verification
```

## 2.3 评论

- `confidence=high` → 直接应用  
- `medium` → 应用并在注释标「评论建议」  
- `low` / unmatched → 不应用，列出供人工确认  

细节：`figma-ios-comments-integration`。

## 2.4 警告

`manifest.summary.warnings`：多 Mode → 文件头注释；其它 → `// WARNING: …`。

---

# 阶段 4：视觉还原验收（强制）

1. **节点覆盖率**先于视觉；< 100% 且无豁免 → 回阶段 2（`figma-ios-commercial-delivery`）
2. 读 `{data_dir}/screenshot.png`（禁止再调 MCP）
3. 按 `figma-ios-screenshot-verification` 比对并输出报告

用户写「无需比对」只豁免视觉，**不豁免**覆盖率对账。

---

# 验收标准

**阶段 1**：`data_dir` + `manifest.json`；warnings 已读。  
**阶段 2**：继承 host 基类；色/字/icon 走 bindings；相对布局；iOS 12+；节点注释 `// Figma node: …`；audit 已对账。  
**阶段 4**：覆盖率 100%（或合法三类豁免）+ 视觉报告。  
**可追溯**：manifest 与生成代码一并归档。

---
> Source: [mythkiven/figma-ios-codegen](https://github.com/mythkiven/figma-ios-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
