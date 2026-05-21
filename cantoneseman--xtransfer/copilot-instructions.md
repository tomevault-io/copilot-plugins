## xtransfer

> XTransfer 是面向超算中心 TB 级互连的高性能传输底座。当前路线明确维持高性能：DPDK、HugePages、RUDP、pacing/RRCC、P-FEC、TC-SACK、io_uring、O_DIRECT。易部署靠显式 preflight 和清晰错误，不靠隐式降级。

# AGENTS.md

## 项目一句话

XTransfer 是面向超算中心 TB 级互连的高性能传输底座。当前路线明确维持高性能：DPDK、HugePages、RUDP、pacing/RRCC、P-FEC、TC-SACK、io_uring、O_DIRECT。易部署靠显式 preflight 和清晰错误，不靠隐式降级。

## 新对话启动顺序

除本文件外，项目级说明文档只保留三个，必须按顺序阅读：

| 顺序 | 必读文件 | 目的 |
|---|---|---|
| 1 | `README.md` | 了解项目入口、当前结论和最快验证方式 |
| 2 | `docs/PROJECT_STATE.md` | 了解技术路线、已完成工作、进度、风险、下一步 |
| 3 | `docs/ENVIRONMENT.md` | 了解 VMware、服务器、网卡边界和验证命令 |

以后不要再恢复 `PLAN.md`、`progress.md`、`findings.md`、`task_plan.md`、`lessons.md` 等重复文档；需要记录进度、风险、经验和下一步时，统一写入 `docs/PROJECT_STATE.md`。需要记录环境和命令时，统一写入 `docs/ENVIRONMENT.md`。

## 高性能红线

| 红线 | 说明 |
|---|---|
| 禁止隐式降级 | DPDK/HugePages/SQPOLL/O_DIRECT 等关键能力失败必须直接失败 |
| 禁止生产 Mock | Mock 只能用于本地调试，统一隔离到 `127.0.0.1:xxxx/mock`，不能进生产路径 |
| 禁止热路径堆分配 | 数据面热路径禁止 `new`/`malloc`，除非文档证明不在热路径 |
| 禁止热路径锁竞争 | 数据面优先 SPSC、atomic、per-core 数据结构 |
| 禁止无验证优化 | 性能优化必须能用测试或指标证明 |
| 禁止先做复杂多路径 | 单路径端到端没闭环前，不做复杂调度 |
| 禁止吞错继续 | I/O、协议、对齐、校验失败必须暴露 |
| 禁止假 DPDK 成功 | DPDK RX 必须看到 `rx_received>0 decoded>0 accepted>0 drained>0` |

## 开发顺序

| 阶段 | 原则 |
|---|---|
| 先状态 | 开始任务先看 `docs/PROJECT_STATE.md` 的红线、风险和下一步 |
| 再契约 | 先明确协议、buffer、chunk、session、task、resource 契约 |
| 再闭环 | 先打通单路径端到端 |
| 再性能 | 有真实指标后再调 pacing/RRCC |
| 再多路径 | 单路径稳定后才做 PathManager/Scheduler |

## 环境规则

