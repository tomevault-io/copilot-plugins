## odooseek

> **新功能开发前，必须先有对应文档和 Issue。** 设计决策先写入 `docs/` 目录下的对应文档，确认后检查 GitHub Issues 是否有对应 Issue，如果没有应当及时创建，然后再开始编码。

# OpenCode

## 重要约定（文档及 Issues 优先原则）

**新功能开发前，必须先有对应文档和 Issue。** 设计决策先写入 `docs/` 目录下的对应文档，确认后检查 GitHub Issues 是否有对应 Issue，如果没有应当及时创建，然后再开始编码。

**此原则不适用于：测试、错误修复**

## Development Workflow

### 仓库分工（Issue / PR 必须遵守）

| 仓库 | 用途 | Issue | PR |
|:---|:---|:---:|:---:|
| **[FDE-GROUP/odooseek](https://github.com/FDE-GROUP/odooseek)** | 主开发仓（`origin`） | ✅ **唯一** | ✅ |

```bash
gh issue create --repo FDE-GROUP/odooseek ...
gh pr create --repo FDE-GROUP/odooseek ...
```

- **GitHub Flow**: main ← PR ← feature-branch, PR body references issues with `closes #N`
- **Branch naming**: `feat/N-desc`, `fix/N-desc`, `refactor/N-desc`, `docs/N-desc`, `test/N-desc`, `perf/N-desc`
- **Documentation language**: Chinese (中文)
- **Keep main green**: main branch must always build and pass all tests
- **Commit format**: `type: description (refs #N)` — types: feat, fix, docs, refactor, test, perf, chore
- **PR format**: title uses `(refs #N)`, body contains `closes #N` for auto-close on merge

### 提交前必须本地执行 CI 预检测

提交代码或创建 PR 前，**必须**在本地运行以下检查，全部通过后才能提交：

**前端 (apps/oweb)**:
```bash
cd apps/oweb
bun run build    # tsc -b && vite build (类型检查 + 构建)
bun run lint     # biome check
bun run format   # biome format --write
```

**后端 (crates/)**:
```bash
RUSTFLAGS="-D warnings" cargo fmt --check --all
RUSTFLAGS="-D warnings" cargo clippy --all-targets --no-deps
RUSTFLAGS="-D warnings" cargo build --workspace
RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps
RUSTFLAGS="-D warnings" cargo test --workspace -- --test-threads=1
```

---
> Source: [FDE-GROUP/odooseek](https://github.com/FDE-GROUP/odooseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
