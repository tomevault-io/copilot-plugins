## nfcx

> 本文件适用于整个 NFCX 仓库。它既是项目背景说明，也是后续开发代理必须遵守的架构和实现约束。

# NFCX 项目开发指南

本文件适用于整个 NFCX 仓库。它既是项目背景说明，也是后续开发代理必须遵守的架构和实现约束。

## 项目背景

NFCX 是一个面向 Windows、macOS 和 Linux 的桌面 NFC 工具，目标是提供类似 MifareOne Tool 的图形化操作体验，同时避免只支持 Windows、只支持单一串口转换器或要求用户手工安装大量命令行工具。

项目最初使用 PN532 + FT232RL，通过 PN532 UART/HSU 模式完成验证。`scripts/` 中的 Python 程序已经证明当前硬件链路可以：

- 检测 PN532 和卡片；
- 读取 MIFARE Classic 1K；
- 将卡片保存为原始 dump；
- 将 dump 写回卡片；
- 检查 BCC、访问控制位并进行写后校验。

这些脚本是协议验证和硬件回归工具，不是正式桌面应用的运行时依赖。不要简单地把脚本里散落的 PN532 十六进制帧逐行翻译成 Go。

NFCX 只应用于用户拥有或明确获准测试的卡片和系统。

## 产品方向

NFCX 的定位是“小而完整的桌面工具”，不是通用 NFC 研究框架。优先保证：

1. 三个平台上的一致 GUI；
2. PN532 UART + FT232RL 开箱可用；
3. 普通读写稳定且不会轻易写坏卡片；
4. libnfc 支持的其他读卡器能够逐步接入；
5. 复用成熟的 MIFARE Classic 密钥恢复工具，不在 Go 中重新实现破解算法。

第一阶段以 MIFARE Classic 1K 为重点。除非当前任务明确要求，否则不要为了支持更多卡型而扩大实现范围。

## 固定技术栈

- Go：**1.25.1**；`go.mod` 必须以此版本为项目基线。
- 桌面框架：Wails v2。
- 前端：Vanilla TypeScript、HTML、CSS。没有明确收益时不要引入 React、Vue 或大型组件库。
- NFC 硬件抽象：libnfc。
- Go/libnfc 集成：CGO + 一个很薄的项目内 C shim。
- 破解引擎：以子进程方式调用 `mfoc`、`mfcuk` 等原生命令行程序。
- 串口：由 libnfc 对应驱动管理。正式主路径不要再单独维护一套 Go PN532 UART 协议栈。
- 测试：Go 标准测试、mock reader、固定响应/dump fixtures；真实硬件测试必须显式启用。

不采用 Electron。除非用户明确改变技术路线，否则不要迁移到 Tauri，也不要让 Rust 成为构建依赖。

## 总体架构

```text
Wails GUI
    |
Go application services
    |-- DeviceManager
    |-- CardService
    |-- DumpService
    |-- KeyService
    `-- AttackManager
          |
          |-----------------------------|
          |                             |
    in-process libnfc              external processes
    through CGO shim               mfoc / mfcuk / others
          |                             |
          `-------------+---------------'
                        |
                    NFC reader
```

必须保持下面三层边界：

1. GUI 只调用 Go application service，不直接接触 CGO、libnfc 或 `os/exec`。
2. 普通 NFC 操作通过项目内的 `Reader` 接口完成。
3. 破解算法通过 `AttackEngine` 接口运行外部程序。

## 哪些功能链接 libnfc

以下功能在 NFCX 主进程内通过 CGO 链接 libnfc，并通过 C shim 调用其 API：

- 枚举 libnfc 设备和连接字符串；
- 打开、初始化和关闭读卡器；
- 检测读卡器状态；
- 选择 ISO/IEC 14443A 卡片；
- 获取 UID、ATQA、SAK 等卡片信息；
- MIFARE Classic Key A/Key B 认证；
- 读取单个 block；
- 写入单个 block；
- 已知密钥的逐扇区扫描；
- 普通卡整卡 dump；
- 普通卡 dump 恢复；
- 写入后的逐块回读验证；
- CUID 等允许使用普通写命令修改 block 0 的操作，但必须经过危险操作保护。

