## ferris-fury-road

> 123 crate 的 Cargo workspace。Edition 2024，Toolchain stable 1.85+。

# Ferris 的狂暴之路 (Rust Fury Road)

123 crate 的 Cargo workspace。Edition 2024，Toolchain stable 1.85+。
人类 × Claude AI 深度协作。废土美学包装 + 专业技术内核。

## 命令

```bash
cargo check --workspace          # 编译检查
cargo test -p lesson-ownership   # 单课测试
cargo test --workspace           # 全部测试
cargo fmt --all                  # 格式化
cargo clippy --workspace         # Lint
cargo run -p project-calculator  # 运行项目
mdbook build book/               # 构建在线书
mdbook serve book/               # 本地预览
python scripts/generate-summary.py  # 重新生成书目录
```

## 结构

三层路径（废土命名 + 技术内核）：
- **核心生存** (01-07)：荒原初醒 → 资源法则 → 生存图腾 → 变异铸造 → 净化仪式 → 禁忌咒语 → 并发风暴
- **铁匠工坊** (08-09)：工程实践 → 数据绿洲
- **领域探索** (10-16)：部落联盟 / 控制终端 / 机械先知 / 天空之城 / 深渊引擎 / 铁锈交易所 / 链上废墟

Crate 命名：`lesson-{语义名}` / `deep-dive-{语义名}` / `project-{语义名}`。

## 规范

- 练习：`pub fn` + `todo!()` + `#[cfg(test)] mod tests`
- 项目：`src/main.rs` + `src/lib.rs` + `tests/integration.rs`
- Deep-dive：与正课同格式，纯 Rust 模拟，无外部依赖
- 中文注释，`cargo fmt` + `cargo clippy` 通过
- 依赖统一在 `[workspace.dependencies]`
- Phase README 标题用废土名，技术正文保持专业客观
- 读者称为 War Boy

## 书与部署

- `book/` 目录：mdBook 配置，废土暗色主题
- `scripts/generate-summary.py`：自动生成 SUMMARY.md
- `.github/workflows/book.yml`：push main → GitHub Pages；release → PDF/EPUB
- `.github/workflows/ci.yml`：跨平台编译检查 + fmt + clippy

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Coh1e) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
