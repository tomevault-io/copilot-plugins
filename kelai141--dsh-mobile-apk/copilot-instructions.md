## dsh-mobile-apk

> > **本文件是你进入壳侧的入口**：仓库地图 + 去哪查 + 更新协议。不堆历史记录。

# AGENTS.md — dsh-mobile-apk 项目地图

> **本文件是你进入壳侧的入口**：仓库地图 + 去哪查 + 更新协议。不堆历史记录。
> 一切以源码为准；发现文档与源码不一致，**当场改文档**。

---

## 1. 这是什么仓

DeepSeek Harness 的**安卓壳应用**（包名 `com.dsharnessmobile.shell`）。

**职责边界**：只保留安卓平台权能与桥——前台服务 / 看门狗 / WebView（主 + 隔离 BrowserHost）/ SAF 桥 / 快照解压 / UndoGate / Shizuku 特权 transport 与虚拟屏 / 无障碍与 ADB 授权 / 通知中心与通知内应答 / 返回网关 / 审计 / 控制台 / 日志。

**关键约束：AI 可见的能力全部来自插件**，壳侧不直接注册工具。

**运行时**：内嵌 Termux 快照（`assets/snapshot.tar.xz` → `files/usr` + `files/home`）；引擎 `@deepseek-ai/dsh` 0.1.5-rc.1 监听 `127.0.0.1:3080`；WebView 加载引擎 Web UI。

**构建链**：minSdk 26 / targetSdk 34 / compileSdk 36；Kotlin 2.0.21；AGP 8.8.2；Java 17。

**兄弟子仓**（本仓内含自包含副本，见 §4 同步铁律）：
`dsh-shell-termux` 0.2.0 · `dsh-client-ui-responsive` 0.3.3 · `dsh-host-web-compat` 0.1.13 · `plugins/`（bridge 0.2.4 / manage 0.3.0 / model-capability 0.2.1 / file-open 0.1.0 / browser 0.1.0 / linux-env 0.1.2 / vdisplay 0.1.0）· `vendor/`（marketplace / undo-savepoint / dsh-model-sync）

**上游** `deepseek-ai/deepseek-harness`（协调仓 `dsh/` 只读 checkout）：**零改动**，一切适配走补丁/插件/壳侧。

**当前状态：以 `docs/AGENTS/changelog-archive.md` 最新一行为准**（本文件不维护版本号，避免双份漂移）。

---

## 2. 怎么建、怎么装、怎么验

```powershell
# 在【协调仓根】执行（壳侧不单独构建）
pwsh -File scripts\build-apk-013.ps1 -Suffix ""   # 双 ABI 全链，门禁失败即拒打包
pwsh -File scripts\build-apk-013.ps1 -Fast         # dev 档：单 ABI x86_64 + preset 1（禁发布）

# 装机（<serial> 用 adb devices 查；arm64 真机必须用 arm64 产物）
adb -s <serial> install -r -t out\v<版本>\dsh-mobile-apk-v<版本>-arm64.apk
```

**设备**：
- MuMu 模拟器 `127.0.0.1:16416`（竖屏）、`127.0.0.1:16384`（横屏，真 1600x900）——开发循环主用。
- 真机 V2425A `10AF2B0GN0001F2`（arm64，1260x2800，SDK 36）——**发布前**补充门禁。

**验证脚本**（`scripts/`，必须**逐个单独跑**，每个都重建 WebView target）：

| 脚本 | 用途 | 参数约定 |
|---|---|---|
| `verify-webview-015.mjs` | WebView / polyfill / 内联脚本 / 面板 | ws **positional**；`--wide` 横屏 |
| `verify-state-sync.mjs` | 跨层状态同步真源用例集 | **不要传 `--ws`**（会禁用 target 重解析） |
| `verify-browser-host.mjs` | 隔离浏览器视口/PC 身份 | ws **positional** |
| `verify-vdisplay-viewer.mjs` | 虚拟屏 viewer 两阶段契约 | `--ws` |
| `verify-browser-panel.mjs` / `verify-vdisplay-float.mjs` / `verify-engine-log-copy.mjs` | 面板/浮窗/日志 | 见脚本头注释 |

**CDP 调试**：`adb shell "cat /proc/net/unix | grep webview_devtools"` → `adb forward tcp:29225 localabstract:<socket>` → `ws://127.0.0.1:29225/devtools/page/<id>`。

