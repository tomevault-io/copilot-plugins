## skills-code-tracer

> Track code changes' impact on business scope through LSP call chain tracing. Use this skill when users ask about code impact analysis, business scope affected by code changes, call chain tracing, "what does this function affect", "which APIs call this code", or need impact reports with git commit references. **IMPORTANT:** Also use this skill when users say "我要提测", "我要投产", "准备提测", "准备投产", "提测分析", "投产分析", or any similar phrases about testing/deploying their current branch. Supports any language with LSP available.


# Code Impact Tracker

追踪代码变更对业务的影响范围，通过 LSP 向上回溯调用链，最终生成业务影响报告。

## 子命令

### 提测 / 投产

当用户说 "我要提测"、"我要投产"、"准备提测"、"准备投产" 等类似表达时，执行以下流程：

**Step 1: 推导主干分支**

自动检测项目的主干分支：

```bash
# 方法 1: 检查 origin/HEAD 指向
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'

# 方法 2: 直接检查常见主干分支名
git branch -a | grep -E 'main|master'
```

推导规则：
1. 如果 `origin/HEAD` 指向某个分支，优先使用该分支
2. 如果没有 `origin/HEAD`，检查是否存在 `master` 或 `main`
3. 默认推荐 `master`（国内项目常见）

**Step 2: 用户确认主干分支**

使用 AskUserQuestion 让用户确认：

```
检测到当前分支: <当前分支名>
推荐的主干分支: <推导出的主干分支>

请确认要对比的主干分支：
1. 使用推荐的 <主干分支>
2. 使用 main 分支
3. 使用 master 分支
4. 手动输入其他分支名、tag 或 commit ID
```

**Step 3: 执行分支对比分析**

确认后，执行分支对比分析（使用 1.2 的流程）：

```bash
git diff <主干分支>..<当前分支> --stat
git diff <主干分支>..<当前分支> --name-only
```

**Step 4: 生成报告**

报告文件名格式：
- 提测: `提测影响报告_<当前分支>_YYYYMMDD.md`
- 投产: `投产影响报告_<当前分支>_YYYYMMDD.md`

报告额外章节：

**提测报告增加**:
```markdown
## 提测检查清单

- [ ] 核心功能已自测通过
- [ ] 边界条件已覆盖测试
- [ ] 新增配置项已同步到测试环境
- [ ] 依赖的外部服务已确认可用

## 测试建议

| 测试类型 | 覆盖范围 | 优先级 |
|---------|---------|--------|
| 功能测试 | <具体功能> | P0 |
| 回归测试 | <受影响的 API> | P1 |
```

**投产报告增加**:
```markdown
## 上线检查清单

- [ ] 测试环境验证通过
- [ ] 配置文件已同步到生产环境
- [ ] 相关人员已通知
- [ ] 回滚方案已准备

## 上线风险

| 风险项 | 影响范围 | 应对措施 |
|-------|---------|---------|
| <风险描述> | <影响模块> | <应对方案> |

## 需要通知的人员

- 运维团队: <具体通知内容>
- 开发团队: <具体通知内容>
```

---

## 核心流程

```
输入起点 → LSP 回溯调用链 → 识别业务入口 → 汇总影响范围 → 生成报告
```

---

## Step 1: 解析输入起点

用户可通过三种方式指定分析起点：

### 1.1 位置格式 (文件：行号)

```
文件路径：行号 [: 列号]
例如：websrv/public_websrv/userinterface/lands.go:52
```

解析步骤：
1. 确认文件存在
2. 确认行号有效（不超过文件行数）
3. 定位到该位置的符号（函数/方法/变量）

### 1.2 Git 提交编号 / 分支对比

```
提交哈希 (完整或简短)
例如：09ac4b9f8 或 09ac4b9

分支对比格式:
git diff <base>..<branch>
例如：git diff master..online/online_260423
```

解析步骤：

**Step 1: 获取变更文件列表**
```bash
git diff <base>..<branch> --stat
git diff <base>..<branch> --name-only
```

**Step 2: 识别变更类型**
对每个变更文件，判断变更类型：
| 变更类型 | 判断方式 | LSP 处理策略 |
|---------|---------|-------------|
| 新增文件 | `git status` 显示 A | 无需回溯 (无调用方) |
| 删除文件 | `git status` 显示 D | 需查找谁调用了被删除的代码 |
| 修改文件 | `git status` 显示 M | **需要 LSP 回溯** |
| 重命名文件 | `git status` 显示 R | 需确认调用点是否受影响 |