上层不应直接使用 `nfc_context`、`nfc_device`、`nfc_target` 或 libnfc 的 union/struct。它们只能出现在 `internal/nfc/libnfc` 或等价的封装目录中。

推荐的 Go 接口形态：

```go
type Reader interface {
	Open(ctx context.Context, connString string) error
	Close() error
	CardInfo(ctx context.Context) (CardInfo, error)
	Authenticate(ctx context.Context, block byte, keyType KeyType, key Key) error
	ReadBlock(ctx context.Context, block byte) ([16]byte, error)
	WriteBlock(ctx context.Context, block byte, data [16]byte) error
}
```

C shim 只暴露 NFCX 实际需要的窄接口，并将 libnfc 错误转换为稳定的 NFCX 错误码。不要在 Wails binding 或业务层直接写 `import "C"`。

## 哪些功能调用外部程序

以下功能不链接进 NFCX 主进程，而是通过 `os/exec` 启动对应平台的外部二进制：

- `mfoc`：存在至少一个已知密钥时执行 Nested 密钥恢复；
- `mfcuk`：尝试通过 Darkside 流程恢复第一把密钥；
- `mfoc-hardnested` 或选定的等价工具：未来的 Hardnested 支持；
- `nfc-mfsetuid` 或 NFCX 维护的兼容版本：需要特殊后门序列的 Gen1A/UID 卡操作；
- 其他专用密码分析工具：只有在定义了明确适配器后才允许加入。

Windows 上这些文件通常带 `.exe`；macOS 和 Linux 使用对应的本地可执行文件。代码和文档中不要把外部引擎统称为 Windows EXE。

不要通过 shell 拼接命令。必须使用 `exec.CommandContext` 和参数数组，避免路径空格、转义差异和命令注入：

```go
cmd := exec.CommandContext(ctx, executablePath, args...)
```

外部程序必须通过统一接口接入：

```go
type AttackEngine interface {
	Name() string
	Available(ctx context.Context) error
	Run(ctx context.Context, request AttackRequest, emit func(AttackEvent)) (AttackResult, error)
}
```

不要让 GUI 了解 `mfoc` 或 `mfcuk` 的具体参数格式。

## libnfc 和外部引擎的设备所有权

一个读卡器同一时间只能由一个操作持有。普通扫描、读写和外部破解任务共享一个全局 `DeviceManager` 与设备锁。

启动外部破解程序时必须按顺序执行：

1. 停止 GUI 后台轮询；
2. 关闭主进程内的 libnfc `nfc_device`；
3. 使用选中的准确 connstring 启动外部程序；
4. 实时转发 stdout/stderr，并支持取消；
5. 等待进程退出并校验结果文件；
6. 无论成功、失败还是取消，都清理临时状态；
7. 重新打开读卡器；
8. 使用主进程内的 libnfc 验证恢复出的密钥。

不得让 in-process libnfc 和外部工具同时访问同一设备。

外部进程应继承同一设备选择，例如通过进程环境设置 `LIBNFC_DEVICE=<connstring>`。不要修改用户的系统级 `/etc/nfc` 或全局 libnfc 配置。

## 外部程序输出处理

所有外部任务都要保留原始 stdout/stderr 供 GUI 日志查看，但不要把自然语言输出当成唯一成功依据。

- 首先检查退出码；
- 检查预期输出文件是否存在；
- 验证 dump 大小、结构、sector trailer 和访问控制位；
- `mfoc` 的最终密钥优先从输出 dump 中读取；
- 只有 `mfcuk` 等没有结构化结果的工具才解析少量稳定文本；
- 如果上游输出不稳定，优先维护一个很小的 fork，增加 `NFCX_RESULT ...` 机器可读行，而不是复制整个算法到 Go。

