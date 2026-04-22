## trace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust-based storage I/O trace log analyzer. Parses kernel trace logs (UFS, Block I/O, UFSCustom), converts them to Parquet format, computes latency statistics, and generates charts. Supports MinIO/S3 storage integration and a gRPC server/client for remote processing.

## Build & Run

```bash
cargo build                    # dev build
cargo build --release          # release build (used for actual runs)
cargo test                     # run tests
cargo test <test_name>         # run a single test
```

Requires `protoc` (Protocol Buffers compiler) for gRPC proto compilation (see `build.rs`).

### Running Modes

- **Local CLI**: `./target/release/trace <log_file> <output_prefix>`
- **Parquet analysis**: `./target/release/trace --parquet <ufs|block> <parquet_file> <output_prefix>`
- **MinIO mode**: `./target/release/trace --minio-log <source_path> <target_path>` (requires `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`, `MINIO_BUCKET` env vars)
- **gRPC server**: `./target/release/trace --grpc-server` (listens on port 50051, requires MinIO env vars)
- **gRPC client**: `./target/release/trace --grpc-client <command> [options]` (commands: `process`, `csv`, `list`, `print`)
- **Migration**: `./target/release/trace --migrate <path>` (migrates old Parquet schemas)

## Architecture

### Core Data Flow

Log file → **Parser** → Vec of trace structs → **Processor** (latency computation) → **Output** (Parquet/CSV/Charts/Statistics)

### Module Structure (`src/`)

- **`models/`** — Data structs for trace types: `UFS`, `Block`, `UFSCUSTOM`. Each implements `TraceItem` trait. New trace types go here + register in `TraceType` enum (`trace_type.rs`).
- **`parsers/`** — Log file parsing. `log_high_perf.rs` is the high-performance parser using `memmap2` + `rayon` for parallel parsing. `log.rs` is the legacy parser. `log_common.rs` has shared parsing utilities.
- **`processors/`** — Latency calculation for each trace type (DtoC, CtoC, CtoD, queue depth). Each trace type has its own processor module.
- **`output/`** — Parquet serialization (`parquet.rs`), Parquet reading (`reader.rs`), CSV export (`csv.rs`), chart generation via `plotters` (`charts.rs`), and statistics printing (`statistics.rs`).
- **`storage/`** — MinIO/S3 client (`minio_client.rs`) using `rust-s3` crate.
- **`grpc/`** — gRPC server and client. Proto definition in `proto/log_processor.proto`. Server handles `ProcessLogs`, `ConvertToCsv`, `GetJobStatus`, `ListFiles`, `ReadParquet` RPCs with streaming progress.
- **`commands/`** — High-level command orchestration for MinIO workflows.
- **`migration/`** — Parquet schema migration tool.
- **`utils/`** — Filter options, latency range config, compression (gz/xz/7z/zip/tar), encoding detection, logging.

### Key Patterns

- Global state for filter options (`OnceLock<FilterOptions>` in `lib.rs`) and latency ranges.
- Filter options (`FilterOptions` in `utils/filter.rs`) support time range, sector range, latency range, queue depth, and CPU filtering — used across CLI, MinIO, and gRPC modes.
- Adding a new trace type: (1) add struct in `models/`, (2) add processor in `processors/`, (3) add output functions in `output/`, (4) register in `TraceType` enum.

## Language

Project documentation and code comments are in Korean. Maintain Korean for commit messages and comments.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kakaromo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