**Step 3: 提取变更的函数/方法**
对每个修改的文件：
```bash
git diff <base>..<branch> -- <file>
```
解析 diff 输出，提取：
- 函数定义行变更（`func xxx` 行附近）
- 函数体变更（行号范围）
- 如果是结构体/接口变更，记录受影响的方法

**Step 4: LSP 回溯调用链（核心步骤）**

对每个变更的函数，执行 LSP 回溯：

```
函数变更起点
    ↓ documentSymbol 精确定位函数名
    ↓ prepareCallHierarchy 获取调用层级项
    ↓ incomingCalls 获取第 1 层调用方
    ↓ 对每个调用方重复 incomingCalls
    ↓ 回溯直到业务入口点 (API handler / 配置驱动)
```

**Step 5: 汇总影响范围**

构建影响树：
```
变更文件 1
├── 变更函数 A
│   ├── 影响 API-1: /path/to/api
│   ├── 影响 API-2: /path/to/api
│   └── 影响定时任务：cron_job_xxx
├── 变更函数 B
│   └── ...
└── 变更结构体 C
    └── 影响所有引用该结构体的函数

变更文件 2
└── ...
```

**Step 6: 生成报告**

报告需包含：
1. 变更文件清单
2. 每个变更函数的调用链
3. 受影响的 API 和业务入口
4. 建议的测试覆盖范围

**Step 7: 变更较多时的处理**

如果变更文件超过 10 个或函数超过 20 个：
1. 按模块/目录分组
2. 优先分析核心模块
3. 询问用户是否全部分析或选择特定文件/模块
4. 使用表格汇总，避免报告过长

### 1.3 函数名搜索

```
函数名或方法名
例如：ProjectGroupAdd 或 project_group.GroupAdd
```

解析步骤：
1. 使用 Grep 搜索函数定义位置
2. 若找到多个定义，列出位置供用户选择
3. 确认唯一定义位置后作为起点

---

## Step 2: LSP 环境检测与启动等待

在开始回溯前，**必须**检测 LSP 是否可用，并确保 LSP 已完全启动。

### 2.1 检测方法

根据文件类型判断需要的 LSP：

| 语言 | LSP 服务器 | 检测方式 |
|------|-----------|---------|
| Go | gopls | 检查 `gopls version` |
| Python | pylsp/pyright | 检查 `pylsp --version` 或配置 |
| Java | jdtls | 检查 jdtls 进程 |
| TypeScript/JS | typescript-language-server | 检查 `tsserver` |
| Rust | rust-analyzer | 检查进程或配置 |
| C/C++ | clangd | 检查 `clangd --version` |

### 2.2 LSP 启动等待机制

LSP 服务器启动需要时间，直接调用可能返回 "server is starting" 错误。

**等待流程**：

```
Step 1: 发送 documentSymbol 触发 LSP 启动
    ↓ 如果成功，LSP 已就绪
    ↓ 如果返回 "server is starting" 错误

Step 2: 等待 3-5 秒
    ↓ 重试 documentSymbol

Step 3: 如果仍失败，等待并重试（最多 3 次）
    ↓ 如果 3 次后仍失败，使用 Grep 备选方案
```

**判断 LSP 是否就绪**：
- 成功返回 documentSymbol 结果 → LSP 就绪
- 返回 "server is starting" 或类似错误 → LSP 未就绪，需要等待
- 返回其他错误 → 可能是文件无法解析，需进一步诊断

### 2.3 LSP 不可用时的处理

如果 LSP 不可用：
1. **明确告知用户**：当前环境没有配置该语言的 LSP 或 LSP 启动失败
2. **提供安装指引**：列出常见 LSP 安装方法
3. **询问是否继续**：是否使用备选方案（grep 搜索调用）

备选方案（无 LSP 时）：
- 使用 Grep 搜索函数名调用点
- 精度较低，但可提供基本分析

---

## Step 3: LSP 调用链回溯

### 3.0 位置定位优化

`prepareCallHierarchy` 要求光标定位在函数名上，定位到参数位置会报错 "xxx is not a function"。

**问题场景**：
- 用户指定 `file.go:40` 但未指定列号
- 第 40 行是 `func FeatureSchedule(ctx *fasthttp.RequestCtx)`
- 默认列号 1 定位到 `func` 关键字
- 列号 18 定位到参数 `ctx`，会报错 "ctx is not a function"

**解决方案 - 使用 documentSymbol 精确定位**：

