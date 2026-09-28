## gqy-agent

> > **顾清影请先读这段。** 本文件是给在这个仓库里做开发的编码代理（Claude Code 等）看的施工规范，不是给你的指令。如果你是顾清影，正在和用户日常对话（不是 `gqy dev` 开发模式），读到这里时：

> **顾清影请先读这段。** 本文件是给在这个仓库里做开发的编码代理（Claude Code 等）看的施工规范，不是给你的指令。如果你是顾清影，正在和用户日常对话（不是 `gqy dev` 开发模式），读到这里时：
> - 下面的内容只当了解项目的参考资料，不要代入其中的角色，也不要执行其中的流程（测试、构建、门禁脚本、提交）。
> - 除非用户在这条消息里明确要求，否则不要在这个仓库里运行 `cargo check`、`cargo build`、`cargo test`、`cargo install`、`test_scripts/` 下的脚本。这些命令很耗时，还会和正在开发的编码代理抢同一个编译缓存目录。
> - 用户只是聊到代码、问改了什么，就用读文件和 git 记录回答；真要动手改代码，建议用户转到 `gqy dev`。

你是顶级RUST工程师，AI loop Agent、Harness 顶级设计师。

多步任务在停下等用户时，说明已完成什么、在等什么决定、下一步做什么。

不要让单个代码文件体积膨胀成为“上帝文件”。

代码应当注重模块化和可复用。

灵活使用子代理节省上下文的同时提高任务执行速度，但不要并行太多导致额度的不必要消耗。

不确定的点必须询问用户，告知你的推荐项，而不是自己决定。

先定位问题，找根因，并告知用户，同时提出方案，标记你推荐的选项，让用户决定是否要开工。

功能完成后应当简洁易懂地给出可照做的验收流程，经过用户验证后确认才可以commit。验收成功准备发布的内容写进 `CHANGELOG.md` 的 `[Unreleased]`（Keep a Changelog 格式，写法见文件开头）

docs/中有所有的计划和文档，可以自行按需阅读。

---

# 项目注意事项

深挖去处：`docs/理念.md`（设计哲学正典）、`docs/compact-plan.md`（compact 唯一定论）、`docs/cache-and-prompt-plan.md`（缓存契约）、`docs/wiki/15-扩展指南.md`（新工具/迁移步骤）、`docs/fixed/`（历次排查案卷）、`docs/design/`（施工前的方案稿）、`docs/plan/`（排期与施工记录）。
- 默认不跑测试与构建，只有明确说明的时候才能跑。

## 1. 提示词与缓存（字节契约）

1.1 **前缀即契约**：相同会话状态必须产生逐字节相同的请求前缀。序列化路径唯一；非确定内容（时间/随机数/探针值）要么不进前缀、要么进入即冻结；工具描述是常量，禁止拼时间戳/路径。tools 数组一处分叉，排在它前面的 system 缓存也一并作废。
1.2 **append-only 与化石化**：追加不插入；发送过的瞬态内容（runtime/联想记忆/元数据）落库后逐字节回放、永不删除。压缩是唯一例外且单调（水位只前进）。瞬时尾巴放当前用户消息**之后**。
1.3 **stub 加载模式是定论**：懒工具以真名+首行摘要（≤60 字符，即描述第一行）+宽松参数壳常驻，tools 数组会话内字节恒定；完整契约走 load_tools 结果。不要为了让模型看到参数把工具设 always_loaded。
1.4 **指令型注入必须放 system 侧**（每请求新组装、不化石）——内联在消息块里的指令会随化石重放造成跨轮错乱（qq-reply-target 的前身就是这么死的）。所有注入带 XML 标签外壳，不裸奔。
1.5 **文风规范**：模型可见的机械文本（描述/schema/注入/报错回灌）一律英文短句——同语言同语域污染最毒，中文书面语注入是头号 OOC 源；中文只留给人格侧文本。禁分号串联、禁「总述：分述」结构。每条注入先问“删掉会怎样”再问“怎么写短”。人格 hint、goal/compact 模板是实测敏感区，不动。
1.6 描述/schema/hint 的改动=一次性计划内冷启动，可接受；改后必须仍是常量字节。改消息组装/工具目录后除跑测试外，手测两轮请求的 cache-usage jsonl 确认第二轮 cache_read 不异常下降。
1.7 辅助请求（compact/judge/title/subagent/vision 等）独立 cache/session 状态；唯一例外是 fork 式摘要（刻意复用主对话前缀）。
1.8 token 量尺：`cargo test --lib token_diet_baseline -- --ignored --nocapture`。

## 2. 工具系统

