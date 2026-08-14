## agent4

> ├── Cargo.toml              # single crate (not a workspace)

# Repository Guidelines

## Project Structure & Module Organization

```
agent4/
├── Cargo.toml              # single crate (not a workspace)
├── README.md               # project overview + cross-project refs
├── AGENTS.md               # <- 本文档
├── FSRS_speed_first_architecture.md  # 原始速度优先架构设计稿
├── config/                 # 配置文件 (app.json, flows/, etc.)
├── src/
│   ├── main.rs             # 二进制入口 (serve / ask 子命令)
│   ├── lib.rs              # crate root (库模块注册)
│   ├── flow/               # Flow 引擎 (DAG 拓扑执行)
│   │   ├── engine.rs       # FlowEngine (DAG 执行、Stage-0~3 集成)
│   │   ├── types.rs        # NodeType 枚举 (12+1 种)、FlowNode/FlowResult
│   │   └── mod.rs
│   ├── cache/              # 多级缓存
│   │   ├── ngram.rs        # C0/C1 ngram hash 缓存 (DashMap + TTL)
│   │   ├── intent_normalizer.rs  # C0.5 意图归一化 (聚类共享缓存 + FSRS)
│   │   └── mod.rs
│   ├── fsrs/               # FSRS Oracle 零推理预判
│   │   ├── oracle.rs       # FsrsOracle (predict / KNN / 方向先验 / 场景可缓存率)
│   │   ├── state.rs        # FsrsState (D/S/R + 19 参数权重)
│   │   ├── knn.rs          # KnnDatabase (cosine KNN 难度估计)
│   │   └── mod.rs
│   ├── mcts/               # MCTS 决策树
│   │   ├── tree.rs         # MctsTree (UCB1 + FSRS prior + 可视化)
│   │   └── mod.rs
│   ├── models/             # 模型注册与调用
│   │   ├── registry.rs     # ModelRegistry (chat/embed/rerank + OpenAI 兼容)
│   │   ├── auto_think.rs   # AutoThink 规则引擎 (零 token 场景分类 + 难度估计)
│   │   └── mod.rs
│   ├── embed/              # Embedding registry
│   │   ├── registry.rs
│   │   └── mod.rs
│   ├── pipeline/           # 流水线编排
│   │   ├── stage_router.rs # StageRouter (Stage-0: C0 -> C0.5 -> AutoThink -> Oracle)
│   │   └── mod.rs
│   ├── task/               # DAG 任务管理 (Todo -> Task 升级)
│   │   ├── types.rs        # Task / TaskList / Blocker / TaskStatus
│   │   ├── engine.rs       # 持久化 + 任务创建辅助
│   │   └── mod.rs
│   ├── api/                # Axum HTTP handlers
│   │   ├── handlers.rs     # Flow CRUD + 执行 + 模型管理 + MCTS/FSRS/Cache 状态
│   │   ├── openai.rs       # OpenAI 兼容 /v1/chat/completions
│   │   ├── ws.rs           # WebSocket 实时推送 (stats + MCTS 树)
│   │   └── mod.rs
│   ├── time_budget.rs      # TimeBudget 硬时钟控制器 (FAST 3s / DEEP 30s)
│   ├── config.rs           # AppConfig + AppState
│   ├── error.rs            # Agent4Error enum
│   └── utils.rs            # now_secs() 等共享工具
├── doc/                    # 架构文档 (00-14 + ERRATA + appendices + python/)
├── skills/                 # AI assistant skills
├── static/                 # 前端静态文件
└── python/                 # Python MVP (C 核心 + Python 封装)
```

- agent4 是**单 crate 项目**（非 workspace），库入口 `src/lib.rs`，二进制入口 `src/main.rs`。
- 默认 private；仅 `pub` 对外需要的接口。
- 所有错误统一使用 `Agent4Error` enum（`src/error.rs`）。

## Build, Test, and Development Commands

```bash
# Build the project
cargo build

# Build with release optimizations (LTO enabled)
cargo build --release

# Run all tests
cargo test

# Check compilation without producing binaries (fast)
cargo check

# Format code
cargo fmt

# Lint (strict)
cargo clippy -- -D warnings

# Run the server (default FAST mode)
cargo run --release

# Run in DEEP mode
cargo run --release -- serve --deep

# Single-shot query via CLI
cargo run --release -- ask "your question" --verbose
```

Prefer `cargo check` over `cargo build` during development for faster iteration.

## Coding Style & Naming Conventions

- **Indentation**: 4 spaces. Use `cargo fmt` -- it's the single source of truth.
- **Naming**: `snake_case` for modules/functions/variables, `CamelCase` for types, `SCREAMING_SNAKE` for constants.
- **Error type**: Always use `Agent4Error` enum from `src/error.rs`. Never return `String` as an error.
- **Visibility**: Default to private. Only `pub` what other crates need.

## Testing Guidelines

- Unit tests live in `#[cfg(test)] mod tests` inside the same file.
- Integration tests go in `tests/` at the crate root.
- Test naming: descriptive, e.g. `top_direction_with_weights()`, not `test1()`.
- Run with `cargo test`. CI runs `cargo test` on every push.

## Architecture: Speed-First Pipeline

```
Request -> Stage-0 (C0/C0.5 cache) -> Stage-1 (preheat, TODO) ->
           Stage-2 (FAST/DEEP generation) -> Stage-3 (post-process) -> Response
```

- **Stage-0**: `StageRouter.decide()` runs C0 hash -> C0.5 intent normalizer -> AutoThink classify -> FsrsOracle predict
- **Stage-1**: Preheating (NOT YET IMPLEMENTED -- S1 roadmap)
- **Stage-2**: `FlowEngine.execute_flow()` with TimeBudget gating
- **Stage-3**: Async cache writeback + FSRS KNN update

### Known Gaps (code vs. doc)

| Gap | Status | Notes |
|-----|--------|-------|
| C1 semantic cache (cos>0.92) | NOT IMPLEMENTED | `stage_router.rs` hardcodes `c1_hit: false` |
| FsrsOracle Preheater role | NOT IMPLEMENTED | Only Oracle (predict) role exists |
| FsrsOracle Evictor role | NOT IMPLEMENTED | Only Oracle (predict) role exists |
| Stage-1 preheat layer | NOT IMPLEMENTED | KV Prefix / search prefetch / direction preheat |
| DEEP mode in execute_flow | BUG | Hardcoded `ExecutionMode::Fast` |
| search/ (MCP tool) | NOT STARTED | doc/13-mcp-tool-integration.md |
| routing/ (A* scheduler) | NOT STARTED | doc/14-scheduler-slot.md |
| memory/ (three-layer) | NOT STARTED | doc/08-three-layer-memory.md |
| session/ (git-branch) | NOT STARTED | -- |

## Commit & Pull Request Guidelines

- Follow [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`.
- Commit titles <= 72 characters, in English, imperative mood ("add X" not "added X").
- One logical change per commit. Do not mix refactoring with feature work.
- PRs must pass `cargo fmt --check`, `cargo clippy -- -D warnings`, and `cargo test` before merge.

---
> Source: [1638410/agent4](https://github.com/1638410/agent4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