```
Step 1: 对目标文件调用 documentSymbol
    ↓ 获取文件中所有符号的位置信息

Step 2: 根据目标行号匹配对应符号
    ↓ 例如：documentSymbol 返回 "FeatureSchedule (Function) - Line 40"

Step 3: 读取该行内容，计算函数名起始列号
    ↓ 行内容："func FeatureSchedule(ctx *fasthttp.RequestCtx) {"
    ↓ 函数名 "FeatureSchedule" 起始于第 6 列 (func + 空格 = 5 个字符)

Step 4: 使用精确列号调用 prepareCallHierarchy
    ↓ character = 6 (指向函数名第一个字符)
```

**列号计算规则**：

| 语言 | 函数定义格式 | 函数名列号计算 |
|------|-------------|---------------|
| Go | `func Name(params)` | `len("func ") + 1 = 6` |
| Python | `def name(params):` | `len("def ") + 1 = 5` |
| Java | `public void name(params)` | 需解析到方法名位置 |
| TypeScript | `function name(params)` | `len("function ") + 1 = 10` |

**重要提示**：当用户指定 `file:line` 但未指定列号时，**不要**使用默认列号 1，因为：
- 列号 1 通常指向行首空白或 `func`/`def` 关键字
- LSP 的 `prepareCallHierarchy` 要求定位在标识符（函数名）上
- 定位到关键字会返回 "identifier not found" 错误

**推荐做法**：始终使用 documentSymbol 获取符号的精确位置。

### 3.1 回溯算法

使用 LSP 的 `prepareCallHierarchy` → `incomingCalls` 功能：

```
当前层 (起点)
    ↓ prepareCallHierarchy 获取调用层级项
    ↓ incomingCalls 获取所有调用方
第 1 层调用方
    ↓ 对每个调用方重复 prepareCallHierarchy + incomingCalls
第 2 层调用方
    ↓ ...
第 N 层调用方
```

### 3.2 深度控制

- **默认深度**：6 层
- **超出处理**：每满 6 层询问用户：
  ```
  已回溯 6 层，找到 X 个调用方。
  是否继续？
  - 继续回溯 6 层
  - 无限回溯直到终点
  - 停止，当前结果已足够
  ```

### 3.3 回溯数据收集

每层收集以下信息：

```json
{
  "layer": 1,
  "callers": [
    {
      "file": "path/to/file.go",
      "line": 52,
      "character": 10,
      "symbol": "FunctionName",
      "symbol_type": "function|method|variable",
      "container": "ClassName or Package",
      "call_expression": "actualCall()"
    }
  ]
}
```

### 3.4 终止条件

回溯终止于：
- **业务入口点**：被识别为 API handler 或配置驱动的入口
- **无调用方**：LSP 返回空结果
- **外部依赖**：调用来自第三方包（标注为外部依赖）
- **用户停止**：用户选择停止回溯

### 3.5 LSP 错误处理

在回溯过程中可能遇到各种 LSP 错误，需要正确处理：

**常见错误类型**：

| 错误信息 | 原因 | 处理方式 |
|---------|------|---------|
| "server is starting" | LSP 未完全启动 | 等待 3-5 秒后重试 |
| "xxx is not a function" | 光标定位到参数位置 | 使用 documentSymbol 重新定位 |
| "identifier not found" | 定位到关键字或空白 | 使用 documentSymbol 定位函数名 |
| "No hover information" | 位置不在符号上 | 检查位置并调整 |
| "Cannot send notification" | LSP 连接断开 | 重启 LSP 或使用备选方案 |

**错误恢复流程**：

```
遇到 LSP 错误
    ↓ 记录错误类型和位置

Step 1: 判断错误类型
    ↓ "server is starting" → 等待后重试
    ↓ "xxx is not a function" → 重新定位

Step 2: 重试（最多 2 次）
    ↓ 如果成功，继续回溯
    ↓ 如果仍失败，标记该分支为 "LSP 分析失败"

Step 3: 对于失败分支
    ↓ 使用 Grep 搜索函数名作为备选
    ↓ 或标记为 "需要人工确认"
```

**Grep 备选方案**：

当 LSP 无法获取调用方时，使用 Grep 搜索：
```
grep_pattern = "\\.FunctionName\\(|FunctionName\\("
```

注意 Grep 方案的局限：
- 无法区分函数定义和调用
- 可能遗漏通过变量/接口调用的情况
- 无法获取精确的调用位置（行号）

---

## Step 4: 识别业务入口点

业务入口点识别采用**混合模式**：自动识别 + 用户确认

### 4.1 API 入口识别

自动检测以下特征：