**热重载（仅 JS 插件，不含 Kotlin）**：`node scripts/hot-push.mjs --serial <s> --plugin <dir> [--pkg ...] [--restart]`。

### 2.1 模拟器测试规范（强制；详档 `docs/AGENTS/emulator-test-protocol.md`）

1. **三层验收，缺一层即未验收**：**代码层**（Kotlin 单测 + 静态门禁 + 本改动自带的反证用例）、**CDP 层**（跑对应 `verify-*.mjs`，断言状态语义与跨层数据）、**ADB 用户实际体验层**（adb 真操作真截图：`input tap/swipe/text/keyevent` + `exec-out screencap -p`，证明一个真人能用这块屏幕完成同一件事）。
2. **任一层红 = 这个系统有问题**，且必落在下列至少一类——**不许当噪声，不许只修那一层**：
   1. 不符合人体交互逻辑（点不到、看不见、层级/遮挡/焦点/键盘错）；
   2. 在用户视角里就是坏的（界面显示正常，用起来不对）；
   3. 根本就是代码有问题；
   4. 内部没打通（跨层 / 跨仓 / 跨进程的数据断了，界面照样显示正常）。
   对照读法：CDP 全绿而 ADB 层失败 = 内部没打通；代码层绿而 CDP 红 = 接口与实现脱钩；两层绿而真人做不到 = 交互逻辑错。
3. **任务必须真实，且必须由模型自己编排**：验收给的是**目标 + 约束 + 成功标准**，不给操作序列；步骤由模型自己规划、自己纠错、最后汇报（例：让它自己去打开某个应用、在虚拟屏上玩一个小游戏、在网页应用里截图并发消息，然后汇报结果）。**禁止**把步骤一条条喂给它，**禁止**只做单步或两三步的简单操作——那证明不了编排、纠错与跨层串联，而缺陷恰恰藏在那些地方。过程按工具调用序列、失败重试与最终汇报留证。
   - **B 轨的现成驱动**：`node scripts/verify-screen-scope-matrix.mjs --serial <s>`（屏幕范围/通道/落点；它自己会向模型发起真实任务并只断言设备事实）。屏幕范围、Shizuku、虚拟屏、通道状态相关的改动**先跑它**，再按下方原语补人眼判读。
4. **只跑 CDP 不算验收**：几何/层级/遮挡/焦点/键盘这类缺陷 DOM 断言看不见（坑 50/61 是实锤）；反过来只贴截图也不算——状态语义只有 A 轨能证。两轨互补，缺一不可。
5. **证据落盘**：命令清单 + 每步截图（`.deploy-tmp/<round>/ui-NN-*.png`）+ 三层各自的结论行；PR 描述贴结论行与截图路径。**缺任一层 = 未验收**，不得写「功能完好」。
6. **覆盖两个方向**：竖屏 16416 与横屏 16384 各至少一次 tap 级操作 + 截图；几何/布局类改动两个方向都必跑。发布前 arm64 真机补一次。
7. **前置与假失败**：快照刷新期间禁跑设备验收；uid-mode appop 残留、prefs 僵尸 a11y 标记、应用被切后台都会制造假失败——清理方式与判据见详档 §3、§8。

---

## 3. 去哪查（按需 grep，别通读）

| 要查什么 | 去哪 |
|---|---|
| **坑位（历史实锤，权威）** | `docs/AGENTS/gotchas.md`（递增编号，允许空缺；grep 关键词） |
| 模块职责 / 函数位置 | `docs/AGENTS/modules.md` |
| 架构 / 依赖方向 | `docs/AGENTS/ARCHITECTURE.md` |
| 桥方法签名 / 通道语义 | `docs/AGENTS/BRIDGE-API.md` |
| 无障碍 API 与权限面 | `docs/AGENTS/ACCESSIBILITY-API.md` |
| 构建失败 / 门禁 / WSL 环境 | `docs/AGENTS/build-and-env.md` |
| 运行时补丁（`assets/patched`） | `docs/AGENTS/RUNTIME-PATCHES.md` |
| gradle 依赖与升级 | `docs/AGENTS/DEPENDENCIES.md` |
| GPL 合规三形态 | `docs/AGENTS/gpl-compliance.md` |
| 已知缺口 / 待办 | `docs/AGENTS/known-gaps.md` |
| 运行顺序 / 嵌套 / 耦合 / 流程图（排查入口） | `docs/AGENTS/EXECUTION-MAP.md` |
| 模拟器验收规范（代码/CDP/adb 三层 + 真实任务） | `docs/AGENTS/emulator-test-protocol.md` |
| **版本历史（全量）** | `docs/AGENTS/changelog-archive.md` |

