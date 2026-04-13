## bwa-rust

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

bwa-rust 是一个用 Rust 从零实现的 BWA-MEM 风格 DNA 短序列比对器，支持 FASTA/FASTQ 输入和 SAM 输出。

## Common Commands

```bash
# 构建
cargo build --release

# 测试
cargo test                        # 运行所有测试（133 项）
cargo test --lib                  # 仅库测试
cargo test --test integration     # 仅集成测试
cargo test <test_name>            # 运行单个测试

# 代码质量
cargo fmt --all -- --check        # 检查格式
cargo clippy --all-targets --all-features -- -D warnings  # lint

# 基准测试
cargo bench

# CLI 用法
cargo run -- index <ref.fa> -o <prefix>         # 构建索引
cargo run -- align -i <index.fm> <reads.fq>     # 使用预构建索引比对
cargo run -- mem <ref.fa> <reads.fq>            # 一步比对
```

## Architecture

三层模块设计，数据流：I/O → 索引 → 比对 → SAM 输出。

### 模块结构

**`src/index/`** — FM 索引构建流水线：
- `sa.rs`：后缀数组（倍增法，O(n log²n)）
- `bwt.rs`：Burrows-Wheeler 变换（从 SA 推导）
- `fm.rs`：FM 索引（C 表 + 分块 Occ 采样，`backward_search`，bincode 序列化）
- `builder.rs`：索引构建入口

**`src/align/`** — 比对算法流水线：
- `seed.rs`：SMEM 种子查找（双向扩展）
- `chain.rs`：种子链构建与过滤（DP + 贪心剥离）
- `sw.rs`：带状 Smith-Waterman 局部对齐（仿射间隙）
- `extend.rs`：链到完整对齐转换
- `mapq.rs`：MAPQ 质量估算（BWA 风格）
- `pipeline.rs`：完整比对 pipeline

**`src/io/`** — I/O 层：FASTA/FASTQ 解析、SAM 格式化

**`src/util/dna.rs`** — DNA 编码/解码/反向互补工具

### 关键设计决策

- `unsafe_code = "forbid"`：全项目禁止 unsafe
- 非 Windows 平台使用 jemalloc（显著提升 rayon 多线程场景性能）
- SW 对齐使用 `SwBuffer` 缓冲区复用，减少热路径分配
- SA 稀疏采样（`sa_sample_rate`）平衡内存与查询速度
- FM 索引文件格式：magic `0x424D4146_4D5F5253` + version 2，基于 bincode 序列化

### 错误处理

统一使用 `BwaError` 枚举（`src/error.rs`），`BwaResult<T>` 别名，通过 `anyhow` 在 CLI 层传播。

## Code Quality

- `clippy.toml`：认知复杂度阈值 30，函数最大 200 行，参数最多 8 个，MSRV 1.70
- CI 强制：fmt → clippy → test → release build，所有 lint 错误为 hard failure
- 提交信息遵循 Conventional Commits 规范

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LessUp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LessUp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
