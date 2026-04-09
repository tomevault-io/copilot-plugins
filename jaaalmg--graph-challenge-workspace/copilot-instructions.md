## graph-challenge-workspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库定位（Graph Challenge 工作区）

本仓库用于 Graph Challenge 相关研究与工程实现，包含两套相互关联的 UPMEM DPU 子图计数代码：

- `PimPam/`：PimPam 原始实现（基线）。
- `PRISM/`：在 PimPam 思路上扩展的实现（更多图/模式、虚拟 DPU 扩展、位图加速路径等）。

同时包含两篇论文的文本版本，作为后续研究与实现对照依据：

- `PimPam/PimPam-source-paper.txt`
- `PRISM/Prism-source-paper.txt`

后续任务默认优先在 `PRISM/` 上推进，并与 `PimPam/` 做对照验证。

---

## 常用命令

> 命令需在对应子目录下执行（`PRISM/` 或 `PimPam/`）。

### PRISM

- 默认构建并运行：
  - `make test`
- 指定图与模式运行：
  - `GRAPH=AM0312 PATTERN=CLIQUE3 make test`
- 单 DPU/单 tasklet 快速自检：
  - `make test_single`
- 批量测试（Makefile 内置）：
  - `make test_all`
- 可扩展性扫描（虚拟 DPU）：
  - `GRAPH=AM0312 PATTERN=CLIQUE3 make test_sc`
- 单次覆盖虚拟 DPU 数：
  - `GRAPH=AM0312 PATTERN=CLIQUE3 EXTRA_FLAGS="-DV_NR_DPUS=5120" make test`
- 清理：
  - `make clean`

### PimPam

- 默认构建并运行：
  - `make test`
- 指定图与模式运行：
  - `GRAPH=WV PATTERN=CLIQUE3 make test`
- 单 DPU/单 tasklet 快速自检：
  - `make test_single`
- 批量测试（Makefile 内置）：
  - `make test_all`
- 清理：
  - `make clean`

### PRISM 图数据预处理（Python）

- 边列表转二进制 CSR：
  - `python3 python_tool/adjtsv2csrbin.py input.tsv --output graph.bin --header 1`
- 分析 CSR 图统计信息：
  - `python3 python_tool/analyze_csr_graph.py path/to/graph.bin`

---

## 关键前提与注意事项

- 依赖 UPMEM 工具链（如 `dpu-upmem-dpurte-clang`、`dpu-pkg-config`）。
- 两个项目均通过各自 `makefile` 同时编译 Host 与 DPU 代码。
- 两个 makefile 都没有独立 lint 目标。
- 结果文件通常写入各项目下的 `result/` 目录。

**数据路径注意：**
- `PRISM/include/common.h` 使用 `DATA_DIR "./data/"`。
- `PimPam/include/common.h` 使用 `DATA_DIR "../../origin/Prism/data/"`。

若运行时报找不到数据文件，优先检查当前项目的 `DATA_DIR` 宏与数据落盘位置是否一致。

---

## 架构总览（便于快速上手）

### 1) 编译期宏配置驱动全局行为

两套代码都在 `include/common.h` 通过宏控制：

- `GRAPH=...`：选择数据集（映射到 `DATA_NAME`、`N`、`M`）。
- `PATTERN=...`：选择模式核函数（映射到 `KERNEL_FUNC`）。
- `NR_DPUS`、`NR_TASKLETS`、`DPU_N`、`DPU_M`、`DPU_ROOT_NUM`：控制并行规模与内存上限。

`Graph` 结构体内嵌大数组（按 `N`/`M` 宏分配），因此图配置会直接影响内存占用与运行行为。

### 2) Host 侧负责“准备 + 分配 + 调度 + 回收”

主流程入口与职责：

- `host/main.c`：整体调度（图准备、DPU 分配/加载、启动、结果与 cycle 回收、结果写盘）。
- `host/partition.c`：读入 CSR、重编号/排序、负载估计、root 分配到 DPU、压缩与传输。
- `host/mine.c`：CPU 侧参考实现（用于理解算法语义与对照验证）。

PRISM 相比 PimPam 的扩展点：

- 支持 `V_NR_DPUS` 的批处理执行。
- 支持位图路径与普通路径混合调度（`BM_DPUS` / `BM_NUMS`）。

### 3) DPU 侧是“统一入口 + 模式内核”

- `dpu/main.c`：tasklet 初始化、barrier 同步、调用 `KERNEL_FUNC(tasklet_id)`。
- `dpu/<PATTERN>.c`：具体模式计数逻辑（如 `CLIQUE3.c`、`CLIQUE4.c` 等）。
- `dpu/set_op.c`：交集等集合操作基础能力。

PRISM 还包含位图专用入口：

- `dpu/bitmap.c`：调用 `KERNEL_FUNC_BM`。

### 4) Host ↔ DPU 通过符号名契约交互

不是 RPC，而是基于 DPU 符号读写：

- DPU 端共享缓冲/控制变量集中在 `include/dpu_mine.h`（如 `row_ptr`、`col_idx`、`roots`、`ans`、`cycle_ct`、`root_num` 等）。
- Host 端通过 UPMEM runtime 按符号名进行 `copy_to/copy_from`。

修改 DPU 内存布局或变量名时，必须同步更新 Host 传输逻辑。

### 5) 输入数据格式（CSR 二进制）

读入格式约定：

1. `uint32 n`
2. `uint32 m`
3. `row_ptr`（长度 `n`，Host 侧补 `row_ptr[n] = m`）
4. `col_idx`（长度 `m`）

图文件名与路径由 `include/common.h` 的宏决定。

---

## UPMEM 离线文档参阅（供后续 AI 代码生成）

离线文档已镜像到本仓库，可直接本地查阅：

- 主入口：`upmem-docs-2025.1.0/sdk.upmem.com/2025.1.0/index.html`
- 文档索引：`UPMEM_DOC_INDEX.md`
- 高频速查：
  - `docs/upmem-key-apis/host-api-cheatsheet.md`
  - `docs/upmem-key-apis/dpu-runtime-cheatsheet.md`

推荐查阅顺序（写 Host 代码时）：
1. `UPMEM_DOC_INDEX.md`
2. `.../06_ControllingDPUFromHost.html`
3. `.../032_DPURuntimeService_HostCommunication.html`
4. `.../CAPI/dpu_8h.html`

推荐查阅顺序（写 DPU kernel 时）：
1. `UPMEM_DOC_INDEX.md`
2. `.../030_DPURuntimeService_Tasklets.html`
3. `.../031_DPURuntimeService_Memory.html`
4. `.../071_MeasuringPerformances.html` / `.../263_WramFifo.html`

---

## 面向后续研究任务的建议工作方式

- 新优化优先在 `PRISM/` 实现，再用 `PimPam/` 做基线对照。
- 任何与算法语义相关的变更，优先交叉检查：
  - DPU 内核实现（`dpu/*.c`）
  - Host 参考实现（`host/mine.c`）
  - 对应论文 txt 说明
- 对性能问题，优先查看 `result/*.txt` 与 `python_tool/show_cycle.py` / `show_run_cycle.py` 产出的分布信息，再决定改动方向。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JaaaLmg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JaaaLmg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