---

## 4. 铁律（违反即返工）

1. **需求不延期**：有明确需求且未获准延后，本轮必须完成。技术阻塞只能给替代实现 + 记真因，不得以「实验特性 / 下版本」降级。
2. **模拟器优先**：本地构建 → **MuMu x86_64 实测** → 才谈 PR。禁止以「等真机」推迟模拟器实测，也禁止把模拟器实测挂在 PR 之后。真机是发布前补充门禁。改动必须过**三层验收**（代码层 + CDP 层 + ADB 用户实际体验层，缺一层即未验收）且验收任务是**真实任务、由模型自己编排**（禁止喂步骤），见 §2.1。
3. **批量改、统一验**：成组改动一次改完，再做一次构建 + 打包 + 装机 + 回归。中途只跑廉价检查（Kotlin 单测、静态门禁）。安全敏感项（快照事务 / 签名 / 门禁自身 / 桥协议面）例外，即时验。
4. **一切改动走 PR**：建分支 → 提交（`<type>: <描述>`）→ 推分支 → 开 PR（1-3 标签）→ CI 绿 → 合并 → 删分支。**禁止直接 push `main`**。例外仅限不改仓库内容的外部动作（Release 资产、issue 评论与开关、标签）。
5. **子仓镜像（幽灵缺陷防线）**：改 `dsh-client-ui-responsive` / `dsh-host-web-compat` / `dsh-shell-termux` / `plugins/dsh-android-*` 的源码或 bump 版本后，**必须 robocopy 镜像到本仓同名目录**（src + package.json + **`lib/` 产物**）。漏了 → 云端自包含构建注入旧副本，**编译通过但功能缺失**。
   - 只同步**镜像清单内的文件**，切勿对 `scripts/` 等目录整体 `/MIR`——会删掉本仓独有的设备脚本（真实事故）。
6. **补丁镜像面**：`scripts/patches/**`、`scripts/check-patch-mirror.mjs`、`scripts/build-apk-013.ps1` 等是协调仓的**逐字节镜像**，单边演进即拒。补丁改动**先合本仓镜像 PR，再合协调仓权威源 PR**。
7. **禁 emoji**：提交信息、PR 标题/描述、文档、标签一律不用。

---

## 5. 更新协议（改完代码必须做，否则文档失真）

> **核心原则：AGENTS.md 只是地图，不承载细节。** 任何细节写进 §5.2 的对应详档。
> 本文件**不设更新记录表**——所有版本记录进 `docs/AGENTS/changelog-archive.md`。

### 5.1 按改动类型对号入座

| 你改了什么 | 必须更新 | 位置 |
|---|---|---|
| **任何**代码/行为改动 | 坑位（若踩过坑或修了坑） | `docs/AGENTS/gotchas.md` **文末追加**，编号递增（允许空缺，**不得重编号**） |
| 版本发布 / 里程碑 | 更新记录（一行摘要，**倒序加在最上方**） | `docs/AGENTS/changelog-archive.md` |
| 新增/删除 `.kt` 文件，或函数职责变化 | 模块地图 | `docs/AGENTS/modules.md` |
| 模块边界、依赖方向变化 | 架构 | `docs/AGENTS/ARCHITECTURE.md` |
| 新增/改签名桥方法、通道语义变化 | 桥协议 | `docs/AGENTS/BRIDGE-API.md`（**同步跑** `scripts/check-bridge-symmetry.mjs`） |
| 无障碍 API 用法、权限面变化 | 无障碍参考 | `docs/AGENTS/ACCESSIBILITY-API.md` |
| 构建链、门禁、WSL 环境差异 | 构建与环境 | `docs/AGENTS/build-and-env.md` + `docs/AGENTS/gotchas.md` |
| 改 `scripts/patches/**` 或 `assets/patched` | 运行时补丁 | `docs/AGENTS/RUNTIME-PATCHES.md` §7.1 登记 + §2 字节数按实测更新 |
| gradle 依赖增删/升级 | 依赖台账 | `docs/AGENTS/DEPENDENCIES.md` |
| 发现未实现 / 有意留下的缺口 | 已知缺口 | `docs/AGENTS/known-gaps.md` |
| **代码执行顺序 / 嵌套 / 耦合 / 流程图（排查入口）** | 执行地图对应查点 + 覆盖账本 | `docs/AGENTS/EXECUTION-MAP.md`（改完跑 `node scripts/check-code-map.mjs`） |
| 改设备验收方式 / 新增套件或操作原语 | 模拟器验收规范 | `docs/AGENTS/emulator-test-protocol.md` |
| **本文件 §1-§4 的描述失真** | 就地改**对应行** | 只改那一行，**不要新增章节、不要加历史** |

