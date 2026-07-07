## ktuctl-go

> Go TUI/CLI 工具，通过 Generic Netlink 管理 **KTU Tunnel**（`tutuicmptunnel`）Linux 内核模块。不是 Rust 项目，忽略所有 Rust 相关的 skill/工具。

# AGENTS.md

## 项目简介

Go TUI/CLI 工具，通过 Generic Netlink 管理 **KTU Tunnel**（`tutuicmptunnel`）Linux 内核模块。不是 Rust 项目，忽略所有 Rust 相关的 skill/工具。

## 构建与运行

```bash
go build -o ktuctl-go       # 本地构建
./build.sh                   # 交叉编译所有 Linux 目标 → dist/
```

运行需要 **root** 权限（`sudo ./ktuctl-go tui`）——所有操作通过内核 netlink 通信。

无测试、无 linter、无 CI 配置。

## 架构

三个源文件，均在 `package main`：

| 文件 | 职责 |
|---|---|
| `main.go` | CLI 分发、netlink 辅助函数、所有内核结构体定义、子命令（`status`、`client-add`、`server-add`、`dump`、`script`、`tui` 等） |
| `tui.go` | Bubble Tea 模型——仪表盘 + 添加/编辑表单，复用 main.go 中的 netlink 辅助函数获取数据 |
| `uid_map.go` | 加载 `/etc/tutuicmptunnel/uids` 实现 UID↔主机名映射，通过 `sync.RWMutex` 保证线程安全 |

入口：`main.go` 中的 `main()`——先解析 flag，再分发子命令。

## 关键约束

- **C 结构体内存布局**：`main.go`（86–256 行）中所有结构体严格按字节对齐 C 内核结构体。填充字节用 `_ [N]byte` 显式声明。修改字段顺序、大小或填充会破坏 netlink 序列化，修改前必须对照内核头文件验证。
- **字节序**：网络字段用 `htons()`/`ntohs()`（手动位移，非 `binary.BigEndian`）。除网络序字段外，所有 netlink 载荷以 `binary.LittleEndian` 序列化。
- **TUI netlink 标志**：TUI 使用 `teaNetlinkFlags = 1 | 4`（Request | Acknowledge），定义在 `tui.go:575`。CLI 命令使用 `netlink.Request|netlink.Acknowledge`。
- **无 CGO**：构建使用 `CGO_ENABLED=0` 生成静态二进制（见 `build.sh`）。
- **XOR 混淆**：所有 peer 结构体（`UserInfoValue`、`EgressPeerValue`、`IngressPeerValue`）均携带 `XorKey [64]byte` + `XorKeyLen uint8`。CLI 参数为 `xor <hex>`，hex 长度必须为偶数，最大 128 字符（64 字节）。XOR 仅提供基础混淆，不替代加密。

## 子命令

`status`（无参数时默认）、`server`、`client`、`client-add`、`client-del`、`server-add`、`server-del`、`load`、`unload`、`dump`、`script`、`tui`、`reaper`（已废弃）、`version`。

全局 flag：`-n`（数字 UID）、`-d`（调试）、`-4`/`-6`（IP 协议族）、`-h`（帮助）。

## UID 映射文件

格式（`/etc/tutuicmptunnel/uids`）：每行 `UID 主机名`，`#` 开头为注释。启动时通过 `globalUIDMap.Load()` 一次性加载。

## Script 格式

`ktuctl-go script <file>` 读取行格式配置文件。一行内多条命令用 `;` 分隔。命令名与子命令相同（`client-add`、`server` 等）。

---
> Source: [hrimfaxi/ktuctl-go](https://github.com/hrimfaxi/ktuctl-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