外部任务的取消必须真正终止进程，并在需要时处理它创建的子进程。不要只在 GUI 上隐藏进度窗口。

## libnfc 构建和分发

主程序的 CGO 封装和外部破解工具必须尽量使用同一固定版本、同一驱动配置的 libnfc。不要在 CI 中不加锁定地追踪上游最新提交。

优先启用的驱动：

- `pn532_uart`：首要目标，覆盖 PN532 + FT232RL；
- `pn53x_usb`；
- `acr122_usb`；
- `acr122_pcsc`/`pcsc`：按平台构建能力启用；
- 后续驱动只有经过测试才在 GUI 中宣称支持。

“libnfc 能识别”不代表“支持所有破解功能”。设备能力至少区分：

- 可枚举；
- 可寻卡；
- 可进行 MIFARE Classic 普通读写；
- 支持 Nested；
- 支持 Darkside；
- 支持 Hardnested；
- 支持特殊 UID/后门命令。

安装包预计包含：

```text
runtime/
  windows-amd64/
    libnfc.dll
    required runtime DLLs
    mfoc.exe
    mfcuk.exe
  darwin-arm64/
    libnfc.dylib
    mfoc
    mfcuk
  linux-amd64/
    libnfc.so
    mfoc
    mfcuk
```

使用每个平台的原生 CI runner 构建 CGO、Wails、libnfc 和外部引擎。不要默认依赖从 Linux 交叉编译所有平台。

发布优先级：

1. macOS arm64；
2. Windows amd64；
3. Linux amd64；
4. 经过需求验证后再增加 macOS amd64、Windows arm64、Linux arm64。

macOS 发布时，嵌套的 dylib 和外部可执行文件必须先签名，再签名并公证整个应用。Windows 发布不要使用 UPX 压缩，以降低安全软件误报概率。Linux 优先提供 AppImage 和 tar.gz。

## GUI 功能范围

第一版 GUI 至少包括：

- 设备列表、刷新、手动 connstring 和连接状态；
- 卡片放入/移除状态；
- UID、ATQA、SAK 和识别出的卡型；
- 扇区/block 十六进制查看；
- Key A/Key B 输入、导入和逐扇区状态；
- 默认密钥字典扫描；
- 整卡读取并保存 `.bin`/`.mfd`；
- 加载 dump；
- 单块、单扇区和整卡写入；
- 写后验证及失败 block 标记；
- `mfoc` 和 `mfcuk -> mfoc` 工作流；
- 可取消的任务、进度状态和原始日志；
- 危险写入的明确确认和默认保护。

普通读写完成并稳定之前，不要优先开发高级破解界面。

## Dump 与密钥规则

- MIFARE Classic 1K 原始 dump 为 1024 字节；4K 为 4096 字节。
- 优先保持与现有 `.bin`/`.mfd` 工具兼容。
- NFCX 自有元数据存放在并列的 JSON 文件中，不向标准 raw dump 添加私有字节。
- 元数据可以记录 UID、ATQA、SAK、设备、读取时间、每扇区使用的 Key A/Key B 和失败信息。
- 日志默认隐藏完整密钥；只有用户明确选择导出或显示时才展示。
- 临时 dump 和 key 文件放在任务专用临时目录，任务结束后清理。

## 写卡安全约束

任何写入实现都必须包含：

- 文件长度和卡片容量检查；
- block 0 BCC 检查；
- sector trailer 访问控制位互补关系检查；
- 写入前认证；
- 写后立即回读验证；
- 失败后停止或按明确策略继续；
- 将 sector trailer 安排在普通数据块之后；
- block 0 默认禁止写入；
- 卡片 UID 或类型在任务中途变化时立即中止。

不得用普通“一键写入”绕过这些保护。Gen1A、CUID、FUID、UFUID 等操作要明确区分，不要假设所有“UID 卡”行为相同。

## 错误和并发模型