### 5.2 写作纪律

- **细节不下放 §1-§4**：一行能说清就一行；说不清 → 写进详档，本文件只放**指针**。
- **坑位写「为什么」不只写「怎么做」**：现象 → 真因 → 修法 → 复验证据。无真因的条目等于没写。
- **同一事实只写一处**：其余地方用链接/路径指向它。重复即漂移源。
- **数字不写死**：坑位总数、门禁项数、文件数一律用命令现数（`grep -c '^[0-9]\+\. \*\*' docs/AGENTS/gotchas.md`、`node scripts/check-release-gates.mjs --list`）。
- **改了要跑**：文档改动本身不需要构建；但涉及门禁/桥面/补丁改动时，对应的 `check-*.mjs` 必须跑过再提交。

### 5.3 门禁（哪些是自动挡）

`scripts/check-release-gates.mjs` 是聚合入口（`--list` 现数）。关键几条与文档直接相关：

| 门禁 | 守什么 |
|---|---|
| `check-bridge-symmetry.mjs` | 桥面对称性（改桥方法必跑） |
| `check-api-route-auth.mjs` | 新增 `/api` 路由必须自带鉴权并登记 policy |
| `check-control-ops.mjs` | 控制 op 六面登记一致（新增 op 必须六处同改） |
| `check-kotlin-comments.mjs` | Kotlin 嵌套块注释 |
| `check-plugin-tests.mjs` | 每个插件的单测真实跑通（全 skip 判假绿） |
| `check-tool-surface-budget.mjs` | 模型面 wire 预算（新增工具会撞；近义工具优先复用既有 + 参数） |
| `check-patch-mirror.mjs` | 双仓镜像逐字节一致 |
| `check-code-map.mjs` | 执行地图覆盖完整、锚点有效、编号一致（改代码后必跑；`--self-test` 自检） |
| `check-op-registry-parity.mjs` | 引擎与壳侧两份「会读设备屏的 op」清单逐条相同（漂移 = 缺省 virtual-only 下工具大面积不可用） |
| `check-tool-name-promises.mjs` | 指引里承诺的 `android_*` 工具名必须有声明位（防「指引指向一条不存在的路」） |

**设备侧（不进聚合门禁，发布前由操作者按 §2.1 跑）**：

| 套件 | 守什么 |
|---|---|
| `scripts/verify-screen-scope-matrix.mjs` | 屏幕范围/通道/落点的设备级验收：跨面一致性（引擎面 vs 壳侧面 vs 设备事实）+ 拉起落点回读 + 双屏像素对照 + `real-only` 反证。判据全落在设备事实上；**证据不足判 `INCONCLUSIVE`（exit 2），不得当通过**。`--self-test` 自带 6 例判别力 |

---

## 6. 高频雷点（只有三条，全量见 §3 坑位表）

1. **ABI**：debug 包内嵌快照决定 ABI；x86_64 快照装 arm64 真机**必崩**。真机只用 `-arm64.apk`。
2. **快照刷新**：唯一完成标志 = `.snapshot-fingerprint` **翻转** + `.snapshot-transaction` **消失**。刷新期间（8-12 分钟）**禁 force-stop/杀进程**，且**不得跑设备套件**（会是假失败）。
3. **`Select -First N` 会杀掉长任务子进程**——构建/长脚本必须全量重定向日志，不能用 `Select -First`。

---
> Source: [kelai141/dsh-mobile-apk](https://github.com/kelai141/dsh-mobile-apk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
