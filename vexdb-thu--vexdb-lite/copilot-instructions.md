## vexdb-lite

> VexDB 是一个高性能向量检索插件库，提供 PostgreSQL（`vexdb_lite`）和 DuckDB（`vexdb_lite`）两种适配形式，共享同一套 HNSW 图索引算法内核。

# VexDB — AI Agent Guide

VexDB 是一个高性能向量检索插件库，提供 PostgreSQL（`vexdb_lite`）和 DuckDB（`vexdb_lite`）两种适配形式，共享同一套 HNSW 图索引算法内核。

---

## 目录结构

```
vexdb_lite/
├── common/                    # 两个插件共享的算法内核
│   ├── include/               # 共享头文件
│   │   ├── graph_index/       # HNSW 算法：节点结构、MemStore、邻居选择
│   │   ├── half.h             # 半精度浮点工具
│   │   ├── vec_common.h       # 向量操作基础
│   │   └── macro.h            # 跨平台宏
│   ├── distance/              # 距离计算
│   │   ├── src/               # SSE/AVX/AVX512/NEON dispatcher + architecture 探测
│   │   ├── core/              # 距离模板（distances_simd_template.cpp 等）
│   │   └── pq/                # PQ 距离
│   ├── quantizer/             # Product Quantization 编码/解码、K-means
│   ├── rabitq/                # RaBitQ 近似距离
│   ├── vtl/                   # 模板容器（Vector、HashSet、PriorityQueue、allocator）
│   └── module/                # 工具模块（timer、parallel_counter）
│
├── vexdb_pg/                  # PostgreSQL 插件
│   ├── include/               # PG 专属头文件（floatvector.h、pg_compat.h 等）
│   ├── src/                   # 实现文件
│   │   ├── graph_index_am.cpp      # IndexAmRoutine 注册
│   │   ├── graph_index_build.cpp   # 串行/并行建索引
│   │   ├── graph_index_scan.cpp    # ANN 搜索（Index Scan）
│   │   ├── graph_index_insert.cpp  # 增量插入
│   │   ├── graph_index_xlog.cpp    # WAL
│   │   ├── floatvector.cpp         # floatvector 类型
│   │   ├── guc_config.cpp          # GUC 参数 + reloptions
│   │   └── distance/               # PG 端 SIMD 分发入口
│   ├── sql/                   # vexdb_lite--1.0.sql
│   ├── vexdb_lite.control
│   └── CMakeLists.txt
│
├── vexdb_duckdb/                # DuckDB 扩展
│   ├── include/               # DuckDB 专属头文件
│   │   ├── vex_graph_index.hpp           # GraphIndex（BoundIndex 封装）
│   │   ├── vex_graph_index_depend_duck.hpp  # MemStore（DuckDB 侧独立实现）
│   │   └── duck_pg_shim.hpp             # 为 common/ 提供 PG 类型 mock
│   ├── functions/             # 注册的 SQL 函数（距离、ANN、index_info 等）
│   ├── index/                 # GraphIndex 生命周期、持久化、WAL
│   ├── optimizer/             # VEXDB_INDEX_SCAN 优化器 rewrite
│   ├── distance/              # SIMD 分发入口（调 common/distance/src/ 模板）
│   ├── compat/                # 平台兼容
│   ├── vex_extension.cpp      # 扩展入口 + 函数/参数注册
│   └── CMakeLists.txt
│
├── build_duck.sh              # DuckDB 可加载扩展构建（含 setup/build/bench/unittest 子命令）
├── scripts/release.sh         # 远程双平台发版流水线
└── tests/spec/                # 多引擎 YAML spec 测试套件
```

---

## 构建命令

### PostgreSQL 插件

```bash
# 构建（推荐从子目录）
cmake -B build/pg vexdb_pg/ -DCMAKE_BUILD_TYPE=Release
cmake --build build/pg -j$(nproc)

# 产物
build/pg/vexdb_lite.so

# 安装
sudo cmake --install build/pg

# Kylin V10 / 无系统 Boost 的机器需要传 fallback 路径
cmake -B build/pg vexdb_pg/ -DBOOST_FALLBACK_INC=/path/to/boost_1_90_0
```

### DuckDB 扩展（可加载 extension）

```bash
# Release 构建（首次自动拉取 DuckDB v1.5.2 源码，~1.5GB）
bash build_duck.sh build

# 产物
build/duck/build/extension/vexdb_lite/vexdb_lite.duckdb_extension

# 全量重建
bash build_duck.sh clean && bash build_duck.sh build
```

---

## 测试

```bash
# DuckDB spec 全量（推荐，需 Docker）
bash tests/spec/_lib/docker/run_duckdb.sh test

# PG spec 全量（需 Docker + PG19）
bash tests/spec/_lib/docker/run_pg.sh test

# DuckDB unittest 直跑（需先 build_duck.sh build）
./build/duck/build/test/unittest "[spec_run]"         # 全量
./build/duck/build/test/unittest "*graph_index_ann*"  # 单用例
```

`tests/spec/` 是多引擎 YAML spec DSL（`duckdb/` + `pg/` + `shared/`），通过 `render.py` 渲染成各引擎的测试格式。

