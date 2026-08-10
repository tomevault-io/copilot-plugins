## sift

> Welcome to the **Sift** repository! This document serves as the primary architectural reference and contribution guideline for both human developers and AI coding agents working on the codebase.

# AGENTS.md - Developer & Agent Architecture Guide

Welcome to the **Sift** repository! This document serves as the primary architectural reference and contribution guideline for both human developers and AI coding agents working on the codebase.

---

## 📌 Project Overview

**Sift** is a fast, safety-first local file organization and automation CLI engine built in Rust. It automatically categorizes, moves, compresses, and cleans up files based on customizable YAML rules and curated recipes without risking accidental data loss.

- **Repository**: [https://github.com/taevel02/Sift](https://github.com/taevel02/Sift)
- **Language**: Rust (2024 Edition, `rustc >= 1.85`)
- **License**: [MIT](LICENSE)

---

## 📁 Repository Structure

```
.
├── src/                    # Core Rust engine source code
│   ├── main.rs             # CLI entry point (Clap subcommands & dispatch)
│   ├── engine.rs           # Core OrganizerEngine (dry-run & execution)
│   ├── matcher.rs          # Rule matching logic (filters, EXIF, ID3, tags)
│   ├── action.rs           # File actions (move, copy, trash, delete, compress, script)
│   ├── transaction.rs      # Transaction logging & 1-click Undo/Revert engine
│   ├── watcher.rs          # Real-time background file system monitor (notify)
│   ├── tui.rs              # Interactive TUI status dashboard (Ratatui + Crossterm)
│   ├── recipe.rs           # Recipe management CLI engine (download & sync)
│   ├── config.rs           # Config parsing & XDG directory resolution
│   ├── validator.rs        # Syntax & semantic config validator (sift check)
│   ├── daemon.rs           # Native macOS launchd background daemon manager
│   ├── metadata.rs         # EXIF photo & ID3 audio metadata extraction
│   ├── tags.rs             # macOS Finder color tags reader & writer (xattr)
│   └── notification.rs    # macOS Hazel-style native desktop notification banners
├── recipes/                # Curated community YAML recipes catalog
├── tests/                  # Integration test suite (tempfile-isolated)
├── .github/workflows/      # CI/CD workflows (CI testing & Release packaging)
├── install.sh              # One-line shell installer script
├── schema.json             # IDE JSON Schema for config & recipe autocompletion
├── AGENTS.md               # Developer & Agent architecture reference
├── CONTRIBUTING.md         # Contribution guidelines
├── CODE_OF_CONDUCT.md      # Code of conduct
├── LICENSE                 # MIT License
└── README.md               # Product documentation & usage guide
```

---

## 🛠️ Architectural Guidelines & Safety Guarantees

### 1. Zero Data Loss & 1-Click Rollback
- Every destructive file modification (move, copy, trash, compress) must log a `FileOpRecord` inside a `Transaction`.
- All transactions are recorded in `~/.config/sift/history.json`.
- The `sift undo` command and TUI `u` keybinding reverse operations in exact opposite order and touch file modification timestamps (`mtime = now`) to reset age-based rule counters.

### 2. Rule Matching Strategy
- **First-Match Wins**: Rules are evaluated sequentially top-to-bottom. The first matching rule processes the file.
- **Default Exclusions**: Hidden files (starting with `.`) and symbolic links are excluded by default.

### 3. Collision Resolution Policy
- Target path collisions default to `CollisionPolicy::AutoRename` (`document.pdf` -> `document_1.pdf`, `document_2.pdf`). Never overwrite existing user files without explicit policy.

### 4. UI & Output Consistency
- All CLI output uses card badges (`sift <COMMAND>`) and clean, standardized terminal symbols (`✓`, `✗`, `⚠`, `›`, `•`). Emojis are avoided in standard CLI output for cross-terminal compatibility.

### 5. Strict Error Handling
- Return `Result<T, OrganizerError>` everywhere using `thiserror`.
- Avoid `unwrap()` or `panic!()` in production code paths. Use pattern matching or `?` operator to propagate errors cleanly.

---

## 🤝 Contribution Protocols

### 1. Code Base Contributions
Before submitting a Pull Request for core engine changes:

```bash
# 1. Format code according to rustfmt standard
cargo fmt --check

# 2. Run Clippy lints (warnings treated as errors)
cargo clippy -- -D warnings

# 3. Execute the full integration test suite
cargo test
```

### 2. Community Recipe Contributions
We warmly welcome contributions of new community recipes to `recipes/`!

- Place your `.yaml` recipe file inside the `recipes/` folder.
- Ensure the recipe includes the `$schema` header:
  `# yaml-language-server: $schema=https://raw.githubusercontent.com/taevel02/Sift/refs/heads/main/schema.json`
- Include `version: "1"`, `name`, `description`, and `enabled: true`.
- Validate your recipe syntax with `sift check`.
- Submit your recipe via a Pull Request to `main`.

---
> Source: [taevel02/Sift](https://github.com/taevel02/Sift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
