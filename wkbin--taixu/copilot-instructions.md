## taixu

> > 本文件为导航入口，默认载入。全部细节（技术栈 / 模块拓扑 / 调用链路 / 文件索引 / 铁律 / 命令）在 [`docs/AI_NAVIGATION.md`](docs/AI_NAVIGATION.md)，按需读取。

# 🧭 太墟 (TaiXu / LinuxAIRuntime) — AI 导航入口

> 本文件为导航入口，默认载入。全部细节（技术栈 / 模块拓扑 / 调用链路 / 文件索引 / 铁律 / 命令）在 [`docs/AI_NAVIGATION.md`](docs/AI_NAVIGATION.md)，按需读取。

## 是什么

Android 无 Root 下用 PRoot 跑 Linux 多发行版沙箱 + AI Agent Harness + 原生 PTY 终端。Kotlin 2.4 / Compose / Hilt / Room / Navigation3，仅 `arm64-v8a`。

## 模块

`app`(壳/装配/JNI) · `core`(model·common·database·datastore·network·security) · `runtime`(PRoot/RootFS/PTY/工作区) · `tools`(Registry/安装事务/Provider安全) · `harness`(Agent循环/MCP/子智能体) · `feature`(components·theme·home·chat·terminal·workspace·settings·developer·welcome·navigation)

## 动手前先读

| 要做什么 | 读 `docs/AI_NAVIGATION.md` 的 |
| :--- | :--- |
| 写代码 / 改模块 | §2 模块拓扑 + §2.5 跨模块要点 |
| 追踪数据流 / 调用链 | §3 高频调用链路 |
| 定位某个类 / 文件 | §4 关键文件速查表 |
| 构建 / 测试 / 打包 | §6 常用命令（**必读**） |
| 避免架构坑 | §5 架构铁律（**必读**） |

---
> Source: [wkbin/taixu](https://github.com/wkbin/taixu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