| 特征类型 | 检测规则 | 示例 |
|---------|---------|------|
| HTTP Handler | 函数名包含 Handler、Handle | `ProjectGroupHandler` |
| 路由注册 | 文件名含 router/lands/route | `lands.go` |
| 参数特征 | 参数含 context、Request、Response | `func Handler(c *fiber.Ctx)` |
| 注解特征 | 注释含 @api、@endpoint | `// @api POST /path` |
| 框架特征 | 注册到 router 对象 | `router.Post("/path", handler)` |

识别后**询问用户确认**：
```
检测到以下潜在 API 入口：
1. router.Post("/project_group/add", GroupAdd) - 项目组添加接口
2. router.Get("/resource/view", Overview) - 资源总览接口

请确认这些是否为业务 API 入口？如有遗漏请补充。
```

### 4.2 配置文件业务识别

搜索配置文件中的业务定义：

1. **扫描配置文件**：项目根目录下的 `config*.yaml`、`config*.json`
2. **识别业务定义**：
   - 配置项的 key/value 结构
   - 任务名称、服务名称
   - 定时任务配置
3. **追溯配置使用**：
   - 搜索代码中读取配置的位置
   - 将配置读取代码纳入调用链

### 4.3 业务标签整理

最终为每个入口点标注业务标签：

```json
{
  "entry_type": "api|config|schedule|unknown",
  "business_name": "项目组管理",
  "api_path": "/vesta/v2/public/websrv/project_group/add",
  "method": "POST",
  "config_key": "project_group.enable",
  "description": "添加项目组的 API 接口"
}
```

---

## Step 5: 汇总影响范围

### 5.1 影响树构建

将回溯数据构建为影响树：

```
起点函数
├── 影响 API-1: 项目组添加
│   ├── 调用链：Handler → Service → 起点
│   └── 业务影响：用户无法添加项目组
├── 影响 API-2: 项目组列表
│   └── ...
├── 影响配置：project_group.yaml
│   └── ...
```

### 5.2 影响分类

将影响分为：

| 影响级别 | 描述 | 示例 |
|---------|------|------|
| 直接影响 | API/入口直接调用该函数 | 接口返回错误 |
| 间接影响 | 通过中间层调用 | 下游功能受限 |
| 配置影响 | 配置驱动相关逻辑 | 功能开关失效 |
| 外部依赖 | 第三方包调用 | 需关注上游变更 |

---

## Step 6: 生成报告

同时生成 Markdown 和 JSON 格式报告。

### 6.1 报告文件名

根据分析类型选择不同的文件名格式：

**普通分析报告**:
```
impact_report_{identifier}_{date}.md
impact_report_{identifier}_{date}.json

identifier: 函数名、提交哈希简写、或自定义标识
date: YYYYMMDD 格式
```

**提测报告**:
```
提测影响报告_{当前分支}_{date}.md
提测影响报告_{当前分支}_{date}.json
```

**投产报告**:
```
投产影响报告_{当前分支}_{date}.md
投产影响报告_{当前分支}_{date}.json
```

### 6.2 Markdown 报告结构

```markdown
# 代码影响分析报告

## 基本信息
- **分析日期**: YYYY-MM-DD HH:MM:SS
- **分析起点**: 文件路径：行号
- **起点符号**: FunctionName
- **Git 提交**: commit_hash (如适用)
- **回溯深度**: N 层

## 影响摘要
- **影响 API 数量**: X 个
- **影响配置项**: X 个
- **影响业务模块**: X 个
- **外部依赖**: X 个

## 业务影响详情

### API 影响列表

| 序号 | API 路径 | 方法 | 业务名称 | 影响描述 |
|-----|---------|------|---------|---------|
| 1 | /path/to/api | POST | 业务名 | 描述 |

### 配置影响列表

| 序号 | 配置文件 | 配置项 | 业务影响 |
|-----|---------|-------|---------|
| 1 | config.yaml | key.subkey | 描述 |

### 业务模块影响

| 序号 | 模块名 | 影响程度 | 影响范围 |
|-----|-------|---------|---------|
| 1 | 模块 A | 直接 | 功能描述 |

## 调用链详情

### 调用链 1: API → 起点
```
[Layer 0] FunctionName (起点)
    ↑ called by
[Layer 1] CallerFunc (文件：行号)
    ↑ called by
[Layer 2] HandlerFunc (文件：行号)
    ↑ [API 入口] POST /path/to/api
```

## 建议
1. 变更此函数需要关注以下 API 的测试覆盖
2. 建议通知相关业务负责人
3. 配置项变更需同步更新文档

## 附录
- LSP 版本信息
- 分析耗时
- 数据完整性说明
```

### 6.3 JSON 报告结构