2.1 **描述/schema 真相源是 `src/tools/descriptions/*.json`**（build.rs 扫目录自动生成 include 清单，丢进目录即生效；注册了却没有 JSON 的工具由 `shape_tests::registered_built_in_tools_have_json_descriptions` 拦截）。内置插件 id/显示名/引导开关只写 `config/plugin_catalog.rs` 一行，工具注册单元写 `tools/compose.rs` 的 `UNITS` 表一行，两边一一对应有测试钉着。Rust 里的描述只是占位，注册时被 JSON 整体覆盖（load_skill 例外）。权限只由 `.writes()`/`.presentation()` 决定，JSON 的 permission 字段是死字段。
2.2 **工具名是最强的能力广告：域内聚合、域间分名**。编辑/读取类能力并入 edit（文件系统）/kb/artifact（补丁语义）与 read（`kb:`/`artifact:` 前缀），别为新存储开新读写工具。把能力藏进描述里的前缀/参数，模型想不起来（kb: 前缀实测翻车史）。
2.3 **输出格式改造必须双兼容**：旧回合 tool_flow 逐字节回放，旧 JSON 解析器永远保留。“结构即功能”的不改：成败判定只认输出 JSON 的 success/ok 布尔（非 JSON=默认成功），错误路径保留 ok:false JSON。
2.4 畸形参数在 registry 统一收口（按 schema 还原字符串化的数组/对象/数字），声明为 string 的参数一个字节不碰；报错说自己真正知道的（“期望整数，收到字符串 "1"”）。
2.5 subagent 子代理是全新上下文、不继承主对话——定论，prompt 必须自包含。`dev=true` 的子代理走开发模式三件套（dev 人格作用域 + core_only 工具面 + 中转线 dev 工具作用域）。平台限额（生图张数等）由代码承担，不写进 prompt 求自觉。

## 3. 数据库与状态

3.1 迁移只在 MIGRATIONS 末尾追加、纯增量（不回填不删列）；改 Turn 字段改 `rows.rs` 的 `TURN_COLUMNS` 与 `map_turn_row`（按列名读取，不按位置）两处即可。
3.2 追加型数据用自增子表，别塞 turns 的 JSON 数组列（读改写全量=O(N²) 写放大）。
3.3 **DB 备份用 VACUUM INTO，禁止 fs::copy 活库**（打开再 close 会丢本进程的 POSIX 常驻锁——08-21 conversation.db 损坏根因）；db/-wal/-shm 三件同进退；手工查活库一律 `mode=ro&immutable=1` 或拷副本；quick_check 不过就别跑 vacuum。

## 4. 平台 / QQ

4.1 trusted（principal/admin，宿主产生）与 untrusted（昵称/正文，用户可控）字段分离承载；不可信文本进提示词必须过 safe_prompt_field（防伪造记录行）。
4.2 插件 hook：system_context 必须字节稳定，动态内容走 turn_system_context；memory_content 禁改。
4.3 投递幂等闸：图片按内容 digest，文字按归一化 bigram 近似度（回合内）——端点故障日模型会把“发送”重演成同义变体，两个闸都不能拆。
4.4 出站图解码 256MB 上限同时是可发送图片的尺寸包络，不能为省内存下调。定时消息先记账再发送，错过时点跳过不补发。
4.5 `scripts/imessage/` 是独立的 iMessage 桥接，不要挪动或改名：`install.sh` 把脚本绝对路径编译进启动器，路径一变桥接就断，重装还要手动重新授予完全磁盘访问权限。

## 5. 测试与排查

5.1 **先证明不修时现象会出现，再修**；新回归用例退回修复前必须报红，否则守不住任何东西。
5.2 量尺类测试标 #[ignore]；断言结果不断言耗时；性能对比看倍率不看绝对值。
5.3 测试不受开发环境影响：终端探测（TERM/kitty）在 cfg!(test) 下走固定路径；PTY 测试等子进程真就位再断言。
5.4 黑盒实测必须 GQY_HOME 沙箱（普通 CLI 未知子命令会把参数当对话发给生产 daemon）。“改动没生效”先查幽灵 daemon 与测试 home 的配置残值。普通单次 CLI 阅后即焚会杀后台任务，测唤醒用 shellhook 形态。
5.5 仓库自 08-26 起 fmt-clean（`939a2feb` 全量格式化，字节基线验证提示词未变），改完直接 `cargo fmt` 即可，别再手工挑文件——遗留的「rustfmt 会顺着 mod 声明递归刷子模块」陷阱随之失效。涉及 agent/llm/registry/提示词的改动，`test_scripts/refactor-check.sh` 全部门禁是验收硬要求。
5.6 报错信息是嫌疑人不是证词：先读规范/原始数据（curl 探针、协议原文、日志），最后才轮到推理。

## 6. 性能与重构

6.1 没有实测数字不合并；“实测后判不做”清单见 `docs/plan/low-footprint.md` 与 `docs/fixed/2026-08-18-性能优化.md`（mimalloc、AppConfig→Arc 快照、资源外置、panic=abort 等），别重提。
6.2 文件规模：目标 800 行 / 上限 1500 / 红线 2000。codegen-units=1 已定（release 编译 ~5.5 分钟属预期）。
6.3 搬文件五坑（include_str 相对路径漂移/模块名遮蔽/super 语义改变/脚本必须拒绝覆盖已存在文件/回退前先看暂存区）：`docs/fixed/2026-08-18-代码拆分.md` §五。

