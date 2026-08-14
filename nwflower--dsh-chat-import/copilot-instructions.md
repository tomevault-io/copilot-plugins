## dsh-chat-import

> `dsh-chat-import` 是 DeepSeek Harness 的 Host 插件：把 Claude Code / Codex / ChatGPT 的外部聊天记录 **全保真**导入为**可继续（resume）**的 DSH 会话。DSH 的哲学是 **everything is a plugin**——本仓库只做插件，不碰引擎。改代码前先读 `README.md`（对外契约）与 `test/`（现有行为）。

# AGENTS.md

`dsh-chat-import` 是 DeepSeek Harness 的 Host 插件：把 Claude Code / Codex / ChatGPT 的外部聊天记录 **全保真**导入为**可继续（resume）**的 DSH 会话。DSH 的哲学是 **everything is a plugin**——本仓库只做插件，不碰引擎。改代码前先读 `README.md`（对外契约）与 `test/`（现有行为）。

## 仓库布局：发布面 / 本地工程面

根目录只放发布到 GitHub / npm 的文件；本地工程文件一律收进 `dev/`（gitignore，永不提交）。

```
index.mjs        插件入口（唯一 host 面文件）：注册 import_claude / import_codex / import_chatgpt
convert.mjs      转换核心（纯函数、零 DSH 依赖、可独立单测）
cordis.patch.yml bundle 声明（insert import-claude）
.github/         GitHub Actions CI（npm test，不进 npm 包）
package.json     npm 包元数据；files 白名单 = 发布内容
README.md        对外契约（英文，GitHub/npm 默认）；README.zh-CN.md 中文版——行为变更必须同步两版
LICENSE          MIT
assets/          LOGO（import.svg，README 双语顶部引用，进 npm 包）
test/            单测 + mock ctx 集成测试（进 GitHub，不进 npm 包）
dev/             ❌ 本地工程面：HANDOFF.md、GROWTH.md、脚本（bin/）、夹具、并发协调状态（sessions/）——永不提交
```

- `package.json` 的 `files` 白名单就是 npm 发布面：`index.mjs`、`convert.mjs`、`cordis.patch.yml`、`README.md`、`README.zh-CN.md`、`assets/import.svg`、`LICENSE`。新增被 `index.mjs` import 或 README 引用的文件必须同步加进 `files`。
- **永不提交**：`dev/`、`node_modules/`、`.prev-session*.jsonl`、真实用户 transcript（含敏感内容）、任何凭据/密钥。

## 命令

```sh
npm test        # node --test 跑 test/*.test.mjs（convert 单测 + index mock 集成测试）
node --test "dev/bin/*.test.mjs"   # dev/bin/session.mjs 并发协调工具的自测（本地工程面）
```

无构建步骤：纯 ESM，`index.mjs` / `convert.mjs` 即发布产物。DSH 手工验证：`dsh plugin --profile web add -w link:<本仓库路径>` 后重启 dsh，在会话里调 `import_claude` / `import_codex` / `import_chatgpt`。

## 提交纪律（保持仓库干净）

- **conventional commit 前缀**：`feat:` / `fix:` / `refactor:` / `chore:` / `docs:` / `test:`，中文描述，沿用现有历史风格（如 `feat: batch import (#7) — directory scan, per-file sessions, summary`）。
- **一个逻辑变更一个 commit**：不混改（重构不带新功能，修 bug 不带 docs），不提交 WIP / 中间态。
- **提交前必过**：
  1. `npm test` 全绿；
  2. `git status` 无杂物（`dev/`、`node_modules/`、快照不得出现在待提交里）；
  3. `git diff --cached --check` 无空白错误。
- **行为变更同 commit 更新 README 与测试**：README 是对外契约，测试描述现有行为；改行为必须连测试一起改，并在 commit 信息里说明为什么。
- 提交信息说明「为什么」而非复述代码；指向关联 issue/PR 编号。
- push 前自查：`git log --oneline` 每一条都是一个完整、可读的逻辑单元；工作树干净。
- 重写已推送历史时只用 `--force-with-lease`，远程有变动立即中止——本仓库是单人直推 `main`，尽量不重写。

## 多会话并发开发（并行 Agent 协调）

同一台机器可能并行开多个 Agent 会话操作**同一个工作目录**。会话间靠 `dev/bin/session.mjs`（本地工具，不入库）协调文件占用，避免互相覆盖、避免共享文档（HANDOFF / GROWTH）被并发改写。

