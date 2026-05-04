## sqlite-vaporetto

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Project

`sqlite-vaporetto` is a Rust SQLite loadable extension. It registers an FTS5
tokenizer named `vaporetto`, backed by the Vaporetto Japanese tokenizer.

The extension is intended to make practical Japanese full-text search available
inside ordinary SQLite databases. It supports FTS5 `MATCH` queries, `bm25()`
ranking, `highlight()`, ASCII case-insensitive matching by default, POS/tag
filtering, and SQL helper functions for query-side tokenization.

The project intentionally keeps SQLite source archives, extracted SQLite trees,
Vaporetto model files, and compiled artifacts out of git. They are temporary
build inputs under `.tmp/` or `target/`.

## Build And Test

Use the Makefile for end-to-end validation:

```sh
make test
```

This downloads SQLite and a Vaporetto distribution model into `.tmp/`, builds
the Rust extension, builds a local SQLite CLI with FTS5 enabled, loads the
extension, and verifies Japanese FTS5 search.

For Rust-only checks:

```sh
cargo test
```

If the SQLite source tree is not present, use `make build` rather than plain
`cargo build`. The Rust build depends on `sqlite3ext.h`, which Makefile fetches
from the SQLite source tarball and exposes via `SQLITE_INCLUDE_DIR`.

Use `make release` for a release build without an embedded model, and
`make embedded-release` for a release build with the default model embedded.
The default embedded model is
[`bccwj-suw+unidic_pos+kana.model.zst`](https://github.com/daac-tools/vaporetto-models/releases).

CI uses `bccwj-suw_c0.003` for smoke tests that exercise builds without a
bundled model. Tag filtering requires a tag-prediction model such as
[`bccwj-suw+unidic_pos+kana.model.zst`](https://github.com/daac-tools/vaporetto-models/releases);
do not assume the lightweight CI model supports `tags`.

GitHub Actions should run for pull requests and release tags only. Do not add a
plain branch `push` trigger unless the project owner explicitly asks for it.
Release artifacts are produced only for tags matching `v*`.

## Development Rules

- Keep `.tmp/`, `target/`, downloaded SQLite archives, extracted SQLite source,
  and Vaporetto model files out of git.
- Do not vendor SQLite source into this repository.
- Prefer TDD: update integration SQL/scripts first for behavioral changes, then
  update the extension implementation.
  - Use `tests/search.*` for core tokenizer/search behavior.
  - Use `tests/query_helpers.*` for `vaporetto_split`,
    `vaporetto_and_query`, and `vaporetto_or_query`.
  - Use `tests/tag_filter.*` for `tags` and `keep_untagged`.
  - Update both Unix `.sh` and Windows `.ps1` expectations when changing shared
    integration behavior.
- Keep the SQLite C surface small. The current design uses `src/sqlite_shim.c`
  only for SQLite extension/FTS5 API calls and keeps tokenizer logic in Rust.
- Prefer `sqlite3_vaporetto_init` in docs and tests. Preserve all exported
  entry points unless intentionally changing loader compatibility:
  - `sqlite3_vaporetto_init`
  - `sqlite3_sqlitevaporetto_init`
  - `sqlite3_extension_init`
- The tokenizer name exposed to FTS5 is `vaporetto`.
- When changing tokenizer args, document the SQL syntax in `README.md` and add
  an integration test.
- The SQL helper functions are part of the public surface:
  - `vaporetto_split(text[, separator[, options]])`
  - `vaporetto_and_query(text[, options])`
  - `vaporetto_or_query(text[, options])`
- Query helper functions should quote generated FTS5 tokens and omit
  whitespace-only tokens. Do not replace them with raw `split + join` behavior.
- ASCII letters are folded to lowercase by default for both index and query
  tokenization. Preserve original offsets so `highlight()` still marks the
  source text. Use `case sensitive` only when callers explicitly request
  uppercase/lowercase distinctions.
- `tags <prefixes>` is prefix-based POS/tag filtering. No-tag tokens are
  intentionally filtered out unless `keep_untagged` is present. Treat
  `keep_untagged` as "include tokens with empty/no POS tag"; do not label these
  tokens as unknown/UNK unless the Vaporetto model actually provides such a tag.
- Release artifacts are built in two variants:
  - unqualified asset name: requires `model <path>` or
    `SQLITE_VAPORETTO_MODEL`; contains no bundled model and is licensed as
    `sqlite-vaporetto` under `MIT OR Apache-2.0`.
  - `-with-model`: embeds
    [`bccwj-suw+unidic_pos+kana.model.zst`](https://github.com/daac-tools/vaporetto-models/releases);
    contains `sqlite-vaporetto` under `MIT OR Apache-2.0` and the bundled model
    under [BSD-3-Clause](https://opensource.org/license/BSD-3-Clause).
- Tagged release asset names include the release tag, platform, and model
  variant, for example
  `sqlite-vaporetto-v0.4.0-linux-x86_64.tar.gz` and
  `sqlite-vaporetto-v0.4.0-linux-x86_64-with-model.tar.gz`.

## Release Notes

- Keep not-yet-released notes in `docs/releases/HEAD.md`.
- Keep published release notes in `docs/releases/vX.Y.Z.md`, for example
  `docs/releases/v0.1.0.md`.
- `scripts/release-notes.sh <tag>` extracts `docs/releases/HEAD.md` first. This
  lets a tagged GitHub Release publish notes for the current HEAD.
- Before tagging a release, bump `Cargo.toml` and `Cargo.lock`, update examples
  in this file if they mention the current artifact version, and create
  `docs/releases/<tag>.md` with the same user-facing content as `HEAD.md`.
- After the GitHub Release is published, reset `docs/releases/HEAD.md` to just
  `# HEAD`, then commit and push that release bookkeeping change.

## Important Commands

```sh
make build
make test
make test-embedded
make test-release-light
cargo test
cargo fmt
```

Run `cargo fmt` before committing Rust/C-adjacent changes.

## Notes

The extension supports model configuration either through tokenizer arguments:

```sql
CREATE VIRTUAL TABLE docs USING fts5(
  body,
  tokenize='vaporetto model /path/to/model.zst wsconst DGR'
);
```

or through environment variables:

```sh
SQLITE_VAPORETTO_MODEL=/path/to/model.zst sqlite3
```

The default `wsconst` is `DGR`.

Common tokenizer arguments:

- `model <path>`: Use a Vaporetto `.model` or `.model.zst`.
- `wsconst <chars>`: KyTea/Vaporetto character classes not to segment. Defaults
  to `DGR`.
- `tags <prefixes>`: Comma-separated tag prefixes to index.
- `keep_untagged`: With `tags`, also keep tokens with no POS/tag data.
- `case sensitive`: Preserve ASCII case distinctions.
- `case insensitive`: Explicitly request the default ASCII case-insensitive
  behavior.

For content-oriented Japanese search indexes, a practical tag-filtered pattern
is:

```sql
CREATE VIRTUAL TABLE docs USING fts5(
  body,
  tokenize='vaporetto tags 名詞,動詞,形容詞,副詞,接頭辞,接尾辞 keep_untagged'
);
```

Use `sqlite3_vaporetto_init` when writing `.load` examples:

```sql
.load ./target/debug/libsqlite_vaporetto sqlite3_vaporetto_init
```

---
> Source: [hotchpotch/sqlite-vaporetto](https://github.com/hotchpotch/sqlite-vaporetto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
