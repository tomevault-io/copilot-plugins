## p-pass

> issue（为什么做）→ 分支 → PR（怎么做的）→ 验收人 review + CI → squash 合入

# P-Pass 执行 agent 工作规范

唯一必读。一页以内，读完就开工。

## 流程（就这一条循环）

```
issue（为什么做）→ 分支 → PR（怎么做的）→ 验收人 review + CI → squash 合入
```

- **执行 session**（验收人给了 issue 号）：`git fetch origin --prune`，确认不落后
  远端，读该 issue 即全部任务上下文。issue 里 `requires:` 指到的文档才读，
  否则不读。**没有 QUEUE/PROGRESS/NEXT/cards 需要读——它们已全部退役。**
- **状态对齐**（问进度/下一个做什么）：看 GitHub Issues + milestone，
  不在仓里读任何状态文件。

## 红线（违反=事故）

1. **issue 外的活不做。** 发现新问题开新 issue 挂号，继续当前活，不顺手修。
2. **范围即边界。** 只动 issue「范围」段列出的文件；确需扩大时先改 issue 再改
   代码，同批 push。产品红线：只做图片，不做文件备份/同步。
3. **不捏造。** 报绿必须附命令 + 真实输出；故障/安全类判据必须带反证
   （去掉故障条件必须变红）。"CI 绿"不等于验收。
4. **凭据只在 GitHub Secrets。** 不进代码、issue、文档；本机路径/设备状态
   只写 `local-state.md`（不进 git）。
5. **红测不进 main。** 一 issue 一分支本身就是隔离：红测停下时把红留在自己
   分支上、别开 PR（或标 draft），并在 issue 里写明停在哪、剩什么。

## 验收证据分级

- **E1** 编译 + `just ci` 硬门——永远可信
- **E2** 针对当前架构 case matrix 写的 contract 测试——可信
- **E3** 真机/真环境实证输出（日志/截图/测试计数）——L3 必需
- **E4** legacy 测试——仅参考，不许单独作为验收通过依据

报绿时说明每条结论出自哪一级。

## 交付

- **一 issue 一分支，走 PR 合入，不直推 main。**
  `git fetch origin && git switch -c <type>/<issue号>-<slug> origin/main`
  （type ∈ feat/fix/docs/test/ci）。
- **分支上快验，PR 上全验。** 推分支不触发 CI（四条 lane 的 `push` 限定
  `branches: [main]`）：纯文档 `just ci-docs`，动了代码 `just ci`。
  开 PR 才跑 lane，分两档：
  - `ci-rust` / `ci-desktop` **workflow 层不设 `pull_request.paths`**
    （CI-07 #189）——它们的检查在 `main` 的必需列表里，而被 paths 跳过的
    workflow **根本不汇报状态**，必需检查会永久停在 Pending 并挡住合并，
    带 paths + 设必需 = 自锁。
    **但跳过被下放到了 job 层**（CI-11 #255）：一个 `changes` job 用
    `tools/ci-needs-full-lane.sh` 判断，纯文档改动时下游 job 被 `if:` 跳过。
    被 `if:` 跳过的 job **会汇报 `skipped`，而 `skipped` 对必需检查算通过**
    （PR #260 实测），所以既省资源又挡得住。
    ⚠️ 判据是 **allowlist**：只有改动**全部**落在无害清单里才跳过，其余一律跑。
    往清单里加东西前先确认没有测试或构建脚本读它（`assets/i18n/**` 被 diag
    测试消费，**不许加**）。
    判据有**两个 lane**（CI-17 #373），因为 6 个 job 的依赖面不同：**默认**
    lane（上面那份清单）给 `architecture enforcement` 与 `ci-desktop` 用
    （`arch-check.sh` 扫 `apps/**`，桌面壳门禁编的就是 `apps/desktop`）；
    **`rust-workspace`** lane 把 `apps/**` 也算惰性，给 `clippy` / `test` /
    `test (windows)` / `license+deny` / `fmt` 用（`src-tauri` 是独立 workspace，
    主 workspace 编不到它；`apps/` 的 Rust 由 `ci-desktop` 自己检查）。
    ⚠️ 不带 lane 参数 = 默认 lane，这是**向后兼容契约**；未知 lane 一律判要跑。
  - `ci-android` / `site` / `ci-docs` 仍按 paths 只在改到自己域时跑，
    它们不进必需列表，被跳过无害。
  - ⚠️ **任何 job 的 `name:` 都不许随手改**：`main` 的必需检查列表绑在名字上，
    改名 = 全部 PR 永久 Pending。改名和改分支保护必须在同一次操作里成对完成。
  - ⚠️ **进必需列表的 job，名字必须是写死的常量，不许用 `matrix` 现算。**
    `job 被 if: 跳过` 与 `name: xxx (${{ matrix.os }})` 这两件事**单独都对，
    凑一块儿就废**：跳过时 GitHub 不展开 matrix，只汇报一条名叫
    `xxx (${{ matrix.os }})` 的检查（模板原文），分支保护要的那几个名字一条
    都不来 ⇒ 必需检查永久 Pending。#240 加 Windows 矩阵、#261 加按需跳过，
    几小时内先后落地，探针 PR #271 才撞出来（纯文档 PR `BLOCKED`）。
    **同一次跳过里，名字写死的 job 全都正常汇报 `skipped`，一个没漏**——
    问题只出在「名字现算」这一个特征上。要跑多平台就拆成多个 job、`name:`
    各写死，步骤抽进 `.github/actions/<x>/action.yml` 共享（`ci-desktop` 的
    `desktop-linux` / `desktop-windows` 就是这么做的）。
    composite action 里每个 `run` 步骤**必须**显式写 `shell:`，且 actionlint
    默认不扫那个文件，语法错误要到 CI 真跑才暴露。
