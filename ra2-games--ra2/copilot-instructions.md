## ra2

> 本文件适用于整个仓库。项目在 v86 中运行 RA2/YR 原版程序，不启动 Windows。

# 仓库协作规范

本文件适用于整个仓库。项目在 v86 中运行 RA2/YR 原版程序，不启动 Windows。
长期规则放这里；实现说明放 docs；历史记录不作为执行指令。

## 开始工作

- 内部开发与贡献基于 dev，不将 dev 推送到 GitHub；外部贡献向 main 提 PR。

- 先查看 Git 状态，保留已有改动；不要覆盖或顺手提交无关文件。
- 包管理只用 pnpm。不固定 Node 版本，pnpm 版本以 package.json 的
  packageManager 为准；工作区使用根 pnpm-lock.yaml，relay 独立构建另维护
  packages/relay/pnpm-lock.yaml；共享依赖变更须同步检查两份锁文件，使用 pnpm install --frozen-lockfile。
- 改架构前读 [架构要求](docs/ARCHITECTURE_REQUIREMENTS.md) 和 docs/ARCHITECTURE.md；
  改测试前读 docs/TESTING.md。
  UI、联机、hook、资源 CI 的专门入口见 docs/README.md，按任务阅读。
- 注释、维护文档和提交说明优先中文，代码标识符遵循现有模块约定。
  关键兼容行为解释原因、边界与证据，不重复翻译代码。

## 改动边界

- 实现或修改功能时持续区分通用机制与游戏策略，主动抽取可供其他游戏复用的能力。
  通用能力按职责放入 utils、resources、graphics 或 platform，不在工具层硬编码具体
  游戏的文件名、规则或地址；差异通过配置、契约或游戏模块注入。utils 不反向依赖业务层，
  抽取时同步迁移测试、文档和资源加载路径，并保留原有行为与生命周期所有权。
- 通用 vm86 不依赖具体游戏或浏览器；通用层不提供具体游戏的文件聚合入口，RA2/YR
  固定地址、ABI 与补丁放各自游戏模块；不扩大架构测试白名单来规避分层。
- 文件契约与纯 provider 归 resources，浏览器后端归 platform/browser/files；
  会话所有权归 app/session，呈现调度归 graphics。详细依赖以架构文档和测试为准。
- 普通网页 UI 由单一 React 树管理。VM、WebGL、音频与高频输入不进入 React state；
  effect、Worker、计时器、端口和缓冲必须有明确的销毁所有者。
- 未实现的客体调用不能猜测成功。补丁必须校验目标版本/指令签名并保持 RA2/YR 隔离；
  不用跳过判负、伪造资源、模拟输入或修改时钟让测试通过。
- 保留文件“未知、缺失、零字节、读取失败”的区别和覆盖优先级。
  INI 设置只改会话 overlay；禁止 transfer 客体 WASM 内存或共享 EXE 缓存。
- 主线程与 Worker 使用相同游戏策略。实验 SR 仅在开发入口加载，生产不能引入 ORT
  或实验模型 Worker。性能结论须有同场景对照，不将微基准等同于整局 FPS。

## 验证与交付

- 代码改动至少运行 pnpm run check，并按 docs/TESTING.md 追加对应回归。
  纯文档改动核对命令、源码路径与本地链接；不能用历史测试数字冒充当前验证结果。
- 修改 src/vm86/boot.asm 后运行 pnpm run build:boot，确保生成的 boot.bin 同步。
- 公共准入不得依赖私有素材或隐式下载 EXE。真实游戏测试缺资源、跳过、崩溃、
  超时都不能算通过；单独复测通过也不能抹去此前失败。
- 真实游戏 CI 的信任边界和配置见 docs/REAL_GAME_CI.md；未配置 runner 时不能
  宣称远端 CI 已启用。双端短局不证明公网长局或 4/8 人完整对局可靠。
- CI 统一放 .github/workflows，使用 ubuntu-latest。无素材准入接受 dev/main PR；真实游戏
  按 RA2/YR 分别下载资源，在 dev/main push 后运行；格式通过不能代替重型验收。
- 游戏包、EXE、模型权重、截图和本机缓存不入库；保留已有第三方许可与来源说明。
- 未获明确要求不 commit、push、部署或上传资源；交付说明实际验证及未验证范围。

## 文档维护

- CLAUDE.md 是指向 AGENTS.md 的符号链接，只维护本文件。

- 改接口、目录或命令时同步维护对应文档，版本和脚本以仓库配置为唯一来源。
- 当前指南不混入逐次提交日志、本机临时路径或旧“待办”；历史实验只有在仍有维护价值时
  才保留，并显式标明基线与时效。移动文件时修正维护文档链接，不删除逆向证据。
- 文档与代理说明不写外部域名或外链。来源使用项目名、版本或标签和源码路径；
  连接示例使用环境变量或本地回环地址。不要因此改动运行时配置或第三方许可原文。

---
> Source: [ra2-games/ra2](https://github.com/ra2-games/ra2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
