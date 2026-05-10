## tauri

> Tauri(Rust) 开发规范


# Tauri / Rust 规范
- 任何前端要调用 Rust 命令时，优先查 `src-tauri/src/lib.rs` 中现有 command，再决定新增
- command 命名使用清晰的动宾结构（例如 getHeroAlias / exportVpk），避免缩写
- 涉及资源文件/工具链时，优先放在 `src-tauri/resources/` 并通过统一路径访问，避免散落到其它目录
- 改动 Rust 逻辑后，注意同步前端调用处与 i18n 文案（如有）

---
> Source: [lanxiuyun/DotaTerrainSwitcher](https://github.com/lanxiuyun/DotaTerrainSwitcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
