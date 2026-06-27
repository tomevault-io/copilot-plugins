## tick-tui

> `tick-tui` 是 Tick 任务管理系统的命令行 TUI 客户端，基于 Go + bubbletea。

# CLAUDE.md

## 项目概述

`tick-tui` 是 Tick 任务管理系统的命令行 TUI 客户端，基于 Go + bubbletea。
设计理念是 lazygit 风格的窄窗口（约 40 字符宽），全程同一画面，没有任何弹框 / modal / popup —— 所有编辑就地内联。

数据直接读写本地 markdown 文件（路径由首次启动 wizard 决定，默认 fallback `~/tick/tasks.md`），**不再依赖任何服务端**。
推荐放在 Obsidian vault 里，手机端在 Obsidian 里手敲新任务即可，tick-tui 启动时自动补全 metadata。

我（开发者）个人的实际路径是 `~/Documents/hoard/tick/tasks.md`（hoard = 我的 Obsidian vault），仅作为本文档中举例参考；新用户的路径由 wizard 选定。

## 架构

```
cmd/tick/main.go           入口：加载配置、建 store、跑 bubbletea
cmd/seed/main.go           dev 工具：灌假数据（--days/--avg/--out）
internal/store/
  types.go                 Feature / TodayResponse / ProjectItem
  markdown.go              parser / serializer / 文件 IO / Store 接口实现
  markdown_test.go         单测
  stats.go                 GetCompletionsByDate：跨 tasks.md+archive.md 按日期计数
  stats_test.go            单测
internal/config/
  config.go                读 / 写 ~/.config/tick/config（key=value，行内 ` #` 注释）
  config_test.go
internal/setup/
  detect.go                Obsidian vault 检测（读 obsidian.json）
  detect_test.go
  strings.go               wizard 自己的 i18n 表（与主屏 i18n 解耦）
  wizard.go                首次启动 / O 修改文件夹的 bubbletea 子 model
  wizard_test.go
internal/i18n/
  i18n.go                  TUI 双语字符串表 + Lang 类型 + 星期/月份本地化
  i18n_test.go             单测
internal/tui/
  model.go                 Model + 状态机常量 + buildRows + 项目分组排序 + lang/strings 字段
  update.go                Update：消息分发、store tea.Cmd、按键 handler、l 切换语言
  view.go                  View：列表渲染、padBetween、scrollWindow（走 m.strings 表）
  editor.go                ComputeGhostText / renderTitleWithGhost / renderProjectField
  styles.go                lipgloss 样式集中
  keys.go                  bubbles/key 绑定 + footerShortHelp（用 m.strings）
  stats.go                 renderBars30 / renderHeatYear 纯渲染函数（接 i18n.TUIStrings）
  stats_test.go            stats 渲染断言（EN + ZH 双语断言）
  update_test.go           关键状态机单测（含 stats/settings/lang 切换）
internal/watcher/
  watcher.go               fsnotify-based tasks.md 监听
```

依赖：`charmbracelet/bubbletea` v1.3 · `bubbles` v1.0 · `lipgloss` v1.1 · `atotto/clipboard`。

## 数据模型

```go
type Feature struct {
    ID          string  // 8-char hex (e.g. "a3k7m2x9"); empty = not yet assigned
    Title       string
    ProjectName *string
    IsDone      int     // 0/1
    CompletedAt *string // YYYY-MM-DD; nil 表示未完成
    CreatedAt   string  // YYYY-MM-DD
}
```

## 文件格式

紧凑 ASCII 单行，字段顺序自由（解析时位置无关）：

```
- [ ] buy milk @home +2026-05-01 [a3k7m2x9]
- [x] write report @work +2026-04-29 *2026-04-30 [b1d4e5f0]
- [ ] 买菜 @家庭 +2026-05-01 [c2f3a4b5]
```

| 部分 | 含义 | 必填 |
|---|---|---|
| `- [ ]` / `- [x]` | 状态 | 是 |
| 描述文本 | task title（含 CJK） | 是 |
| `@project` | 可选项目（`@` + 非空白） | 否 |
| `+YYYY-MM-DD` | 创建日 | 否（缺则 sweep 时补 today） |
| `*YYYY-MM-DD` | 完成日（仅 `[x]` 行） | 否（缺则 sweep 时补 today） |
| `[ID]` | 8 字符 hex 随机 ID，**强制行尾** | 否（缺则 sweep 时随机分配） |

ID 用 8 字符 hex（`crypto/rand` 4 字节）。**为什么不用自增数字**：手机插件 + Mac CLI 双向同步时，两边都按"max + 1"会撞 ID（实际遇到过两条任务 [63] 同 ID 导致 mark-done 走错行）。8 hex = 32 bit ≈ 40亿种，碰撞概率近 0。

正则 `\s\[([a-zA-Z0-9]{1,16})\]\s*$` — 接受 1-16 字符，兼容旧数字 ID 直到下次 sweep 自动重写为 hex。`[3 个]` 这类 CJK 描述不会误匹配（中文不是 alphanumeric）。

如果两条行碰巧同 ID（迁移残留 / 极端碰撞），sweep 会给第二条重新 roll 一个。

## 双文件 + 7 天滚动归档

```
<wizard 选定的目录>/
  tasks.md       ← undone + 过去 7 天的 done — 任意时刻 < 350 行 / 35 KB
  archive.md     ← 7 天前的 done；append-only；TUI 列表不读它
