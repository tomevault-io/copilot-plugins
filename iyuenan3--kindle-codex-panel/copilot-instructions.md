## kindle-codex-panel

> 本文件是 Kindle Codex Panel 的 Codex 路由入口。项目的长期真相源位于 `AIREADME/`。

# AGENTS.md

本文件是 Kindle Codex Panel 的 Codex 路由入口。项目的长期真相源位于 `AIREADME/`。

## 当前状态

- 生命周期：active。真实 PW4 已安装 0.7.8，maxwell-mbp 与 yuenan-mbp 继续运行兼容的 0.7.5 服务代码。0.7.8 在三帧时域中值后加入端点安全的 DU4 灰阶限速，相邻帧每像素最多变化一个灰阶，不改变 Mac 协议、状态模型、刷新面积或帧率。多级灰阶跳变和纯黑白直接翻转均降为 0。164 项自动化和两次确定性包构建通过；PW4 安装返回 `success/completed`，`/mnt/us/kcp-reinstall-backup-0.7.8` 完整保留 0.7.7，当前与备份 KPM 数据库均为 `ok`，37 张完整与裁切庆祝帧的合并哈希和仓库一致，无自启动 job。用户决定保留约 3.2 FPS 的真实 DU4 完成栅栏，不再强行提速；0.7.8 真机 framebuffer 已更新到 README。
- Mac 服务采用 Python 3.9 标准库，无第三方运行依赖。
- 经过哈希固定的短时 Sanctuary 局域网镜像已完成真实设备任务并停止。
- `;log`、`JAILBROKEN.txt`、安装日志和 hard-float FBInk 已形成成功证据，调试标记、core dump 和填充文件已清理，正常重启后的持久性复核和设备能力探针均已完成。
- 首个内容基线已提交为 `d101a3b`，AIREADME 同步锚点已更新为该 SHA。
- GitHub 公开仓库为 `iyuenan3/kindle-codex-panel`，默认分支 `main`，远端已识别 Apache-2.0 许可证。
- 用户已明确授权并启用 maxwell-mbp 的项目 Hooks 与用户级 LaunchAgent；既有 Hooks 已备份。当前九类 Hook 均已在 Codex CLI `/hooks` 中显示为 Active 和 Trusted，真实 `PreToolUse`、`PermissionRequest` 与生命周期事件已进入面板并提供模型、模式和最近工具；`SubagentStart`、`SubagentStop` 的真实任务验收仍待进行。安装副本哈希与仓库一致，服务健康，`config.toml` 未修改。
- 调试 SSH 只绑定 `usb0`、禁用密码和 Wi-Fi，并使用 Dropbear `-F` 把日志留在已重定向的 stderr，避免污染 framebuffer。自动启动经两轮真实重启仍不能可靠保持 Dropbear，已撤回。人工 Start 启动维护壳层，屏上双击退出与 watchdog 都调用 Stop 并尝试恢复 USBMS；文档 Stop 继续作为独立回退。用户可见文档只保留 `KCP_Start`、`KCP_USB_SSH_Start` 和 `KCP_USB_SSH_Stop`，本次 0.6.3 userstore 安装包、runner、日志、结果与截图文件已删除，`/mnt/us/kcp-reinstall-backup-0.6.3` 保留 0.6.2。正式面板启动后立即停止 Dropbear，且 `/etc/init/kcp-panel.conf` 未安装。

## 启动顺序

1. 读本文件。
2. 读 `AIREADME/INDEX.md`。
3. 按任务路由读取对应 AIREADME 文件。
4. 开始实现前检查 Git 状态，保留用户已有改动。
5. 涉及 Codex 接口时重新核对 OpenAI 官方文档。
6. 涉及越狱时重新核对 KindleModding 对 PW4 5.18.1.1.1 的当前兼容说明。

## 任务路由

| 任务 | 必读 |
|---|---|
| 理解项目定位 | `AIREADME/CORE.md` + `AIREADME/PRD.md` |
| 设计或修改组件 | `AIREADME/ARCHITECTURE.md` + `AIREADME/DECISIONS.md` |
| 增加 HTTP 或事件接口 | `AIREADME/SPEC.md` + `AIREADME/ARCHITECTURE.md` |
| 设计 Kindle 页面 | `AIREADME/PRD.md` + `AIREADME/CONVENTIONS.md` |
| 接入 Codex Hooks | `AIREADME/SPEC.md` + OpenAI 官方 Hooks 文档 |
| 接入 App Server | `AIREADME/ARCHITECTURE.md` + OpenAI 官方 App Server 文档 |
| 本地运行与开机启动 | `AIREADME/DEPLOYMENT.md` |
| 越狱与 Kindle 原生客户端 | `AIREADME/CORE.md` + `AIREADME/DECISIONS.md` |
| 调整优先级 | `AIREADME/ROADMAP.md` |
| 发布版本 | `AIREADME/CHANGELOG.md` + `AIREADME/DEPLOYMENT.md` |
| 记录事故和踩坑 | `AIREADME/MEMORY.md` |

## 已确认设备基线