| 环境 | 规则 |
|---|---|
| Windows 本地 | 只做编辑、文档和基础检查，不做真实性能验证 |
| VMware Linux `192.168.1.114` | 优先用于构建、单元测试、普通网络 smoke |
| 大服务器 `192.168.1.203` | 公共资源；用户 `Su`；项目 `/home/Su/projects/XTransfer`；只有 DPDK/双机/全功能明确需要时才用 |
| 小服务器 `192.168.1.112` | 公共资源；用户 `Su`；项目 `/home/Su/projects/XTransfer` 当前由 Windows 本地 `D:\Project\XTransfer` 通过用户态 SFTP 反向隧道 + `sshfs` 持续挂载；DPDK/编译依赖、HugePages、liburing 已配置 |
| 项目共享目录 | 大小服务器项目目录都映射到 Windows 本地项目；不要从 Windows 用 SFTP 把本地同一文件写回这些共享挂载路径，避免自拷贝截断 |
| 服务器管理网卡 | `eno12399` / `0000:31:00.0`，禁止绑定 DPDK、禁止 down |
| 双机 100G 链路 | 大服务器 `enp177s0np0` / `0000:b1:00.0` 为 `192.168.100.2/30`，小服务器 `enp130s0np0` / `0000:82:00.0` 为 `192.168.100.1/30`，ping 已通 |
| 大服务器 25G 候选口 | `enp178s0f0np0` / `0000:b2:00.0` 已接通，`carrier=1 speed=25000`，等明确 DPDK 实测窗口 |
| 小服务器 25G 候选口 | `enp5s0f1` / `0000:05:00.1` 已接通，`carrier=1 speed=25000`，NIC safety 通过；`vfio-pci` 绑定因 IOMMU 未启用失败，已恢复 `ice`；`enp5s0f0` / `0000:05:00.0` 未接通 |

## 代码风格

| 项 | 规则 |
|---|---|
| 语言 | C++ 高性能系统风格，优先显式契约 |
| 错误处理 | 关键错误抛出或返回明确错误，禁止吞掉 |
| 注释 | 只写架构契约、性能红线、危险边界 |
| 数据结构 | 优先 cacheline 对齐、NUMA 友好、无锁队列 |
| 依赖 | 高性能依赖必须显式写入构建和部署文档 |

## 工作规范

| 规则 | 要求 |
|---|---|
| 第一性原理 | 动机不清先停，路径明显不优要直接纠正 |
| Let it crash | 发现问题尽早暴露，禁止兜底、降级、吞错 |
| 不擅自开分支 | 不创建新分支或新 worktree，除非用户明确要求 |
| 保证开发质量 | 不能说能运行就算了，一定要高质量完成 |
| 单一文档源 | 项目状态写 `docs/PROJECT_STATE.md`，环境命令写 `docs/ENVIRONMENT.md` |
| 验证优先 | 改代码后尽量用最小相关测试验证，再做更大范围 smoke |
| 不修无关问题 | 发现无关问题可记录，不顺手大改 |
| 输出收口 | 最终答复必须说清改了什么、验证了什么、下一步是什么 |

## 当前关键事实