```

mark-done / undone 是**就地操作**（仅修改 tasks.md），不跨文件移动。

每次 `loadTasks()` 跑一次"被动 sweep"：
1. 缺 `[ID]` → 分配 `genID()`（8 hex chars）
2. 重复 `[ID]` → 第二条重新 roll
3. 缺 `+date` → 补 `+today`
4. 状态 `[x]` 但缺 `*date` → 补 `*today`（手机端手动勾选语义补丁）
5. 状态 `[x]` 且 `*date < today-7d` → 移到 archive.md

效果：手机端在 Obsidian 里手敲 `- [ ] 任务 @项目` 一行，sync 到 Mac，tick-tui 启动 → 自动补成完整一行写回。

## TUI today 语义

- `pending = tasks.md 中所有 [ ] 行`
- `done section = tasks.md 中 [x] 且 *date == today 的行`（TodayResponse.Done）
- `done section 续` = tasks.md 中 [x] 且 *date == yesterday 的行（TodayResponse.DoneYesterday）
  - 两者共用同一个 "── done ──" separator，yesterday 行尾显示 dim 的 `-1d` 标记
  - row.daysAgo: 0 = today/pending，1 = yesterday done
- 历史完成（2-6 天前 + archive.md）TUI 列表不读

## Stats 30-day drill-down

### 选中模式（drill-down）

- 进入 modeStats30 默认**未选中**（无右 panel，首屏）
- 按 `←` 一次 → 选中 today，进入 drill-down；右 panel 显示该日 task list
- 再按 `←` → selectedDate -= 1d；超出 bars 窗口左边界时 statsWindowEnd -= 1（窗口左移），无界往前
- `→` 反向；selectedDate 不超过 statsEnd（today）
- `↑` / `k` / `↓` / `j` 滚动 task panel（selectedScroll）
- `esc` 第一次：清 selectedDate，回首屏；第二次：退 stats 回 modeList
- `s` 切 30 天：重置 selectedDate / selectedTasks / selectedScroll / statsWindowEnd

### Task panel

- 最多显示 10 条；超出显示 `↑ 上方 X 条` / `↓ 还有 X 条`
- 切到新日期时 selectedScroll 重置为 0
- task 行格式：`· @proj title`（proj 为空时 `· title`）；title 截断到 panel 宽度（lipgloss.Width 安全）

### 布局

- `width >= 70`（`wideStatsWidth`）→ 宽布局：bars 占 `barsAreaWidth=36` 列，右侧 panel 占剩余
- `width < 70` → 窄布局：panel 在 bars 下方单列堆叠

### Streak 算法

- 从 today 倒数，遇到第一个 count == 0 的日停止
- 使用 `statsData`（最近 30 天），streak >= 30 显示 `🔥 30+`
- 在 `statsLoadedMsg` 到达时（加载完成后）由 `computeStreak()` 计算一次，缓存到 `m.streak`
- 标题行右侧显示 `🔥 N 天`（ZH）/ `🔥 Nd`（EN）

### 数据来源

- `store.GetTasksOnDate(d)` 扫 tasks.md + archive.md，返回指定日的所有完成任务，按 ID 升序稳定排序
- 每次 selectedDate 变化触发 `cmdLoadTasksOnDate`；stale response 用 `sameDay()` helper 按 Y/M/D 比较丢弃（不依赖字符串 format，避开 time-of-day drift）
- `store.OldestCompletionDate()` 在 enterStats30 / enterStatsYear 触发一次（缓存到 `m.oldestDataDate`），ScrollLeft 在 proposed date 早于此值时拒绝并显示 footer 提示 `NoOlderData`

## bubbletea 状态机

```go
type mode int
const (
    modeList          // 默认浏览态
    modeEdit          // a 新建 / e 编辑
    modeConfirmUntick // U 后等 y/n
    modeConfirmDelete // D 后等 y/n
    modeGraceUndo     // t 后 3s grace
    modeStats30       // s：30 天柱状图
    modeStatsYear     // S：年度热力图
    modeSettings      // O：修改文件夹（复用 wizard sub-model）
)