| 时机 | 动作 |
| --- | --- |
| 会话开始 / 每个里程碑 | `node dev/bin/session.mjs sync --note "本次要做什么"` |
| 动手改文件之前 | `node dev/bin/session.mjs claim <path>...`（他人活跃占用 → 拒绝，exit 1） |
| 对方占用时 | `status` 看谁在用；等对方 `release`，或对方 **stale**（心跳 2h 过期，`DSH_SESSION_STALE_MS` 可调）后 `--force` 接管 |
| **想改的文件被活跃会话占用，又不愿干等** | `pending <path> <新内容文件>` 把改动写进**临时待合并区**（`dev/sessions/pending/`），不阻塞任何会话 |
| commit + push 之后 | `node dev/bin/session.mjs release`（释放认领）——**解锁时会自动检查待合并区**，提示需要新合并的内容 |
| 解锁后被提示有 pending | `pending apply <path>` 三路合并落盘（无冲突自动合并）；有冲突时手动解决后 `pending drop <path>` 清理 |
| 崩溃 / 中断后恢复 | 先 `status`，必要时 `prune` 清掉 stale 记录 |

规则：

1. **身份**：默认取 `DSH_SESSION_ID`（DSH 注入，每个 Agent 会话天然唯一）；可用 `--as <tag>` 覆盖。身份缺失时 `claim` / `sync` / `release` / `pending`（写/合并/丢弃）直接报错，不凭空建会话。
2. **先 claim 再动手**：要改的文件必须先处于自己名下；他人活跃认领的文件不得修改。`dev/HANDOFF.md`、`dev/GROWTH.md` 等共享文档同样要 claim。
3. **最小认领粒度**：只认领本次要碰的文件；`claim .` 表示整仓库（与一切冲突），仅全局重构用；目录认领覆盖其下所有路径。
4. **stale 接管**：`--force` 只能接管 stale 会话的认领，永远抢不了活跃会话的文件；被接管者丢的只是认领记录，文件内容不受影响。
5. **push 前 `git pull --rebase origin main`**：小步提交（一个逻辑变更一个 commit）可把 rebase 冲突降到最低。本协议覆盖同一工作目录的并行会话；跨机器并行靠 git 纪律，registry 不跨机器同步。
6. **状态位置**：全部运行时状态在 `dev/sessions/`（gitignore，永不提交、不进 npm 包）；损坏时删目录重建即可，不影响仓库。
7. **pending 待合并区（简单会话的异步写作）**：`pending <path> <内容文件>` 把「改好的新内容」连同写入时 git HEAD 版本（base）存进 `dev/sessions/pending/<路径>/`；`release`（解锁）会自动检查并提示待合并内容；`pending apply` 做三路合并（current × base × pending，`git merge-file`），**无冲突自动落盘并清除条目，有冲突写入冲突标记、保留条目**；`base` 缺失时拒绝盲合。`apply` 要求路径无活跃占用（防止与在改会话打架）。

命令速查：`sync`（登记 + 心跳）`claim`（独占认领）`release`（释放 + 解锁检查）`status`（总览 + 待合并区）`who <path>`（谁占用某文件）`prune`（清 stale）`drop <tag>`（移除会话）`new`（生成 tag）`pending <path> <内容文件>`（写入待合并区）`pending list|show|apply|drop <path>`（查看/合并/丢弃）。

## DSH 插件约束

- **只消费 host 公开服务**：`sessionPersistence`（create + append）、`fs`、`tools`、`workspaceRegistry`。不发布服务 → 无需 isolate realm；无 Browser 侧。
- **插件，不是引擎改动**：新行为走公开扩展点（工具注册）；绝不修改 DSH 引擎 / apiproxy / 官方 UI 包。
- **会话日志 append-only、deep-frozen**：只 `create` + `append`，绝不改写历史事件。
- **模型可见 ⟺ 落盘**：进入模型上下文的任何内容必须能从会话日志重建；新模型可见输入必须对应会话事件。
- **事件纪律**：`seq` 从 0 连续；surface 事件（`user/message` / `assistant/message` / `tool/result`）必须带 `surfaceOp: 'append'`；`tool/result` 用 `sourceEventSeqs` 关联其 `tool/call`；`SessionHeader` version 保持 `0`，只做结构性变更才 bump。
- **幂等**：目标会话已存在时跳过（`sessionPersistence.list()` 判重），不重复写入。
- **归组**：`workspaceRegistry.resolveByPath(cwd)` → `workspace.attachSession(id)`，否则会话显示「未分组」。
- **失败要大声**：畸形 JSONL 行计数上报（`skipped`），绝不静默吞掉；读取工作区外的 transcript 需会话沙箱允许。

## 质量约定

- 文件以**恰好一个**换行结尾；空 `catch` 必须说明吞掉什么且 `try` 只包一条语句；不注释代码里显而易见的事实。
- 保持 `convert.mjs` 零依赖纯函数：任何 DSH 依赖只允许出现在 `index.mjs`。
- 测试描述行为而非背书正确性；fixtures 用合成数据，永不掺真实 transcript。
- 不写行内文档废话：注释写契约与上下文，不叙述控制流。

## 编辑本文件

规则保持自包含；改完须与仓库现状一致（目录、命令、约束过时了要同步更新）。

---
> Source: [Nwflower/dsh-chat-import](https://github.com/Nwflower/dsh-chat-import) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