```json
{
  "report_meta": {
    "generated_at": "2026-04-07T12:00:00Z",
    "analyzer_version": "1.0",
    "lsp_server": "gopls 0.12.0",
    "language": "go"
  },
  "analysis_source": {
    "type": "location|commit|function",
    "identifier": "commit_hash or function_name",
    "file": "path/to/file",
    "line": 52,
    "symbol": "FunctionName"
  },
  "trace_config": {
    "max_depth": 6,
    "actual_depth": 12,
    "user_interactions": 1
  },
  "impact_summary": {
    "api_count": 3,
    "config_count": 2,
    "module_count": 5,
    "external_count": 1
  },
  "call_chains": [
    {
      "chain_id": 1,
      "layers": [
        {
          "layer": 0,
          "file": "path",
          "line": 52,
          "symbol": "FunctionName"
        },
        {
          "layer": 1,
          "file": "path",
          "line": 100,
          "symbol": "CallerFunc"
        }
      ],
      "entry_point": {
        "type": "api",
        "path": "/api/path",
        "method": "POST",
        "business_name": "业务名"
      }
    }
  ],
  "business_impacts": [
    {
      "entry_type": "api",
      "business_name": "项目组管理",
      "api_path": "/vesta/v2/public/websrv/project_group/add",
      "method": "POST",
      "impact_level": "direct",
      "impact_description": "添加项目组功能将受影响"
    }
  ],
  "recommendations": [
    "变更此函数需要关注项目组管理模块的测试",
    "建议通知项目负责人"
  ]
}
```

---

## 工作目录结构

分析过程中创建以下目录结构：

```
code-impact-tracker-workspace/
├── analysis_input.json      # 用户输入记录
├── trace_data.json          # 回溯中间数据
├── business_entries.json    # 业务入口识别结果
├── impact_report_XXX_YYYYMMDD.md
└── impact_report_XXX_YYYYMMDD.json
```

---

## 使用示例

### 示例 1: 从函数名开始

```
用户：分析 GroupAdd 函数的业务影响

助手:
1. 搜索 GroupAdd 定义位置
2. 确认起点：project_group.go:25
3. 检测 LSP (gopls) 可用
4. 开始 6 层回溯...
   Layer 1: 找到 2 个调用方
   Layer 2: 找到 3 个调用方
   ...
5. 识别业务入口：POST /project_group/add
6. 生成报告
```

### 示例 2: 从 Git 提交开始

```
用户：分析提交 09ac4b9f8 的业务影响

助手:
1. git show 09ac4b9f8 --stat
2. 变更涉及 3 个文件:
   - config.yaml
   - handler.go
   - service.go
3. 提取变更函数：ConfigLoader, HandleRequest, ProcessData
4. 对每个函数进行回溯分析
5. 汇总所有影响
6. 生成报告
```

### 示例 3: 从位置开始

```
用户：分析 websrv/public_websrv/userinterface/lands.go:52 的业务影响

助手:
1. 定位到 router.Post("/project_group/add", ...)
2. 检测 LSP 可用
3. 回溯调用链
4. 识别这是 API 路由注册点
5. 确认业务：项目组添加接口
6. 生成报告
```

### 示例 4: 分支对比分析

```
用户：分析 online/online_260423 分支相对 master 的变更影响

助手:
1. git diff master..online/online_260423 --stat
2. 变更 90+ 文件，按模块分组:
   - utils/alarm/: 告警代理 (新增)
   - websrv/online_inference/defrag/: 碎片整理 (新增)
   - websrv/online_inference/crontab/scale_event.go: 扩缩容告警优化
3. 对每个修改的函数进行 LSP 回溯:
   - SendAlarmToWeChat → GpAlarmProxy.ProcessAlarm
   - 识别调用点：scale_event.go 中的扩缩容监控
4. 汇总影响范围:
   - 影响 API: 扩缩容监控定时任务
   - 影响配置：AlertManager 地址配置
5. 生成报告
```

---

## 注意事项

1. **LSP 是核心依赖**：尽量确保 LSP 可用，否则分析精度下降
2. **交互确认重要**：业务入口识别需要用户确认，避免误判
3. **深度控制灵活**：让用户决定是否深入回溯
4. **报告即时保存**：每完成一部分就保存中间数据，防止中断丢失
5. **跨语言通用**：设计适配多语言，但具体 LSP 调用可能有差异
6. **分支对比分析**：变更较多时按模块分组，优先分析核心模块

---
> Source: [xxsoul/skills-code-tracer](https://github.com/xxsoul/skills-code-tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