---

## 关键架构

### 共享内核 vs 适配层

`common/` 目录是两个插件共用的算法内核，不包含任何 PG 或 DuckDB 的 API 依赖。两侧通过宏隔离：

- `PG_VEXDB_TARGET_PG`：编译 PG 插件时定义
- `PG_VEXDB_TARGET_DUCK`：编译 DuckDB 扩展时定义

DuckDB 侧用 `vexdb_duckdb/include/duck_pg_shim.hpp` 提供 PG 类型的 mock（`MemoryContext`、`palloc` 等），使 `common/` 代码无需修改即可在 DuckDB 上下文中编译。

### HNSW 图索引

- **GraphIndexCore**（`common/include/graph_index/`）：纯算法层，不依赖任何存储后端
- **MemStore / DiskStore**：存储抽象；DuckDB 端用 `FixedSizeAllocator`（`vexdb_duckdb/include/vex_graph_index_depend_duck.hpp`），PG 端用 palloc + DSM
- **并行构建（PG 端）**：fork 出 worker process + DSM shared memory + LWLock；不使用 `std::thread`（PG 进程全局态不兼容跨线程）
- **并行构建（DuckDB 端）**：`std::thread` + 共享地址空间，不存在 PG 全局态问题

### 距离计算

运行时 SIMD dispatch（`common/distance/src/architecture.cpp: get_best_arch()`），无需重新编译即可利用当前 CPU 的最高 SIMD 级别。

实现文件组织：
- `common/distance/src/` — dispatcher（`sse_dispatcher.cpp`、`avx_dispatcher.cpp` 等）
- `common/distance/core/` — 实际 SIMD 模板（`distances_simd_template.cpp`）
- 各平台入口（`distance/sse.cpp`、`distance/avx.cpp` 等）通过 `#include` 引用 core 模板，以 `-march=...` 编译

### DuckDB 查询优化器

`vexdb_duckdb/optimizer/vex_optimizer.cpp` 检测 `ORDER BY distance_func(...) LIMIT k` 模式并改写为 `VEXDB_INDEX_SCAN`，触发向量索引扫描而非全表扫描。

---

## PG 端关键约束

1. **禁止跨 PG boundary 抛 C++ 异常**，只用 `elog(ERROR)`
2. **所有内存**通过 PG memory context（palloc/pfree），不用裸 malloc
3. **`shared_preload_libraries`**：`vexdb_lite` 需要预加载才能注册 LWLock tranche
4. **LWLock 注册顺序**：必须在 `shmem_request_hook` 中注册，在 `shmem_startup_hook` 中使用；`_PG_init` 时不能使用
5. **向量存储**：独立的 `_vec` 文件（PG extension 无法注册新 fork 类型）
6. **并行 worker 中禁用 `std::thread`**：PG fork 模型下 worker 是独立进程，`std::thread` 会碰到 PG 进程全局态（`PG_exception_stack`、`CurrentMemoryContext`、buffer `PrivateRefCount`）导致 SIGBUS/SIGSEGV

---

## 版本约束

| 组件 | 版本 |
|------|------|
| PostgreSQL | 16 / 17 / 18 / 19 |
| DuckDB | v1.5.2 only |
| CMake（PG 插件） | ≥ 3.14 |
| CMake（DuckDB 扩展） | ≥ 3.28，**< 4.x** |
| Boost | vendored 1.91（`thirdparties/boost/`，已打 patch） |
| 目标平台 | Linux x86_64 / AArch64（ARMv8.2+） |

---

## 运行参数

### PostgreSQL

```sql
SET vexdb.ef_search = 64;                 -- 搜索宽度（默认 64）
SET vexdb.vec_architecture = '';          -- SIMD 架构（空 = 自动；格式 'usage:arch'）
SET vexdb.enable_vec_buffer_manager = on; -- 向量缓存管理器（默认 on）
SET vexdb.vector_buffers = '128MB';       -- 向量缓存大小（KB 单位）
SET vexdb.vector_buffer_workers = 1;      -- 缓存后台 worker 数
```

### DuckDB

```sql
SET vexdb_ef_search = 40;                -- 搜索宽度（默认 40）
SET vexdb_brute_force_threshold = 10000; -- 行数阈值，低于此值走全量扫描（默认 10000）
SET vexdb_pq_search_mode = 'off';        -- 'off' 或 'pq_only'
SET vexdb_pq_refine_k_factor = 1.0;      -- refine 倍率（1.0 = 关闭）
```

---

## 常见误区

- `vexdb_index_info()` 无参数，返回所有索引的信息表
- `CLEAN_DUCK_BUILD=1` 只在 `scripts/release.sh` 中有效，`build_duck.sh` 本身不识别
- `build_duck.sh` 没有 `debug` 子命令；debug 构建走 `bash build.sh dev`
- 修改 `common/` 代码时，必须确保同时在 PG 和 DuckDB 两个上下文下可编译

---
> Source: [VexDB-THU/VexDB-Lite](https://github.com/VexDB-THU/VexDB-Lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