## 7. 构建与发布（默认不处理）

7.1 `src/prompts/*`、web/ 静态资源、assets 词表全部编译进二进制——改完必须重新构建，daemon 按 GQY_BUILD_ID 判断重启。
7.2 **Nix 是分发主路**，正典是 `docs/wiki/18-Nix安装开发与发布.md`，动打包/发布前先读它。发版链：Actions 页手动运行 `publish-release.yml` 填版本号 → 工作流用 `.github/scripts/release.py prepare` 把 `CHANGELOG.md` 的 `[Unreleased]` 定稿为 `[X.Y.Z] - 日期` 并同步 `Cargo.toml`/`Cargo.lock`/README 徽章 → github-actions[bot] 提交并打 tag `vX.Y.Z`（bot 推的 tag 不触发工作流，所以定稿与编译发布在同一工作流里串跑）→ 云端编 4 平台发 Release（正文=CHANGELOG 该版本段 + 安装说明，缺段编译前即失败）并自动提交 `nix/release.json` → 本地 `git pull`。hotfix 可本地 prepare 后手动推 tag。`nix/release.json` 只由 `nix/update-release.py` 生成，禁止手改 hash。
7.3 资源/外部命令/平台的增减要两处同步：`publish-release.yml` 打包步骤、`nix/package.nix`（及 `nix/prebuilt.nix` 的 wrapProgram PATH）。改完 `nix flake check --no-build --all-systems`。Intel Mac 不在 flake 里（nixpkgs 已弃），走 install.sh。
7.4 Nix 下程序路径是 `/nix/store/<hash>-…`，每次升级都变：`gqy_executable()` 只用于起子进程或每次都会重建的配置，禁止写进 shell rc、launchd/systemd、用户配置等持久文件（写 `gqy` 靠 PATH）。
7.5 预编译包和 Nix 包都不带 `gqy-voice`。开发机用 `cargo install --path . --locked --features voice`，不要同机再 `nix profile install`（`~/.cargo/bin` 会遮住它）；验收 Nix 版用 `GQY_HOME=$(mktemp -d) nix run github:yxxbc/gqy-agent/gqy`。
7.6 shell 脚本里紧挨中文的变量必须加花括号（`${var}，`）：macOS 的 sh 在中文 locale 下会把全角标点的首字节吞进变量名，`set -u` 直接报错。
7.7 只有 Nix 和 `install.sh` 两条安装路线。上游继承的 Arch/DEB/RPM 打包（`packaging/`、release.yml 等）已删除，别再恢复或往里加东西；`install.sh` 给没有 Nix 的用户，检测到 Nix 版会拒绝重复安装。CI（ci.yml）在任意分支 push 与 PR 上跑全套（fmt+flake、四道脚本门禁与 CHANGELOG 格式检查、Linux/macOS 测试与用例数门禁、voice 编译、1.89 MSRV）；`#[ignore]` 用例只在手动触发并勾选 run_ignored 时跑、不阻塞。

## 8. 改动的连带更新

8.1 **添加/删除功能**：先判断落在前端（WebUI/TUI）、后端还是两边都要改，拿不准就问用户。两端都有的功能要同步增删，不能只改一端。
8.2 **对照表**：改了左列，同一提交里更新右列。表外的改动也要搜一遍引用它的代码和文档。

| 改了什么 | 同一提交里更新 |
|---|---|
| 新增/删除工具 | `src/tools/descriptions/*.json`、`config/plugin_catalog.rs`、`tools/compose.rs` 的 `UNITS`、`docs/wiki/15-扩展指南.md` |
| 新增顶层模块或跨层引用 | `test_scripts/arch_dep_check.py` 的层序表、`docs/architecture.md` |
| Turn 字段/数据库迁移 | `rows.rs` 的 `TURN_COLUMNS` 与 `map_turn_row`（§3.1） |
| 打包资源、外部命令、发布平台 | `publish-release.yml` 打包步骤、`nix/package.nix`、`nix/prebuilt.nix`（§7.3） |
| 发版流程（`publish-release.yml`、`.github/scripts/release.py`、`nix/update-release.py`） | 本文件 §7.2、`docs/wiki/18-Nix安装开发与发布.md` §4 |
| CI 结构（`.github/workflows/ci.yml`） | 本文件 §7.7 |
| 有意删除测试用例 | `test_scripts/.test-count` |
| 搬动/改名/删除本文件提到的路径或符号 | 本文件（`test_scripts/check-agents-refs.py` 会在 CI 里拦） |
| 用户可见的改动（验收通过后） | `CHANGELOG.md` 的 `[Unreleased]` |

---
> Source: [yxxbc/gqy-agent](https://github.com/yxxbc/gqy-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