| 项 | 事实 |
|---|---|
| 非 DPDK 开发 | 优先在 VMware Linux `/home/sumu/projects/XTransfer` |
| 最新普通网络/文件层稳定性 | VMware `ctest 34/34`、`tools/xtransfer-vm-pending-acceptance.sh --full-base-port 56400`、latest full `tools/xtransfer-vm-stage-acceptance.sh --profile full --base-port 58400 --skip-build --skip-ctest`、latest extended `--profile extended --base-port 58800 --skip-build --skip-ctest` 和 latest quick `--profile quick --base-port 60000 --skip-build --skip-ctest` 已通过；stage acceptance 当前引用 18 个 smoke，新增 `udp-dev-extended-matrix` 覆盖更重参数的混合 long-run replay 与 bounded benchmark。已覆盖 task-log replay unsigned 负数拒绝、`loss_ratio` 有界有限值、manifest bytes 溢出、progress `bytes_total` 不变式、failed `failure_kind` 持久化/可观测、资源 endpoint 唯一性、资源 unsigned 负数拒绝、`resource_last_release_*`、ACK/completion 来源绑定、ACK `next_sequence < initial_sequence` 拒绝、ACK `next_sequence` 超过已发送范围拒绝、SACK 越界拒绝、旧 ACK/SACK 不能触发过期重传、completion 早到拒绝、receiver peer 绑定、START active stream_id 冲突拒绝、configured output missing `expected_bytes`/parent dirs/old final preservation/success no-overwrite/unique temp names、receiver `output-root` symlink 逃逸拒绝、receiver 半写输出临时文件清理与原子提交、completion checksum/bytes mismatch、单文件 no-completion、目录 no-completion、loss/TC-SACK replay、timeout replay、long-run replay、anomaly replay、running-cancel replay、restart-recovery、resource replay、directory cascade replay、duplicate replay、file-boundary、directory manifest mismatch、DPDK launcher contract 和 receiver service；FEC real length metadata 尾部非零值也已拒绝并在 `fec_codec_test` 中验证 |
| 最新 no-completion 收口 | `--no-completion-frame` receiver 注入、`kernel_udp_no_completion_timeout_test`、`xtransfer-grpc-udp-dev-no-completion-smoke.sh`、`xtransfer-grpc-udp-dev-directory-no-completion-smoke.sh` 已通过 VMware 验证。单文件场景验证 receiver 写完但不发 completion 时 task `FAILED`、`failure_kind=launcher`、reservation 释放和 `resource_last_release_*` 可见；目录场景验证第一个 child 失败后 parent `FAILED`、后续 child 不创建、replay 不重建 reservation 且拒绝复用 `task_id`。`tools/xtransfer-vm-pending-acceptance.sh` 已修复为先刷新 CMake 构建图，避免旧 build 目录漏掉新增目标 |
| 最新 DPDK 状态契约 | `tools/xtransfer-dpdk-launcher-contract-smoke.sh` 已通过；blocked reason 必须包含 `tools/xtransfer-validate-dpdk-rx.sh` 和 `rx_received/decoded/accepted/drained` 四个指标 |
| 最新 DPDK 安全编排 | 新增 `tools/xtransfer-dpdk-experiment.sh`、`tools/xtransfer-dpdk-nic-snapshot.sh`、`tools/xtransfer-dpdk-nic-restore.sh`；`vfio-exclusive` 用于独占绑定，`mlx5-bifurcated` 用于 100G RDMA/Mellanox 非解绑门禁；`tools/xtransfer-run-dpdk-smoke.sh` 已接入该编排层 |
| 100G 非破坏性检查 | 小服务器 `0000:82:00.0` / `enp130s0np0` 和大服务器 `0000:b1:00.0` / `enp177s0np0` 均为 `mlx5_core`、`carrier=1 speed=100000 mtu=9000`；两端 `libdpdk=21.11.9` 且 `librte_net_mlx5/libmlx5/libibverbs` 可见；`mlx5-bifurcated --allow-active-ip --execute -- true` 通过，双向 ping 5/5 且 0% loss；安全快照为 `.tmp/xtransfer-100g-small-safe-snapshot.env` 和 `.tmp/xtransfer-100g-big-safe-snapshot.env` |
| 最新 DPDK EAL probe | 新增 `tools/xtransfer-dpdk-eal-probe.cpp` 和 CMake 目标 `xtransfer-dpdk-eal-probe`；大服务器 100G 已通过安全编排输出 `driver=mlx5_pci link_status=up link_speed=100000`，运行后 IP/route/MTU 正常且 ping 3/3；小服务器通过 `tools/xtransfer-dpdk-sudo-stage.sh` staged binary 后也已通过安全编排输出 `name=0000:82:00.0 driver=mlx5_pci link_status=up link_speed=100000`，运行后 IP/route/MTU 正常且 ping 3/3，临时 stage 已清理 |
| 真实 DPDK 状态 | 大/小服务器 25G 口均有 carrier；小服务器 IOMMU 未启用导致 `vfio-pci` 绑定失败；100G RDMA/Mellanox 已完成非解绑门禁、链路检查和双端 EAL/PMD 发现，真实 DPDK RX 仍未闭环，不能宣称完成 |
| 下一步方向 | 无 DPDK 窗口时按 `docs/PROJECT_STATE.md` 的深推进计划继续：VMware extended/full smoke、receiver 重启/清理、重复/迟到 control/completion、输出冲突、sequence/window、SACK/FEC/checksum 边界和可观测性；不主动操作服务器 |

---
> Source: [Cantoneseman/XTransfer](https://github.com/Cantoneseman/XTransfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