type editField int
const (
    fieldTitle
    fieldProject
    fieldDate
)
```

### Edit 模式分两种

| 进入方式             | editingDone | 起始字段     | Tab 行为                          | 字段集            |
|----------------------|-------------|--------------|-----------------------------------|-------------------|
| `a`（连续新建）       | false       | `fieldTitle` | title ↔ project 循环              | title + project   |
| `e` 在 pending 行     | false       | `fieldTitle` | title ↔ project 循环              | title + project   |
| `e` 在 done 行        | true        | `fieldDate`  | no-op（只有一个字段）             | date              |

cmdSave 始终带上 titleInput/projectInput 当前值；`dateModified` 只有用户在 fieldDate 按 ↑/↓ 时才会变 true。

### `a` 的连续添加（sticky）

按 `a` → 设 `m.addSticky=true` → enterEditNew → rows 顶部插 `rowDraft` phantom。
保存后 todayLoadedMsg 看到 sticky 仍为 true → 自动重新 enterEditNew。
ESC 或空 Enter 关闭 sticky 退出。

### Mark Done 流程

按 `t` → mode=modeGraceUndo，graceID=feature.id → `tea.Batch(cmdMarkDone, cmdGraceTimer(3s))`
3s 内按 `u` → 调 store.Undone，回 modeList
其他键 / 3s 过期 → 回 modeList，footer 清空

store.MarkDone **就地**改 `[x]` + 加 `*today`，不跨文件。

### Pending 区项目分组排序

`groupByProject` 把 pending 按项目分组，组间按 count desc，无项目组永远放最末。
done 区不分组。

`[` / `]` 跳上/下一个项目首行；`g` / `G` 跳当前 section（pending 或 done）的首/末行。

### 项目过滤（`p` 键）

按 `p` → 取当前光标行的项目作为 filter；列表只显示匹配项目（含 done）；title bar 显示 ` · @work`。
新建任务自动归该项目（enterEditNew 优先用 activeProject 预填 project 字段）。
再按 `p` 关闭 filter。

### Ghost Text

只在 `fieldProject` 工作（`editor.go: computeProjectGhost`）：前缀匹配 `m.projects` 列表第一个，dim 显示在光标后；Tab 接受。
`fieldTitle` 不做 @-completion。

`m.projects` 在 `Init()` 通过 `cmdLoadProjects()` 拉一次。

### CJK 安全

`renderTitleWithGhost` 用 `[]rune` 切片避免 byte 切 UTF-8 中间的 panic。
项目名 regex 用 `@(\S+)`（非空白即可），匹配 CJK 项目名。

## 完整键位

| 键            | 模式         | 作用                                          |
|---------------|--------------|-----------------------------------------------|
| `j` `k` ↑↓    | List         | 上下移动（跳过 separator）                    |
| `Nj` `Nk`     | List         | vim 风格：数字前缀重复 N 次（如 `5j` 下 5 行）|
| `[` `]`       | List         | 跳上/下一个项目首行                           |
| `g` `G`       | List         | 跳当前 section 首/末行                        |
| `t`           | List         | 标 done（pending 行）+ 3s grace               |
| `u`           | GraceUndo    | grace 内反标                                  |
| `U`           | List         | done 行反标（y/n 确认）                       |
| `a`           | List         | 连续新建：Enter 保存后立即开新 draft；Esc 或空 Enter 退出 |
| `p`           | List         | 切换项目过滤：按下=只显示当前光标行的项目；再按=回到全部 |
| `e`           | List         | 编辑当前行（pending: title/project; done: date）|
| `D`           | List         | 删除（y/n 确认）                              |
| `y`           | List         | 复制当前行的 title 到剪贴板                   |
| `s`           | List/Stats   | 30 天柱状图（Stats 模式内：切到 30 天，重置选中）|
| `S`           | List/Stats   | 年度热力图（Stats 模式内：切到年度）           |
| `O`           | List         | 修改文件夹（复用 wizard；选完后 q 重启生效）   |
| `l`           | List/Stats   | 切换中英文（持久化到 ~/.config/tick/config）   |
| `?`           | List         | 切换详细帮助                                  |
| `q` / `Ctrl+C`| List         | 退出                                          |
| `←`           | Stats30      | 首次按：进入 drill-down（选中 today）；继续按：选中前一天；超出 30 天窗口时左移窗口 |
| `→`           | Stats30      | 选中后一天（不超过 today）；窗口右移同步       |
| `↑` / `k`     | Stats30 drill | 向上滚动 task panel                           |
| `↓` / `j`     | Stats30 drill | 向下滚动 task panel                           |
| Tab           | Edit         | 切下一个字段（pending edit 内）；项目 ghost 时先接受 |
| Shift+Tab     | Edit         | 反向切                                        |
| ↑ ↓           | Edit/Date    | ±1 天                                         |
| Enter         | Edit         | 保存所有字段                                  |
| ESC           | Edit         | 丢弃                                          |
| ESC           | Stats30 drill | 第一次：退选中（回首屏）；第二次：退 stats 回主屏 |
| ESC           | StatsYear/Stats30 idle | 退 stats 回主屏                  |
| `y`           | Confirm      | 执行 untick / delete                          |
| 任何其他键    | Confirm      | 取消                                          |

## 开发

```bash
go test ./...                  # 全部测试
make build                     # bin/tick
make install                   # cp 到 ~/.local/bin/tick
./bin/tick                     # 运行（首次启动 wizard 选路径）
make seed                      # 灌 365 天假数据到 /tmp/tick-demo（测试统计视图）
```

`go env -w GOPROXY=https://goproxy.cn,direct` 走中国镜像。