- Kindle Paperwhite 第 10 代，PW4。
- 固件 5.18.1.1.1。
- 该设备带前光，用户此前长期关闭，因此曾记成没有灯。
- 浏览器 PoC 已完成基础验证。
- Sanctuary 公网直连曾卡在 `Loading into sql.js...`，改用 Mac 短时局域网镜像后已成功越狱。
- 当前原生扩展路线是预装 KPM 加 `kindlehf` 包，不使用已废弃的 KUAL 或旧 `kindlepw2` 包。
- 一次性能力探针确认 KPM、FBInk、Upstart、curl、wget、LIPC 和 SQLite 可用；文档 scriptlet 不作为长期客户端入口。
- KPM 0.1.4 静态预览的直接 launcher、重复启动保护、KPM CLI launch 和真实 `SIGKILL` watchdog 恢复均已通过。项目包 0.2.6 的抱枕 DONE、蜷卧 FAILED 与原睡姿 OFFLINE 已通过用户目视确认。0.2.10 的仪表盘、分钟时钟局刷、常亮、窗口所有权和正文触控隔离已通过真机验收。0.7.0 保留 KindleOS、Xorg、KPP、前光、电源和网络服务，以全屏 Application 窗口替代前台界面；0.7.1 把正文更新拆成固定区域；0.7.2 增加顶层任务索引过滤与双层级底部时间；0.7.3 增加物理电源键手动休眠，并在休眠期间暂停轮询和全部绘制。以上版本继续由同一窗口所有权门禁保护，均已通过真实 PW4 验收。
- 已注册且 Kindle Store 可用时，SpringBreak 是备选。
- WinterBreak 和 WinterBreak2 不适用于这台设备的当前固件。

## 红线

- 不把 Kindle 伪装为实时 macOS 扩展桌面作为主路线。
- 不解析 Codex transcript 作为稳定接口。
- 不向局域网暴露 Codex App Server、`auth.json`、环境变量或凭据。
- 不显示完整提示词、内部推理、完整终端输出或私密文件内容。
- 不在未经用户确认时修改全局 `~/.codex/hooks.json` 或 `config.toml`。
- 不自动执行 Kindle 越狱、恢复出厂设置、升级、降级或 OTA 操作。
- 不使用针对其他 Kindle 型号或固件的包。
- 不把本地面板直接开放到公网。
- 不在文档、日志、截图或测试夹具中写入完整设备序列号。

完整边界见 `AIREADME/CORE.md`。

## 实现原则

- 浏览器回退版必须继续保持不越狱可运行。
- Mac 端负责状态归并和视觉渲染，Kindle 端保持轻量。
- Hooks 是核心生命周期状态源，并提供 allowlist 内的模型、权限模式、最近工具与子代理生命周期；公开模型只保留 `SUBAGENTS` 当前活跃数、本 turn 开始数、停止数和类型汇总，不发布代理 ID。App Server 只在 Mac 本机提供当前账号、额度和会话标签。标签优先使用任何非空 `thread.name`，只有名称缺失时才保留脱敏且有界的 `preview` 首行摘要，其他增强后置。
- Hook 命令从 stdin 接受 JSON，必须快速、静默、失败不阻塞 Codex。
- 对外只发布脱敏后的状态模型。
- 内容未变化时不触发电子墨水刷新。
- 状态过期时明确显示 offline 或 stale，不能继续展示成实时状态。
- 原生 Kindle 包必须面向 `kindlehf`，不能误用旧 `kindlepw2` 二进制。
- 浏览器版先测实际 viewport，不凭硬件分辨率硬编码页面尺寸。
- 前光控制、开机启动和省电策略在浏览器 PoC 之后处理。

## 目标状态模型

首版优先支持：

- `offline`
- `idle`
- `working`
- `waiting_approval`
- `done`
- `failed`

新状态必须先更新 `AIREADME/SPEC.md`，再修改实现。

## 验证要求

- 单元测试覆盖事件归并、脱敏、状态过期和修订号变化。
- HTTP 测试覆盖健康检查、页面、图片和非法 token。
- 浏览器 PoC 必须在真实 PW4 上人工验收。
- 越狱版必须单独验证全刷、局刷、休眠恢复和断网行为。
- 真实 Kindle 验收不能由桌面浏览器截图替代。
- 测试通过不代表 OTA、功耗或长时间残影已经验证。

## 文档维护责任

- 定位或红线变化：更新 `AIREADME/CORE.md`。
- HTTP、事件或状态模型变化：更新 `AIREADME/SPEC.md`。
- 组件、数据流或安全边界变化：更新 `AIREADME/ARCHITECTURE.md`。
- 本地服务、launchd 或 Kindle 自启动变化：更新 `AIREADME/DEPLOYMENT.md`。
- 重大取舍：向 `AIREADME/DECISIONS.md` 追加 ADR。
- 实际故障或踩坑：向 `AIREADME/MEMORY.md` 追加记录。
- 里程碑或 release：向 `AIREADME/CHANGELOG.md` 追加版本块。
- 优先级变化：更新 `AIREADME/ROADMAP.md`。

## 常用检查

```bash
git status --short
bash "$HOME/.agents/skills/aireadme/scripts/check.sh" AIREADME
```

当出现首个真实内容 commit 后，更新 `AIREADME/INDEX.md` 的 `last-synced`，不要继续保留 `pre-code`。

## 中文写作

- 使用中文标点。
- 不使用中文破折号。
- 输出只保留会改变判断或行动的信息。

---
> Source: [iyuenan3/kindle-codex-panel](https://github.com/iyuenan3/kindle-codex-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