- 所有可能阻塞的 NFC 操作都接收 `context.Context`。
- 同一设备上的操作必须串行化。
- GUI 关闭、用户取消、读卡器拔出和卡片移除都要有明确错误类型。
- 不要用字符串匹配区分内部错误；定义可判定的 Go 错误或错误码。
- libnfc 回调或 C 指针不得在未明确管理生命周期的情况下跨 goroutine 使用。
- GUI 事件只传递可序列化 DTO，不传递内部 reader、C 指针或文件句柄。

## 建议目录结构

```text
cmd/nfcx/                 Wails 入口
app/                      Wails bindings 和 GUI DTO
frontend/                 Vanilla TypeScript GUI
internal/nfc/             Reader 接口、领域类型和 mock
internal/nfc/libnfc/      CGO 与 C shim；唯一允许 import C 的区域
internal/mifare/          Classic 几何结构、访问位、dump 逻辑
internal/device/          DeviceManager 和设备能力
internal/attack/          AttackEngine 与各工具适配器
internal/workflow/        dump、restore、key recovery 工作流
internal/runtime/         平台资源定位和动态库处理
scripts/                  已验证的 Python 硬件诊断脚本
testdata/                 合法的脱敏 fixture
build/                    Wails 平台构建资源
```

目录可以在实现过程中微调，但不得破坏架构边界。

## 测试要求

不连接真实硬件时，CI 也必须能覆盖：

- MIFARE 1K/4K block 与 sector 计算；
- BCC；
- sector trailer/access bits 编解码和校验；
- dump 长度和格式验证；
- Key A/Key B 选择；
- 外部程序参数生成；
- stdout/stderr 分片读取；
- 取消和超时；
- mfoc/mfcuk 示例输出解析；
- 设备锁和关闭后重开流程；
- Wails binding DTO 序列化。

真实硬件测试必须使用显式的 build tag 或环境开关，并且绝不能在普通 `go test ./...` 中自动写卡。

## 实施顺序

默认按以下顺序推进：

1. 初始化 Go 1.25.1 module 和 Wails GUI；
2. 固定并构建 libnfc；
3. 完成 C shim 和 Go `Reader` 封装；
4. 完成设备枚举、连接和寻卡；
5. 完成已知密钥认证、读块和写块；
6. 完成 dump、restore 和写后验证；
7. 完成密钥字典扫描和 GUI 数据表；
8. 完成设备所有权与外部任务框架；
9. 接入 `mfoc`；
10. 接入 `mfcuk -> mfoc`；
11. 增加特殊 UID 卡功能；
12. 建立三平台 CI、签名和发布包。

## 许可证和来源

libnfc、nfc-tools、mfoc、mfcuk 及其修改版可能使用不同的 LGPL/GPL 条款。引入或分发任何二进制时必须：

- 记录准确的上游仓库、版本和 commit；
- 保留版权与许可证文本；
- 提供许可证要求的对应源码或获取源码方式；
- 记录 NFCX 自己对上游的补丁；
- 不直接复制来源不明的 MifareOne Tool 发布包二进制。

不要在没有确认许可证的情况下把第三方二进制提交进仓库或发布包。

## 开发纪律

- 优先复用成熟的 libnfc 能力，不重新实现完整 PN532 协议。
- MIFARE 命令字节必须集中定义并带名称，禁止在业务和 GUI 代码中散落魔法数字。
- 先实现完整的纵向工作流，再扩大卡型或设备范围。
- 保留并尊重用户已有修改；不要进行无关的大规模重构。
- 修改 C shim、动态库布局或外部工具参数时，必须同时更新对应测试和本文档。
- 新增功能要明确属于“in-process libnfc”还是“external engine”，不能同时维护两套默认实现。
- 每完成一个功能，都必须在 `docs/notes/` 新增一份 Markdown 实施记录，说明实际完成了什么、实现过程中遇到了哪些问题，以及是否仍有被阻塞或未完成的事项；即使没有问题或阻塞，也要明确写“无”。

---
> Source: [BennyThink/NFCX](https://github.com/BennyThink/NFCX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