## 测试 stats 视图

```bash
# 1. 灌假数据
go run ./cmd/seed --days 365 --avg 5 --out /tmp/tick-demo

# 2. 启动（覆盖配置路径）
TICK_TASKS_FILE=/tmp/tick-demo/tasks.md ./bin/tick

# 3. 主屏看到 5 条 pending + 若干 done

# 4. 按 s            → 30 天柱状图（窄窗口 40 列正常）
#    按 S            → 年度热力图（需要 ≥ 60 列；窄窗口显示 resize 提示）
#    在两个图之间    → s ↔ S 互切
#    按 esc / q      → 回主屏

# 5. 按 O            → 修改文件夹（wizard 子模式）
#    选新路径 → enter → footer 提示 "config updated · q to restart"
#    按 q            → 退出；下次启动用新路径

# 6. 清理
rm -rf /tmp/tick-demo
```

flag 说明（`go run ./cmd/seed`）：
- `--days N`：覆盖最近 N 天（默认 365）
- `--avg M`：平均每天 M 条 done（默认 5）
- `--out DIR`：输出目录（**必填**，避免污染真实 tasks.md）

## 配置

首次启动 wizard 自动写 `~/.config/tick/config`（mode 0600）：

```
TICK_TASKS_FILE=<wizard 选定的绝对路径>
TICK_LANG=en        # 或 zh；按 l 即时切换并回写
```

行内注释 ` #`（空格 + 井号）会被截断。空值或字段缺失时 fallback：
- `TICK_TASKS_FILE` → `~/tick/tasks.md`
- `TICK_LANG` → `en`

`archive.md` 自动放在 tasks.md 同一目录。

Wizard 会扫 `~/Library/Application Support/obsidian/obsidian.json`（Mac）或 `~/.config/obsidian/obsidian.json`（Linux）列出已注册的 vault；用户选 vault 后路径自动拼成 `<vault>/tick/tasks.md`。Wizard 内 `l` 切英中（与主屏统一；路径输入框内 `l` 是普通字符，要切语言先 Esc 回选项页再按 `l`；仅影响 wizard 屏，主屏的 `l` 键独立持久化到 TICK_LANG）。

## 后续待做（v2）

- ~~统计面板：30 天柱状图 + 年度热力图~~（已完成：`s`/`S` 键）
- ~~修改文件夹~~（已完成：`O` 键）
- archive 按年拆分（5 年后再考虑）
- `/` 搜索 / 过滤
- 提交 `tick-obsidian` 到 Obsidian 官方插件市场（PR 到 `obsidianmd/obsidian-releases`）— 当前用户走 BRAT

## 设计决策（不要回退）