- PR 开出后盯受影响域 CI 到结论，红了在同一分支上修，不留红 PR。
- **带 GitHub MCP 的会话**（协作者账号 `690591397`）：自己把 PR 全程做完——
  推分支、开 PR（描述里逐项回接收尾检查）、盯 lane、squash 合并。
  未绑定该账号的 agent：推完分支停下，汇报分支名 + 自验结果 + 预填开 PR
  链接，开 PR 与合并由验收人在网页完成。
- PR 合并后清本地：`just cleanup-local`（默认预览，删除要 `--apply`）。
  本仓走 squash 合并，判断分支能否删看上游 `: gone]`，用 `-D` 不用 `-d`。
- tag 只给真发版本。调管线不发版走 Actions → Release → Run workflow。
- **release/tag 正文 = 面向用户的 changelog（红线级）**：正文以该版本的**用户可见变更**开头
  （对应 `CHANGELOG.md` 该版本段），**签名状态 / SHA-256 / 资产清单等构建元信息一律往后放**。
  为什么：release 正文会进 `manifest.json` 的 `notes`，而 **Android 应用内更新弹窗直接展示
  前 200 字**——那 200 字就是用户在手机上看到的更新说明（0.7.4–0.7.7 全是「构建自…签名状态…」，
  用户看不懂）。changelog 内容在 **commit/PR 阶段**维护进 `CHANGELOG.md` 的 `[Unreleased]`，
  publish 前落成正文；细则见 `docs/RELEASING.md` §3 / §3.5。
- 构建产物、日志不进 main。

## 机器兜底（`just ci` 会挡，不用背）

- iroh 只在 `crates/transport/`。
- 平台分叉只许在 `crates/platform/`：自己写 `#[cfg(unix)]` / `#[cfg(windows)]`
  这类按操作系统分叉的代码，出了 platform crate 门禁必挡——含 `cfg_attr`
  与 `cfg!` 形态，含 unix/linux/macos/target_family/target_env 全轴；扫
  `crates/` 与 `apps/`，不扫 `tools/`。判据（QA-09 #186）：框架抹平差异、
  暴露统一 API 的不算分叉；我们自写「A 平台这样、B 平台那样」才算。两个
  登记例外：`windows_subsystem` 链接器指令按**属性名**豁免；#211 挂号的
  存量（迁移中，arch-check 脚本 carve-out 销号后即删）。桌面壳不整体豁免，
  它的自写分叉同属 #211 迁移对象。
- 工具链版本唯一真相：`rust-toolchain.toml`（Rust）、CI `java-version`（JDK）。

## 索引（需要时才打开）

| 你要做什么 | 去哪 |
|---|---|
| 开 issue | GitHub issue 模板「任务卡」 |
| 本地跑测试 | `just --list` |
| 真机/模拟器、配对（无需摄像头）、本机 daemon IPC | 本机 `local-state.md`（不进 git，先读它再说"做不到"） |
| 发版、签名、版本纪律 | `docs/RELEASING.md` |
| 验收协议细则（L 分级由来、抽检法） | `docs/AGENT_PROTOCOL.md` |
| 历史事故与教训（出同类事故才翻） | `docs/lessons/` |
| 历史账本与旧卡（档案馆，非常驻阅读） | `docs/PROGRESS.md`、`cards/done/` |

---
> Source: [hawkeye-xb/P-Pass](https://github.com/hawkeye-xb/P-Pass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
