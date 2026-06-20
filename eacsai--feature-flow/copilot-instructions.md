## feature-flow

> 新项目 bootstrap — 一次性配置项目变量 / 写 sync+approve script（远程项目）/ 跑 /init 扫项目结构 / init 3 个 session snapshot / 加 MEMORY.md 索引 / 提醒开 3 个 claude tab (writer/explainer/data)。**只在开新项目时跑一次**，跑完项目装好。后面日常开发（开 phase branch / codex review loop / cherry-pick / squash-rebase merge）按已有 memory 规则自动走，不在本 skill 里。用法 `/feature-flow`。


# feature-flow — 新项目 bootstrap

**只在一个新项目刚开始 / 还没装过本 workflow 时跑一次。** 跑完项目变量都填好、helper script 就位（远程项目）、`/init` 扫好项目结构、3 份 session snapshot 初始化好、MEMORY.md 索引齐、3 tab 配置说明也告诉用户了 — 之后用户日常开 phase / commit / merge 直接按 memory 规则走（见 [Bootstrap 完成后的日常开发](#bootstrap-完成后的日常开发)）。

> **不是**每开一个 phase 都跑一次。每开 phase 直接走日常流程即可。

## 已 bootstrap 检测（每次启动 skill 先跑）

skill 一启动**先检测**当前项目是否已 bootstrap：

```bash
PROJECT_MEMORY_DIR=<推断>
PROJECT_ROOT=<推断>
DEV_HOST=<已知或问>

ok=true
for f in project_writer_handoff.md project_data_handoff.md project_explainer_handoff.md; do
  test -f "$PROJECT_MEMORY_DIR/$f" || ok=false
done
for key in project_writer_handoff project_data_handoff project_explainer_handoff feedback_three_claude_roles; do
  grep -q "$key" "$PROJECT_MEMORY_DIR/MEMORY.md" || ok=false
done
test -f "$PROJECT_ROOT/CLAUDE.md" || ok=false
# 远程项目还要求 helper 存在；本地项目不要求 helper
if [ "$DEV_HOST" != "local" ]; then
  ls "$PROJECT_ROOT"/sync-*-review.sh >/dev/null 2>&1 || ok=false
  ls "$PROJECT_ROOT"/approve-*-diff.sh >/dev/null 2>&1 || ok=false
fi
```

如果 `ok=true` → 项目已 bootstrap。**回一句**"项目已 bootstrap 过，没事可做。日常开发请按下方 §Bootstrap 完成后的日常开发 那张表走"并退出。**不要**重做 step。

---

## Bootstrap 步骤（按顺序跑）

### Step 1 — 收集项目变量

下面这些值要在 bootstrap 时全部确定。能从环境推断的优先推断，剩下问用户。**注意 `PROJECT_NAME` / `REPO_NAME` / `REPO_SLUG` 是三个独立 var**（一个项目可能 workspace 叫 `MyResearch`，里面 repo 是 `CoolModel`，路径里又要小写 `coolmodel`）。

| 变量 | 例 (illustrative) | 推断方式 |
|---|---|---|
| `${PROJECT_ROOT}` | `~/Documents/your-project` | `pwd`（或问用户） |
| `${PROJECT_NAME}` | `your-project` | local workspace name；`basename ${PROJECT_ROOT}` |
| `${DEV_HOST}` | `your-dev-host` 或字面值 `local` | 询问；如本地开发填 `local` |
| `${DEV_REPO}` | `/path/on/remote/your-repo` | 询问（`DEV_HOST=local` 时与 `${PROJECT_ROOT}` 可能不同 — 多 repo 的情况） |
| `${REPO_NAME}` | `your-repo` | repo 真名；`basename ${DEV_REPO}` |
| `${REPO_SLUG}` | `your-repo` | lowercase shell-safe slug；`echo "$REPO_NAME" \| tr 'A-Z' 'a-z'` |
| `${FORK_REMOTE}` | `origin` 或 `myfork` | `git remote -v` 拿候选名让用户选 |
| `${TARGET_BRANCH}` | `main` 或 `dev` | 询问主集成 branch；典型 `main` / `master` / `dev` / `<proj>_dev` |
| `${GIT_NAME}` / `${GIT_EMAIL}` | `your-name` / `you@example.com` | 读 memory `reference_git_identity*`；没有问用户一次然后存进 memory |
| `${PROJECT_MEMORY_DIR}` | `~/.claude/projects/<your-project-slug>/memory` | 按 cwd 推断 claude project 目录 |
| `${HANDOFFS_DIR}` | `~/.claude/projects/<your-project-slug>/handoffs` | 同上；bootstrap 不用，但日常 codex review 必用，存这里方便后续 |

> 用 `AskUserQuestion` 一次性问完缺的几项，不要每个变量单独问。

### Step 2 — 远程 SSH setup + reachable check（仅 `DEV_HOST != local`）

#### 2a. Reachable check

```bash
ssh -o BatchMode=yes "$DEV_HOST" 'echo OK && hostname && whoami'
```

- 成功（打印 `OK <hostname> <user>`）→ 跳过 2b，进 Step 3
- 失败 → 进 2b 走 walkthrough

> `BatchMode=yes` 强制禁 interactive prompt（密码 / passphrase / host-key 都不弹）→ 没 key 直接 exit non-zero。

#### 2b. SSH setup walkthrough（仅 2a 失败时）

⚠️ **claude 不能替用户输密码**。下面命令必须用户**自己在 prompt 输** `! <command>` 跑（`!` 前缀让 claude code 跳出主对话执行 interactive shell，密码 / passphrase 才能进得去）。

**Step 2b-1: 收集真实远程账户信息**

用 `AskUserQuestion` 问用户两项（如果用户没说，默认 `DEV_HOST` 当 alias name）：

| 问题 | 例 |
|---|---|
| `${REMOTE_USER_AT_HOST}` | `you@host.example.com` (真 user@host, 不是 alias) |
| `${SSH_KEY_PATH}` | `~/.ssh/id_ed25519_${DEV_HOST}` (默认 ed25519 + alias 为后缀) |

**Step 2b-2: 检查 / 生成 ssh key**

```bash
test -f "$SSH_KEY_PATH" && echo "key 已存在" || echo "缺 key，让用户生成"
```

缺 key → 告诉用户在 prompt 跑（**用户自己跑**）：

> ```
> ! ssh-keygen -t ed25519 -f ${SSH_KEY_PATH} -C "${DEV_HOST}-key"
> ```
>
> （keygen 会问 passphrase；不想要每次输 → 直接回车留空。设了 passphrase 后续要 `ssh-add ${SSH_KEY_PATH}` 装进 agent 才免密。）

**Step 2b-3: 推 public key 到远程**

```
! ssh-copy-id -i ${SSH_KEY_PATH}.pub -o StrictHostKeyChecking=accept-new ${REMOTE_USER_AT_HOST}
```

- **这一步 ssh 会问一次远程账户密码** — 用户自己输。输完就免密了。
- `StrictHostKeyChecking=accept-new` 自动 trust 第一次见的 host key（CI 友好 + 用户少一次 prompt）。

**Step 2b-4: 写 `~/.ssh/config`**

`Write` 工具追加 / 创建 `~/.ssh/config`（先 `grep -q "^Host ${DEV_HOST}\b"` 检查是否已有同名 alias；有就跳过，避免重复）：

```sshconfig
Host ${DEV_HOST}
  HostName <REMOTE_USER_AT_HOST 的 host 部分>
  User <REMOTE_USER_AT_HOST 的 user 部分>
  IdentityFile ${SSH_KEY_PATH}
  ServerAliveInterval 60
  ControlMaster auto
  ControlPersist 4h
  ControlPath ~/.ssh/cm-%r@%h:%p.sock
```

> `ControlMaster + ControlPersist` 启用连接复用，后续 ssh / scp 共享同一条 master，第一次握手后 4 小时内的命令都秒级。

**Step 2b-5: 回到 2a 重测**

走 2a `ssh -o BatchMode=yes "$DEV_HOST" 'echo OK'`。还失败 → print 错误明细给用户（typo / 路径 / 防火墙 / 远程 sshd 配置），**停止 bootstrap**。仍成功不下去就别强推 step。

### Step 3 — 写 sync + approve helper script（仅 `DEV_HOST != local`）

> ⚠️ **写文件时必须把所有 `${...}` 替换成本项目的 concrete literal values**（如 `your-dev-host` / `/path/on/remote/your-repo` / `your-repo` / `your-repo` 等）。生成后的脚本**不依赖外部环境变量**，独立可执行。下面是模板，写文件前替换。

写入两个文件并 `chmod +x`：

#### `${PROJECT_ROOT}/sync-${REPO_SLUG}-review.sh`（模板，写入时 substitute concrete values）

```bash
#!/usr/bin/env bash
# Sync remote dev tree to local read-only review snapshot (for codex rescue).
set -euo pipefail

REPO="<concrete PROJECT_ROOT>/codex-review-<concrete REPO_SLUG>/<concrete REPO_NAME>"
REMOTE_HOST="<concrete DEV_HOST>"
REMOTE_PATH="<concrete DEV_REPO>"

SHA=$(ssh "$REMOTE_HOST" "cd $REMOTE_PATH && git rev-parse --short HEAD")
echo "[sync] base = $SHA"

rm -rf "$REPO"
mkdir -p "$REPO"

ssh "$REMOTE_HOST" "cd $REMOTE_PATH && git archive --format=tar HEAD" | tar -x -C "$REPO"
git -C "$REPO" init -q
git -C "$REPO" add -A
git -C "$REPO" -c user.email=tmp@local -c user.name=tmp commit -q -m "base: $SHA"

rsync -aq \
  --exclude='.git/' --exclude='.venv/' --exclude='__pycache__/' --exclude='*.pyc' \
  "$REMOTE_HOST:$REMOTE_PATH/" "$REPO/"

# 记 BASE_SHA 给 approve 校验 (race protection)
printf '%s\n' "$SHA" > "$REPO/.codex_base_sha"

echo "[sync] done. working changes vs base $SHA:"
git -C "$REPO" status --short
```

#### `${PROJECT_ROOT}/approve-${REPO_SLUG}-diff.sh`（模板，写入时 substitute concrete values）

```bash
#!/usr/bin/env bash
# Push codex-edited (or user-edited) local snapshot diffs to remote dev tree.
# Paired with sync-*-review.sh. Does NOT commit/push on remote.
set -euo pipefail

REPO="<concrete PROJECT_ROOT>/codex-review-<concrete REPO_SLUG>/<concrete REPO_NAME>"
REMOTE_HOST="<concrete DEV_HOST>"
REMOTE_PATH="<concrete DEV_REPO>"

cd "$REPO"

# Race protection: 远程 HEAD 必须仍是 sync 时记下的 BASE_SHA
test -f "$REPO/.codex_base_sha" || { echo "[approve] no .codex_base_sha — 先跑 sync"; exit 1; }
BASE_SHA=$(cat "$REPO/.codex_base_sha")
CURRENT_SHA=$(ssh "$REMOTE_HOST" "cd $REMOTE_PATH && git rev-parse --short HEAD")
if [ "$CURRENT_SHA" != "$BASE_SHA" ]; then
  echo "[approve] remote HEAD changed: $BASE_SHA -> $CURRENT_SHA; 先 re-sync 再 approve"
  exit 1
fi

modified=()
deleted=()
while IFS= read -r line; do [ -n "$line" ] && modified+=("$line"); done < <(git diff --name-only --diff-filter=AM HEAD)
while IFS= read -r line; do [ -n "$line" ] && deleted+=("$line");  done < <(git diff --name-only --diff-filter=D  HEAD)

if (( ${#modified[@]} == 0 && ${#deleted[@]} == 0 )); then
  echo "[approve] no diffs vs base — nothing to push"
  exit 0
fi

# 额外检查：要被覆盖的远程 working tree 干净（防 peer 同时改了文件）
DIRTY=$(ssh "$REMOTE_HOST" "cd $REMOTE_PATH && git status --short -- ${modified[*]} ${deleted[*]}")
if [ -n "$DIRTY" ]; then
  echo "[approve] remote working tree dirty on target files:"
  echo "$DIRTY"
  exit 1
fi

echo "[approve] will push to $REMOTE_HOST:$REMOTE_PATH:"
for f in "${modified[@]}"; do echo "  M $f"; done
for f in "${deleted[@]}";  do echo "  D $f"; done
read -p "[approve] proceed? (y/N) " ans
[[ "$ans" == "y" || "$ans" == "Y" ]] || { echo "[approve] aborted"; exit 1; }

for f in "${modified[@]}"; do
  rsync -aR "$f" "$REMOTE_HOST:$REMOTE_PATH/"
done

if (( ${#deleted[@]} > 0 )); then
  printf -v rmlist '%q ' "${deleted[@]}"
  ssh "$REMOTE_HOST" "cd $REMOTE_PATH && rm -f $rmlist"
fi

echo "[approve] done. now on $REMOTE_HOST:"
ssh "$REMOTE_HOST" "cd $REMOTE_PATH && git status --short"
echo "[approve] next: ssh $REMOTE_HOST 'cd $REMOTE_PATH && git add ... && git commit -m ... && git push'"
```

> 写之前先 `ls` 检查是否已存在；存在则跳过、提示用户文件已在。

### Step 4 — 初次 sync + `/init` 扫远程项目结构

让 claude 在 bootstrap 时就理解项目长啥样、有哪些重要文件/目录，写进项目根级 `${PROJECT_ROOT}/CLAUDE.md`（后续 session 进 cwd 时自动 load）。

#### 4a. 拉远程代码到本地（仅 `DEV_HOST != local`）

```bash
bash "$PROJECT_ROOT/sync-${REPO_SLUG}-review.sh"
# 远程整个 working tree → $PROJECT_ROOT/codex-review-${REPO_SLUG}/${REPO_NAME}/ (read-only snapshot)
```

失败 → 报告用户、停止；不要进入 4b。

#### 4b. 调 `/init` 扫描代码结构

确定 `INIT_CWD`（`/init` 跑的位置）：

- **远程项目** (`DEV_HOST != local`)：`INIT_CWD = $PROJECT_ROOT/codex-review-${REPO_SLUG}/${REPO_NAME}`
- **本地项目** (`DEV_HOST=local`)：`INIT_CWD = $DEV_REPO`

```bash
cd "$INIT_CWD"
# 然后调 Skill(skill="init") — 它会扫整个 codebase 生成 CLAUDE.md 写到 $INIT_CWD/CLAUDE.md
```

`/init` 失败 → 报告用户，**不要**继续到 4c 或 Step 5；让用户手动 cd 到 `$INIT_CWD` 重跑 `Skill(skill="init")`。

#### 4c. 把生成的 CLAUDE.md 搬到项目根（带 guard）

`/init` 把 CLAUDE.md 写到 `$INIT_CWD/CLAUDE.md`。想搬一份到 `$PROJECT_ROOT/CLAUDE.md`，这样下次任何 claude session cd 进项目根都自动 load。**但**项目根可能已有 CLAUDE.md（用户手写）— 必须 guard：

```bash
SRC_CLAUDE="$INIT_CWD/CLAUDE.md"
DST_CLAUDE="$PROJECT_ROOT/CLAUDE.md"

test -f "$SRC_CLAUDE" || { echo "[init] /init 没生成 $SRC_CLAUDE — 重跑 4b"; exit 1; }

if [ -e "$DST_CLAUDE" ]; then
  if [ "$(realpath "$SRC_CLAUDE")" = "$(realpath "$DST_CLAUDE")" ]; then
    echo "[init] DST 已是 SRC (本地项目 INIT_CWD == PROJECT_ROOT)，无需搬"
  else
    echo "[init] $DST_CLAUDE 已存在，不覆盖；用户可比对 $SRC_CLAUDE 决定要不要 merge"
  fi
else
  cp "$SRC_CLAUDE" "$DST_CLAUDE"
  echo "[init] $DST_CLAUDE 已就位"
fi
```

#### 4d. Partial-recovery 速查

| 失败位置 | 怎么恢复 |
|---|---|
| 4a sync 失败 | 修网络 / ssh / 远程 disk space，重跑 `bash $PROJECT_ROOT/sync-${REPO_SLUG}-review.sh` |
| 4b /init 失败 | 不重写 helper；`cd $INIT_CWD && Skill(skill="init")` 直接重跑 |
| 4c cp 失败 / 已存在 | 单独跑 4c 那段 shell；用户自决合并 |
| Step 4 整体没通过 sanity（Step 7）| **不要**进 Step 5–9，先把 Step 4 修通 |

### Step 5 — Init 3 份 session snapshot

`${PROJECT_MEMORY_DIR}/` 下若任何一个缺，按下面模板补全。

| 文件 | 写者 |
|---|---|
| `project_writer_handoff.md` | Writer |
| `project_data_handoff.md` | Data |
| `project_explainer_handoff.md` | Explainer |

模板（替换 `<Role>`）：

```markdown
---
name: <role>-handoff
description: <Role> session live snapshot — 最新状态 / peer 不要碰啥
metadata:
  type: project
---

# <Role> session — (uninitialized)

## 当前焦点
当前无 <role> session 在跑。下次 session 启动后会覆盖此模板。

## In-flight files / branch
无

## 刚完成（peer 应该 reload）
无

## ⛔ peer 不要碰
无

## ✅ peer 可以帮看的
无
```

### Step 6 — Upsert MEMORY.md 索引

读 `${PROJECT_MEMORY_DIR}/MEMORY.md`，对下列每个 key 做 **upsert**（不存在则追加；存在但文案过时则替换 — 比如旧 "Stage 0" / "Tab 2 (reader)" 应改成 "Bootstrap Step 5/8" / "Tab 2 (explainer)"）：

```
- [Writer session handoff snapshot](project_writer_handoff.md) — Tab 1 (writer) 实时状态；writer 完成 logical 工作后刷新
- [Data session handoff snapshot](project_data_handoff.md) — Tab 3 (data) 实时状态；feature-flow Bootstrap Step 5 init，data session 每完成 fix/check 刷新
- [Explainer session handoff snapshot](project_explainer_handoff.md) — Tab 2 (explainer) 实时状态；feature-flow Bootstrap Step 5 init，explainer 每出新笔记/图刷新
- [3-claude roles](feedback_three_claude_roles.md) — writer/explainer/data 权限边界 + 触发词 + role confirm 规则；first-message 模板在 feature-flow Bootstrap Step 8
```

**Upsert 步骤**：对每行 key (文件名 stem)，`grep -n "$key" MEMORY.md`；找到 → 删除整行 + 追加新行；找不到 → 直接追加。

### Step 7 — 跑 dry-run sanity check

确保 bootstrap 真装好。**必须设全所有变量后再跑**：

```bash
PROJECT_MEMORY_DIR=<填入>
PROJECT_ROOT=<填入>
PROJECT_NAME=<填入>
REPO_NAME=<填入>
REPO_SLUG=<填入>
DEV_HOST=<填入>
DEV_REPO=<填入>
TARGET_BRANCH=<填入>
FORK_REMOTE=<填入>

echo "=== required vars non-empty ==="
for v in PROJECT_MEMORY_DIR PROJECT_ROOT PROJECT_NAME REPO_NAME REPO_SLUG DEV_HOST DEV_REPO TARGET_BRANCH FORK_REMOTE; do
  val=$(eval "echo \$$v")
  [ -n "$val" ] && echo "OK: $v=$val" || echo "MISSING: $v"
done

echo "=== PROJECT_MEMORY_DIR writable ==="
test -w "$PROJECT_MEMORY_DIR" && echo "OK" || echo "NOT WRITABLE: $PROJECT_MEMORY_DIR"

echo "=== 3 snapshots present ==="
for f in project_writer_handoff.md project_data_handoff.md project_explainer_handoff.md; do
  test -f "$PROJECT_MEMORY_DIR/$f" && echo "OK: $f" || echo "MISSING: $f"
done

echo "=== MEMORY.md indices present ==="
for key in project_writer_handoff project_data_handoff project_explainer_handoff feedback_three_claude_roles; do
  grep -q "$key" "$PROJECT_MEMORY_DIR/MEMORY.md" && echo "OK: $key" || echo "MISSING in MEMORY.md: $key"
done

echo "=== helper scripts present (remote project only) ==="
if [ "$DEV_HOST" != "local" ]; then
  for s in "sync-${REPO_SLUG}-review.sh" "approve-${REPO_SLUG}-diff.sh"; do
    test -x "$PROJECT_ROOT/$s" && echo "OK: $s" || echo "MISSING or not +x: $s"
  done
fi

echo "=== /init source CLAUDE.md generated ==="
if [ "$DEV_HOST" = "local" ]; then
  INIT_CWD="$DEV_REPO"
else
  INIT_CWD="$PROJECT_ROOT/codex-review-${REPO_SLUG}/${REPO_NAME}"
fi
test -f "$INIT_CWD/CLAUDE.md" && echo "OK: $INIT_CWD/CLAUDE.md" || echo "MISSING: $INIT_CWD/CLAUDE.md — 跑 Step 4b"

echo "=== project-level CLAUDE.md present ==="
test -f "$PROJECT_ROOT/CLAUDE.md" && echo "OK: $PROJECT_ROOT/CLAUDE.md" || echo "MISSING: $PROJECT_ROOT/CLAUDE.md — 跑 Step 4c"

echo "=== git identity available ==="
[ -n "$GIT_NAME" ] && [ -n "$GIT_EMAIL" ] && echo "OK: $GIT_NAME <$GIT_EMAIL>" || echo "MISSING: GIT_NAME/GIT_EMAIL"

echo "=== remote reachable (remote project only) ==="
if [ "$DEV_HOST" != "local" ]; then
  ssh -o BatchMode=yes "$DEV_HOST" 'echo OK' 2>&1
fi

echo "=== fork remote configured ==="
if [ "$DEV_HOST" = "local" ]; then
  ( cd "$DEV_REPO" && git remote -v | grep -q "$FORK_REMOTE" ) && echo "OK" || echo "FORK_REMOTE not set"
else
  ssh "$DEV_HOST" "cd $DEV_REPO && git remote -v | grep -q $FORK_REMOTE" && echo "OK" || echo "FORK_REMOTE not set"
fi
```

任何 `MISSING` / `NOT WRITABLE` / failed line → 回去修对应 step；**不通过别进 Step 8**。

### Step 8 — 告诉用户怎么开 3 个 claude tab

⚠️ 角色靠 first-message 字符串触发；用户敲错 → session 会默默以 writer 行为运行。Tab 2/3 的 first response **必须**显式 confirm 角色（已写进 [[feedback_three_claude_roles]]）。

#### Tab 1 (Writer) — 不用做特别配置

当前 session 就是 writer。

#### Tab 2 (Explainer) 开法

> 1. 新终端：
>    ```
>    cd ${PROJECT_ROOT} && claude
>    ```
> 2. 第一句话粘这个：
>    ```
>    explainer mode — 我是 read-only 解释员，不准 Edit / Write 任何代码文件
>    (.py / .yaml / .json / .sh / .ts / .js / .cpp / .h 等；.md / .txt / draw.io
>    可以写). 可以画 draw.io 流程图、写 markdown 总结、跑 read-only bash 命令、
>    web search. 不能 ssh 改远程，不能跑训练 / eval / git push.
>
>    First response 必须显式 confirm "Active role: Explainer. I will not
>    Edit/Write code files." 然后再做任何事.
>
>    先看 ${PROJECT_MEMORY_DIR}/project_writer_handoff.md +
>    project_data_handoff.md 知道两位在干啥.
>
>    Trigger words:
>    - "刷 memory" → 重读 MEMORY.md + 3 份 *_handoff.md
>    - "通知 peer" → 写 project_explainer_handoff.md 并告诉我切去别的 tab
>    ```

#### Tab 3 (Data) 开法

> 1. 新终端：
>    ```
>    cd ${PROJECT_ROOT} && claude
>    ```
> 2. 第一句话粘这个：
>    ```
>    data mode — 我只碰数据相关代码：data prep / dataset loader / parquet+mp4+meta
>    完整性 + 正确性检查、polarity / convention / camera mismatch 等 silent bug.
>    不准改训练 loop / model 定义 / loss / config schema.
>    可以 ssh 改远程，但 scope 限定数据.
>
>    First response 必须显式 confirm "Active role: Data. I will not modify
>    training loop / model / loss / config schema." 然后再做任何事.
>
>    先看 ${PROJECT_MEMORY_DIR}/project_writer_handoff.md 了解 writer 当前
>    branch / phase.
>
>    Trigger words:
>    - "刷 memory" → 重读 MEMORY.md + 3 份 *_handoff.md
>    - "通知 peer" → 写 project_data_handoff.md 并告诉我切去别的 tab
>    每完成一个数据 fix / check 主动调 "通知 peer".
>    ```

> 只开 2 个 tab 也行（按用户需要省略 explainer 或 data）。

### Step 9 — 收尾

打印一段 summary 给用户（**wording 按 `DEV_HOST=local` vs 远程 conditional**）：

**远程项目** (`DEV_HOST != local`):

> ✅ **项目 `${PROJECT_NAME}` bootstrap 完成**。
>
> - 项目变量已 resolve；远程项目的 helper 脚本已就位（`sync-${REPO_SLUG}-review.sh`、`approve-${REPO_SLUG}-diff.sh`）
> - `/init` 已扫远程项目结构，`${PROJECT_ROOT}/CLAUDE.md` 已生成
> - 3 份 session snapshot 已 init，MEMORY.md 索引齐
> - 3 tab 开法 + first-message 模板见上
>
> **下一步**：直接走日常开发。开 phase / commit / merge 按已有 memory 规则自动走，本 skill 不用再调。日常流程概览见下方 §Bootstrap 完成后的日常开发。

**本地项目** (`DEV_HOST=local`):

> ✅ **项目 `${PROJECT_NAME}` bootstrap 完成**。
>
> - 项目变量已 resolve；本地项目无需 helper 脚本（直接在 `${DEV_REPO}` 改）
> - `/init` 已扫项目结构，`${PROJECT_ROOT}/CLAUDE.md` 已生成
> - 3 份 session snapshot 已 init，MEMORY.md 索引齐
> - 3 tab 开法 + first-message 模板见上
>
> **下一步**：直接走日常开发；本 skill 不用再调。日常流程概览见下方 §Bootstrap 完成后的日常开发。

---

## Bootstrap 完成后的日常开发

> ℹ️ **注意**：下面的 `[[feedback_*]]` 是 Claude project memory wikilink names，不是 GitHub 链接 — 在 GitHub 上它们会显示成 literal text。本 public repo 只 ship 了 `feedback_three_claude_roles.md`；下表里其他 memory rule 需要每个项目 / 用户自己提供（如果还没有，把表当 checklist 自己写等效规则再依赖日常自动化）。

本 skill **不再** codify 日常开发的 stage 流程；按 memory 规则自动走。**Claude 在日常开发时遇到下面情况各自参考对应 memory**：

| 用户做什么 | 该按哪条 memory 走 |
|---|---|
| 开 feature/phase branch、commit、push | [[feedback_plan_before_code]]、[[feedback_parallel_ssh]] |
| 改完代码、判断要不要 codex review | [[feedback_auto_codex_after_changes]]（rescue 默认 + mode 选择） |
| 跑 codex review | [[feedback_codex_review_with_handoff]]（handoff 决策）+ [[feedback_codex_review_autosync]]（先 sync 快照）+ [[feedback_codex_review_deep]]（5 dim）+ [[feedback_codex_review_actually_run]]（不要积压） |
| Cherry-pick / merge feature branch 到主集成 branch | [[feedback_prefer_rebase_over_merge]]（squash-rebase + `--ff-only`，禁 `merge --no-ff`） |
| 任何 logical 工作完成后 | 刷 own `project_<role>_handoff.md` + 用 "通知 peer" 触发词 |
| 长任务 / 训练前后、phase 节点 | 立刻刷 `project_active_session.md`（[[feedback_live_save_memory]]） |
| 任何 session 启动时 | 先读 MEMORY.md + 3 份 handoff snapshot；角色 session first response 必须显式 confirm 角色（[[feedback_three_claude_roles]]） |

---

## 行为约束

- **只在新项目跑一次**；启动 skill 先跑顶部"已 bootstrap 检测"，齐则 no-op
- **变量缺失就问，不要瞎猜**：用 `AskUserQuestion` 一次性问完缺的几项
- **远程 reachable 失败立刻停止**：让用户配 ssh 再调
- **不要写多余 stage**：日常 phase loop / merge 等不在本 skill 内
- **Helper script 写文件时必须 substitute concrete values**：不要留 `${...}` 字面，否则脚本依赖外部 env 跑不起
- **`ssh local '...'` 是无效命令**：`DEV_HOST=local` 时跳过远程相关 step (Step 2 reachable / Step 3 helper / Step 4a sync) + Step 7 sanity 远程 check
- **`PROJECT_NAME` ≠ `REPO_NAME` ≠ `REPO_SLUG`**：三个独立 var；workspace 名 / repo 真名 / 小写 slug 各有用途

## 关联

- [[feedback_three_claude_roles]] — 角色权限边界 + role confirm + 触发词
- [[feedback_prefer_rebase_over_merge]] — 日常 merge 流程（squash-rebase + ff）
- [[feedback_codex_review_with_handoff]] — 日常 codex review 触发 handoff 决策
- [[feedback_auto_codex_after_changes]] — 日常大修自动跑 codex 的判定
- [[feedback_codex_review_deep]] — 日常 codex review 5 dim 深度
- [[feedback_live_save_memory]] — session 关键节点刷 memory
- [[feedback_plan_before_code]] — 改代码前先 plan 等 go-ahead
- handoff skill: `~/.claude/skills/handoff/SKILL.md`（codex review 时用）
- dual-session skill (optional, write your own per-project helper to refresh writer snapshot)

---
> Source: [eacsai/feature-flow](https://github.com/eacsai/feature-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