1. **mark-done 就地不跨文件**：tasks.md 保留 7 天 done，sweep 时才挪到 archive。理由：高频路径（每次写）零跨文件 IO；done section 当天能展示；统计面板（archive.md）是 read-only 路径不影响热路径。
2. **手机端手敲容错**：解析非常宽容，缺 ID/date 自动补；`[x]` 缺 *date 当今日处理；`[x] *date < today-7d` 自动归档。让用户在 Obsidian 里随手敲一行就行。
3. **title 字段不做 @-completion**：用户明确说 title 不需要 @；项目改在 fieldProject 选。
4. **done 行 e 只能改 date**：避免误改已完成任务的 title/project。
5. **不做乐观更新**：所有写操作等 store 返回再 reload。本地 IO 极快。
6. **rowDraft phantom**：按 a 在 rows 顶部插一行 phantom，不动 m.today；exitEdit 通过 buildRows 自动清理。
7. **`a` 永远 sticky**：连续新建是默认；不再保留"加一条退出"的非 sticky 模式。
8. **8 字符 hex 随机 ID（不是顺序整数）**：手机插件 + Mac CLI 双向同步时，两端按"max+1"会撞 ID（实际遇到过 [63] 同 ID 导致 mark-done 走错行）。32 bit hex 碰撞概率近 0；sweep 还会兜底 re-roll 重复。
9. **plain `tick/` 目录 + 插件层隐藏（v0.5.1 反转决策）**：v0.5.0 之前用 `.tick/` dot-prefix，靠 Obsidian 文件树原生隐藏 dot 目录避免误改 markdown。v0.5.1 改成 plain `tick/`，理由：**Obsidian Sync 强制忽略所有 dot 文件夹且没有开关**（[2022 至今未实现的 feature request](https://forum.obsidian.md/t/obsidian-sync-sync-hidden-files-as-well/32123)），`.tick/` 让最常见的"vault + Obsidian Sync"组合根本无法跨设备同步。改成 plain dir 后 sync 立即可用；视觉隐藏改由 tick-obsidian 插件 onload 时注入 file-explorer CSS 完成（用户必装 tick-obsidian，所以零额外操作）。代价：纯 CLI 用户 / 不装插件的 vault 用户会在文件树看到 `tick/`，可接受（解析仍宽容，误改 ID 由 sweep 兜底）。
10. **首次启动 wizard，不强制 hard-coded 路径**：`internal/setup/wizard.go` 扫 obsidian.json 列出 vaults，让用户选 vault 或自定义路径或默认 `~/tick/tasks.md`。Tab 或 l 切英中（modeCustom 下只能 Tab）。配置写到 `~/.config/tick/config` 后续不再问。
11. **fsnotify 监听 + 编辑期间延迟 reload**：`internal/watcher` 监听父目录（atomic write 换 inode），消息通过 `tea.Program.Send(FileChangedMsg{})`；如果用户正在 modeEdit/Confirm/Grace/Stats/Settings，先 `m.pendingReload=true` 等回到 modeList 再 drain，避免吞掉用户半途的输入。
12. **stats 路径只读不 sweep**：`GetCompletionsByDate` 用 `loadTasksLockedSimple()` + `loadArchive()` 纯读，不触发 ID/date 补全写盘。统计是只读路径，副作用 sweep 会破坏"不变量：loadCompletions 读 ≠ loadTasks 写"的分离设计。
13. **两个独立 stats 键 `s`/`S`**：30 天图保持 40 列窄窗口（`barRows=5, 30 列`）；年度图需 ≥60 列（53 周 × 1 字符），宽度不足时显示单行 resize 提示而非空白或 panic。
14. **seed `--out` 强制指定目录**：不写默认路径，避免意外污染 `~/tick/tasks.md` 或 Obsidian vault。
15. **修改文件夹不做 hot-swap**：config.Write 后返回主屏，footer 提示"q to restart"；重建 store+watcher 的成本远高于一次重启。
16. **TUI i18n 与 setup wizard i18n 解耦**：`internal/i18n` 服务于主屏（list/stats/footer/transient），`internal/setup` 内置自己的 strings 表（仅 wizard 字段）。两个独立体系：主屏 `l` 切换持久化到 `TICK_LANG`；wizard 内 `l` 切换仅当次屏内有效。理由：字段不重叠（wizard 提的"vault"/"custom path"主屏没有；主屏的"un-tick"/"copied"wizard 没有），合并会产生大量空字段并把两个独立 UI 模块耦合死。
17. **`l` 切换语言不重载数据**：strings 是渲染层；按 l 只改 `m.lang`/`m.strings` 并写 config，不调用 `cmdLoadToday`。同一份 features 用新表重渲即可。
18. **`l` 在 edit/confirm/grace 模式下不响应**：edit 模式下 `l` 是 textinput 普通字符（必须能在标题里输入字母 l）；confirm/grace 模式不期望干扰当前流。仅 modeList 和 modeStats* 接受 `l`。modeSettings 也不响应，让 wizard 自己的键继续生效。
19. **wizard 切语言改用 `l`（与主屏统一），不保留 Tab**：`l` 仅在 modePick 切换；modeCustom 下 `l` 让 textinput 吃，避免误吞合法路径字符（`~/local/foo/tasks.md`）。要切语言先 Esc 回 modePick 再按 `l`。
20. **streak 只看 last 30 天数据**：`computeStreak` 从 today 倒数最多 30 天；超过 30 天的连续 streak 显示 `🔥 30+`。简化版降低 API 调用，足够覆盖绝大多数真实使用场景；需要精确 365 天 streak 可在 v2 扩展。
21. **drill-down 翻到最早一天就停**：`←` 翻到 `oldestDataDate`（最早完成日）再按一次会显示 `NoOlderData` 提示而不是无限往前翻——避免用户进入"全是空柱子"的虚无地带。`oldestDataDate` 在 enterStats30/Year 时一次性懒加载，未加载时（zero）按"无界"处理（首帧体验）。
22. **宽 ≥ 70 时左右布局**：bars 区固定 `barsAreaWidth=36` 列，右侧 panel 按剩余宽自适应；窄窗口（< 70）panel 落底，保持单列可读。
23. **drill-down stale response 丢弃**：`tasksOnDateLoadedMsg` 到达时用 `sameDay()` helper（Y/M/D field 比较）丢弃 stale 响应——不用字符串 format 比较，避免 time-of-day drift 误判。
24. **archive.md 缺失静默兜底**：`loadArchive()` 遇 `os.IsNotExist` 返回空 slice + nil，让 stats 路径在 sync 冲突或手工删除场景下不报错——只读路径不该因可缺数据源而 fail。

## 发布渠道

| 仓库 | 干什么用 | 当前版本 |
|---|---|---|
| [`al4danim/tick-tui`](https://github.com/al4danim/tick-tui) | CLI 源代码 + GitHub Actions release | v0.5.2 |
| [`al4danim/tick-obsidian`](https://github.com/al4danim/tick-obsidian) | Obsidian 插件源代码 | 0.2.1 |
| [`al4danim/homebrew-tick`](https://github.com/al4danim/homebrew-tick) | Homebrew tap formula | 跟 tick-tui 同步 |

发版流程：
1. `tick-tui` 改完 → bump tag `vX.Y.Z` → push → `.github/workflows/release.yml` 跑 goreleaser → 4 平台 binary 上 GitHub Releases，formula 自动推到 `homebrew-tick`
2. `tick-obsidian` 改完 → bump `manifest.json` + `package.json` + `versions.json` 三个版本号 → tag `X.Y.Z`（无 v 前缀，Obsidian 惯例）→ push → action 上传 `main.js` / `manifest.json` / `styles.css` 到 release

GitHub Actions 推 formula 到 `homebrew-tick` 用的是 fine-grained PAT，存为 `tick-tui` 仓库的 `HOMEBREW_TAP_TOKEN` secret。PAT 必须有 `al4danim/homebrew-tick` 的 `Contents: Read and write`。**已知问题**：v0.3.0 / v0.5.0 / v0.5.1 release 时 brew 推送都报 403 `Resource not accessible by integration`，formula 长期手工推（每次发版后手动 git commit 新版 formula 到 homebrew-tick）。release.yml 上的 PAT 应该重新签发 / 验证 fine-grained 权限范围。

用户安装：
- CLI：`brew tap al4danim/tick && brew install tick`
- 插件：BRAT → Add Beta Plugin → `al4danim/tick-obsidian`

## 相关仓库

- `~/Sync/tick-obsidian/`     Obsidian 插件，配套客户端（直接读写同一个 tasks.md）
- `~/Sync/feature-check/`     旧服务端 (FastAPI + SQLite)，**已废弃**，仅作迁移源
- `~/Sync/zsh-tick/`          旧 zsh + fzf 客户端，**已废弃**（注意：曾在 `~/.zshrc` 加过 `export PATH=...zsh-tick/bin:...`，会遮蔽 brew 的 tick；安装新版后记得清理）

---
> Source: [al4danim/tick-tui](https://github.com/al4danim/tick-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
