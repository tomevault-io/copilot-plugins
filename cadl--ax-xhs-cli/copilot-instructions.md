## ax-xhs-cli

> 通过 macOS Accessibility API (axcli) 自动化操作小红书的 Rust CLI 工具。所有交互模拟真人操作——鼠标沿贝塞尔曲线移动，键盘/点击使用 OS 级 CGEvent。

# ax-xhs-cli

通过 macOS Accessibility API (axcli) 自动化操作小红书的 Rust CLI 工具。所有交互模拟真人操作——鼠标沿贝塞尔曲线移动，键盘/点击使用 OS 级 CGEvent。

## 架构

axcli 以 Rust 库形式集成（`axcli-lib`），不再通过子进程调用 CLI。直接操作 `AXNode` 对象提取数据，不解析文本输出。

```
src/
├── main.rs            # CLI 入口，clap 场景命令 + 子命令定义
├── axcli.rs           # axcli 库封装 + human_click + scroll + 截图颜色检测
├── mouse.rs           # CoreGraphics 鼠标轨迹（贝塞尔曲线 + ease-out fast-brake + 过冲修正 + 滚动）
├── output.rs          # 输出格式化（text/json/yaml）
├── session.rs         # Session 管理 + ChildTab + 场景参数(scene_params) + 状态机
├── parser.rs          # AXNode 树直接提取（笔记卡片、用户资料、评论、通知）
└── commands/
    ├── actions.rs     # 共享 NoteAction 子命令 + dispatch（show-note/like-note/...）
    ├── search.rs      # 搜索场景 + 分页滚动采集（scroll_and_collect）
    ├── search_user.rs # 搜索用户场景 + show-user 子命令
    ├── feeds.rs       # 首页推荐场景 + 分页滚动
    ├── user_profile.rs # 用户主页场景（子 tab 管理 + list/close）
    ├── notification.rs # 通知场景（评论和@、赞和收藏、新增关注）
    ├── open.rs        # URL 直接访问（open-note + open-user）
    ├── inspect.rs     # AX 树结构检查（调试用）
    ├── login.rs       # 登录状态检查
    └── session_cmd.rs # session start/list/end/status
```

## 命令体系

命令按**场景**组织，每个场景有可选的**子命令**：

```
search -k <keyword> [--scene-sort ...] [子命令]    # 搜索笔记场景
search-user -k <keyword> [--size N] [子命令]       # 搜索用户场景
feeds [子命令]                                      # 首页推荐场景
user-profile --scene-name <用户> [子命令]           # 用户主页场景
notification --scene-tab <tab> [子命令]              # 通知场景
open-note <URL> [子命令]                             # URL 笔记场景
open-user <URL>                                      # URL 用户场景
```

共享子命令（NoteAction）：show-note, show-user, like-note, unlike-note, favorite-note, unfavorite-note, comment-note, show-comments

search-user 子命令（SearchUserAction）：show-user

## 场景参数 (--scene-xxx)

以 `--scene-` 前缀的参数定义场景上下文，保存到 `session.scene_params` HashMap 中。

- 不带子命令时：进入/刷新场景，保存场景参数
- 带子命令时：检查场景参数一致性
  - 省略场景参数 → 使用 session 中保存的
  - 相同参数 → 正常执行
  - 不同参数 → 报错，提示需先切换场景

## 关键设计决策

- **axcli 库集成**: 通过 `AXNode::locate()` / `locate_all()` 直接查找元素，用 `.text()` / `.texts()` / `.position()` 提取数据
- **Locator 语法**: `.class` 选择器前不能加 role 短名（如 `group.note-item` 会被解析为两个 class）。正确写法：`.note-item`、`#detail-title`、`textfield#search-input`
- **Session 模型**: 每个 session 通过 Chrome tab ID 绑定到一个 tab，场景参数保存到 `scene_params` HashMap
- **human_click = move_to（含过冲修正）+ 500-1500ms 停顿 + CGEvent click**
- **human_type = 拼音 IME 节奏**: 中文按 2-3 字词组出现，7% 错字率（`AX_TYPO_RATE` 可调）
- **搜索流程**: 非首页时点 logo 回首页（不用 URL 导航），同关键词翻页直接滚动
- **Like/Favorite 状态**: 通过截图颜色检测 icon（红心=已点赞，黄星=已收藏）
- **子 tab 管理**: show-user/open-user 开新 tab，ChildTab 记录 tab_id/nickname/xhs_id/results，保留至 session end
- **0-based 索引**: 所有结果索引从 0 开始
- **输出格式**: 全局 `--format text|json|yaml`，output.rs 模块封装
- **分页滚动**: `scroll_and_collect()` 共享函数，CGEvent 滚轮滚动 + 去重累积

## 状态机

四个稳定状态：**首页**、**搜索结果页**、**用户搜索结果页**、**通知页**。note/user/like 等是临时操作（弹窗/子tab）。

- 首页 → `search` → 搜索结果
- 搜索结果 → `search`（换关键词）→ 首页（点 logo）→ 搜索结果
- 任意 → `search-user` → 用户搜索结果（先搜索再点"用户"tab）
- 任意 → `feeds` → 首页（点 logo）
- 任意 → `notification` → 通知页（点侧栏"通知"）
- 通知页 → `feeds` → 首页（点 logo）
- `open-note` → 笔记详情页（URL 导航）
- `open-user` → 先回首页 → 子 tab 打开用户页

## 构建和运行

```bash
cargo build
./target/debug/ax-xhs-cli --help
```

## 测试

测试用例在 `tests/cases/` 目录，是自然语言描述文件（不是代码）。

### 快速冒烟测试

```bash
CLI=./target/debug/ax-xhs-cli
$CLI status
rm -f ~/.ax-xhs-cli/sessions/*.json
$CLI session start "test"
$CLI --session test search -k "编程" --scene-sort "最新" --size 5
$CLI --session test search show-note 0
$CLI --session test search show-user 0 --size 3
$CLI --session test user-profile list
$CLI --session test user-profile --scene-name "用户名" show-note 0
$CLI --session test user-profile close "用户名"
$CLI --session test search-user -k "编程" --size 5
$CLI --session test search-user show-user 0
$CLI --session test feeds --size 5
$CLI --session test feeds show-note 0
$CLI --session test notification --scene-tab "赞和收藏"
$CLI inspect ".feeds-container" --depth 3
$CLI session end "test"
```

## 已知限制

- Chrome 必须在运行，且运行本 CLI 的终端 app 需有辅助功能权限
- 小红书页面结构可能随版本更新变化，locator 选择器需要维护
- Chrome PID 通过 `pgrep -x "Google Chrome"` 获取，多 Chrome 实例时取第一个
- 笔记 URL 需包含 `xsec_token` 参数才能直接打开

---
> Source: [cadl/ax-xhs-cli](https://github.com/cadl/ax-xhs-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
