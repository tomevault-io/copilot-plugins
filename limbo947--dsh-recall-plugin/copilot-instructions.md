## dsh-recall-plugin

> 给 AI 编码代理（和快速上手的人类）的项目速览。读完本文件即可定位任意改动的落点，无需通读源码。

# AGENTS.md

给 AI 编码代理（和快速上手的人类）的项目速览。读完本文件即可定位任意改动的落点，无需通读源码。

## 一句话理解

DSH 消息撤回插件：在用户消息气泡旁加「撤回」按钮，把**项目文件**（独立影子 git 仓库快照）与**对话历史**（官方 sessions.fork）一并回退到该消息发送之前。

## 核心机制（三个关键词）

1. **影子仓库**：每个工作区在 `~/.dsh/dsh-recall-snapshots/<工作区路径SHA256>/git/` 有独立 git 仓库，`--work-tree` 指向项目目录——项目本身零污染（无 .git、无快照文件落地）。home 不可写时降级到项目内 `.dsh-recall-snapshots/`。
2. **tag 即快照**：每条用户消息触发一次 `write-tree + commit-tree + tag snap-<消息ID>`。不建分支、不动工作区文件；消息 ID 就是快照主键，索引丢失可从 tag 名反推重建（`rebuildOrphans`）。
3. **双轨回退**：文件走影子仓库 reset 到 tag；对话走官方 `sessions.fork({ atSeq: cutSeq })`——cutSeq 是该消息之前最近一次 `turn/end` 的 seq。原会话归档（可恢复），新会话继承原标题（不传 `increaseTitle`，避免"标题 2"递增）。

## 文件地图（改动先看这里）

| 文件 | 职责 | 什么时候改它 |
|---|---|---|
| `lib/index.js` | Host 半入口：装配三个域模块、注册 `/api/recall/*` 四端点（init/snapshot-info/preview/execute）、接线 `session/event` 快照触发与启动预热 | 加 API 端点、改事件触发逻辑 |
| `lib/client.js` | Client 半（浏览器）：抢注 `conversation.chat.node` user 渲染槽位（priority -1，冲突递减重试到 -3）、撤回按钮/确认面板/toast、调 fork + 归档 | 改 UI、改 fork 行为 |
| `lib/store.js` | 执行与存储层：`runShell`（统一 `danger-full-access` 宿主身份 + UTF-8 prelude）、root/git 解析、home/降级 store 解析与迁移、`ensureGit` | 改存储布局、shell 执行策略 |
| `lib/snapshots.js` | 快照域：capture/diff/rollback、index.json 落盘与载入、孤儿重建、`resolveCutSeq`（live 内存优先，冷会话走 sessionQuery，结果永久缓存） | 改快照/回退算法 |
| `lib/maintenance.js` | 维护域：定期 `git gc`（每 50 拍或 24h，环境变量 `DSH_RECALL_GC_SNAPS/HOURS` 可调）、会话删除联动清 tag | 改磁盘治理策略 |
| `lib/scripts.pwsh.js` | PowerShell 命令模板（win32） | 改 Windows 命令细节 |
| `lib/scripts.posix.js` | bash 命令模板（linux/darwin），与 pwsh 版**同名导出** | 改 POSIX 命令细节 |
| `cordis.patch.yml` | 持久插件挂载声明（bundle insert 行） | 基本不动 |

**重要约束**：两套脚本模板必须保持同名导出接口——所有调用方统一走 `rt.scripts.*` / `S.*`，按 `process.platform` 在 store.js 单选。

## 关键设计决策（为什么这样写）

- **shell 以宿主身份执行**（`sandboxPolicy: { mode: 'danger-full-access' }`）：受限会话（workspace-write/read-only）写不了 home，read-only 连项目都写不了，回退会直接失败。安全边界靠「命令全是固定模板，唯一变量是插件自己推导的路径，模型无法注入」。
- **串行队列 `state.queue`**：一条消息一次快照，gc/清理排在同队——与快照天然互斥，无 git 锁竞态。
- **幂等与节流**：`ensureGit` 用 `gitReady` Set 跳过重复初始化；home 迁移失败 5 分钟节流（`HOME_RETRY_MS`）；gc 失败也推进时间戳（环境性失败不该堵队列）。
- **win32 索引写入走 base64 分块**（每块 20000 字符）：Windows 命令行 32767 字符上限；POSIX 直接 stdin 写全文。
- **diff 不用 `-z`**：PowerShell 捕获原生命令输出会丢弃含 NUL 的行；改用 `core.quotePath=false` + 逐行按 TAB 解析。
- **pwsh 哈希用 `SHA256::Create()`**：兼容 Windows PowerShell 5.1（无 `Get-FileHash -AsHash` 等新语法依赖）。
- **零构建依赖**：client.js 是纯 JS + `React.createElement`，无打包步骤。

## 数据流速查

```
用户消息 → session/event → captureSnapshot（串行队列）
  → git add -A（exclude.txt 排除 + 超大文件跳过）→ write-tree → commit-tree → tag
点撤回 → preview（diff 当前 vs tag）→ 确认 → execute（文件回退）
  → resolveCutSeq（找 turn/end）→ client 调 sessions.fork → 原会话归档
```

## 存储布局

```
~/.dsh/dsh-recall-snapshots/
├── exclude.txt                    # 用户自定义排除（gitignore 语法，全局共享）
└── <工作区路径SHA256>/
    ├── git/                       # 影子仓库工作目录（空，仅持有 .git）
    │   └── .git/                  # 真实 git-dir（config/info/objects…）
    └── index.json                 # [{id,time,count,sessionId}] 快照索引
```

降级时（home 不可写）：以上结构整体落到 `<项目>/.dsh-recall-snapshots/`，exclude.txt 移入 store 目录内部。

## 开发与验证

```powershell
# 本地源码 → 已安装实例（改完即时生效需重启 dsh-web，Host 半变了光刷页面不够）
Copy-Item .\lib\* "$env:USERPROFILE\.dsh\profiles\web\node_modules\dsh-recall-plugin\lib\" -Force

# 发布流程：bump package.json version → git commit/push → npm publish → GitHub Release
```

- 冒烟路径：中文路径工作区 → 发消息（出快照）→ 改文件 → 撤回（确认面板清单正确、文件恢复、对话回退、标题不变）。
- 回归注意点：每次动 scripts.*.js 必须两个平台过一遍心智检查（路径引号、编码、命令长度上限差异）。
- 版本规范：修复 bump patch，新功能 bump minor；metadata-only 可不发 GitHub Release。

## 已知坑（踩过的，别再踩）

- keyed slot（`conversation.chat.node` key=user）不指定 priority 会因冲突拒载整个插件——必须负值优先级 + 冲突递减重试。
- `node.id` 才是真实消息 ID（快照主键）；`node.key` 是位置键（如 `13:input`），不能用于查询。
- Client 查 `snapshot-info` 前必须等 `ensureInit` 完成回调，否则冷启动误判 `has:false` 且不重试，按钮永不出现。
- fork 不传 `increaseTitle`（2026-08 修复）：否则撤回出的新会话标题变「xxx 2」且多次撤回递增。
- `git init <dir>` 把真实 git-dir 建在 `<dir>/.git`——代码里 repo 与 git 是两个不同路径概念。
- shell 子进程不继承主进程 `DSH_HOME`：POSIX 侧探测为空时回退 Node 主进程 env 再回退 `os.homedir()`。

---
> Source: [limbo947/dsh-recall-plugin](https://github.com/limbo947/dsh-recall-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
