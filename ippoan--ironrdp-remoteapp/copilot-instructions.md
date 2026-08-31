## ironrdp-remoteapp

> Devolutions/IronRDP の縮小 fork。手を入れた 2 crate + web-client だけを持ち、

# IronRDP (ippoan fork)

Devolutions/IronRDP の縮小 fork。手を入れた 2 crate + web-client だけを持ち、
他の ironrdp-* は全部 上流 git 依存 (rev 固定)。詳細は README.md。

## 触るときの約束

- ironrdp 系の依存は **git (rev 固定) に統一**。crates.io 版を混ぜると同名型が二重化して壊れる。
- `crates/ironrdp-session` は `fill_rect` を足しただけの vendor。root の `[patch]` が上流版を差し替えている。
  上流 rev を上げるときは rev 一括置換 + この vendor の rebase をセットで。
- wasm ビルドは `web-client/iron-remote-desktop-rdp/pre-build.js` が担当 (旧 `cargo xtask web build` 相当)。
- 検証: `cargo check --workspace --target wasm32-unknown-unknown` と
  `cargo test -p ironrdp-session`。

---
> Source: [ippoan/ironrdp-remoteapp](https://github.com/ippoan/ironrdp-remoteapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
