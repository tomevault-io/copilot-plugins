## sie-parser

> A library crate: parser, CP437 codec, label schema, diagnostic codes, and

# sie-parser — architecture notes

A library crate: parser, CP437 codec, label schema, diagnostic codes, and
typed `SieDocument` model for the [SIE 4B](../docs/spec.md) bookkeeping
file format. No LSP, async, or tokio dependencies — those live in the
sibling [`sie-lsp`](../sie-lsp) repo, which depends on this crate.

## Layout

```
Cargo.toml           publishable package manifest
justfile             build/test recipes
src/
├── lib.rs           re-exports + read_file() + offset_to_line_col()
├── types.rs         Item / Field / Span / Diagnostic
├── parser.rs        line-based tokenizer + item builder + field validator
├── labels.rs        LABELS: &[LabelSpec] — the schema for all 36 labels
├── diagnostics.rs   stable `&'static str` codes (stringly-typed on purpose)
├── cp437.rs         handrolled 128-entry CP437 high-half → Unicode table
└── document.rs      typed SieDocument model (Account, Company, FiscalYear, …)
tests/
├── sample.rs        parses the real 4080-line Visma export, asserts 0 errors
├── broken.rs        parses broken.se, asserts every diagnostic code fires
└── fixtures/
    ├── sample.se    copy of docs/SIE4 example file.SE (CP437 bytes + CRLF preserved)
    └── broken.se    handcrafted; one occurrence of each diagnostic code
```

## Load-bearing design decisions

Each of these is more expensive to change than to keep. If you're about to
undo one, re-read the reasoning first.

### 1. Schema-driven stringly-typed AST
`Item { label: String, fields: Vec<Field>, children: Vec<Item>, span }` with
validation driven by `LABELS: &[LabelSpec]` in `src/labels.rs`. **Do not**
replace this with 36 typed variants.
- The grammar is uniform — one label per line, whitespace-separated fields,
  one container (`#VER`). A schema table handles this naturally.
- Unknown labels must round-trip (spec §7.1) and unknown trailing fields
  must be accepted (§7.3). A typed AST needs an `Unknown` escape hatch
  anyway.
- `src/labels.rs` is the single source of truth for parsing, hover, and
  completion. Adding a label in a future spec rev = add a row. With typed
  variants it'd be a new struct + enum variant + match arm in every
  consumer.
- If you need richer semantic checks later (balance, cross-ref), write
  them as thin functions over `&[Item]` that pull fields by index. The
  cost is "this field name → index 2" lookups, which is fine.

### 2. Stateless `parse(input: &str) -> ParseOutput`
A full parse of the 4k-line sample is sub-millisecond in release mode — no
incremental parser, no rope, no document tree.
- `parse` never returns `Err` — all problems become `Diagnostic`s.
- Don't add a `Parser` struct with fields. Error recovery is per-line:
  emit a diagnostic, skip to next line, keep going.

### 3. Handrolled CP437 (no encoding crate)
`encoding_rs` doesn't cover CP437 (WHATWG Encoding Standard only).
`codepage-437` and `oem_cp` exist but bring in a dependency for ~60 lines
of code.
- `src/cp437.rs::CP437_HIGH` is a 128-entry `[char; 128]`. The low half is
  identical to ASCII so `if b < 0x80 { b as char }`.
- If you ever need more codepages (CP850, CP865), consider switching to
  `oem_cp`. Otherwise leave it alone.
- `detect_encoding` sniffs for `#FORMAT PC8` in the first 4 KiB, then
  falls back to UTF-8 validity. **`parse()` itself only takes UTF-8.**
  Encoding is handled at the file-reading boundary, never in the parser.

### 4. Byte-offset spans, no line/col
Every `Span { byte_offset, byte_len }` in the AST uses byte offsets into
the original source string. `offset_to_line_col` is provided as a helper
for downstream tools (e.g. `sie-lsp`) that need line/column conversion,
but the AST never pre-computes it — that would tie the AST to a
presentation layer.

### 5. Quoted strings: slice, don't build byte-by-byte
`src/parser.rs::read_quoted` scans for the terminating `"` (respecting
`\"` and `\\` escapes) and then extracts the body as a single UTF-8
`&str`, then runs `unescape_quoted_body` over it. An earlier version
pushed bytes one at a time and corrupted multi-byte UTF-8 sequences
(Swedish `Ö` came out as `Ã` + control char). If you touch this function,
make sure the sample test still decodes `Övningsbolaget AB`.

### 6. Brace handling via 2-state machine
`ParseState { TopLevel, InsideVer { parent_index, open_span } }`. SIE only
has one level of nesting (`#VER { #TRANS ... }`), so no stack is needed.
If a future spec adds more containers, generalize to a stack — but for
now the simple enum is more readable.

### 7. Diagnostic codes are stable strings
`src/diagnostics.rs` defines `&'static str` codes. These are **public
API** — the `broken.rs` test asserts the set, editors filter on them,
external tools key off them. Don't rename without a deprecation period.
Add new codes by appending to `ALL`.

### 8. Fixtures are committed intact
`tests/fixtures/sample.se` is a copy of the real Visma export — CP437
bytes and CRLF line endings preserved. **Do not** convert to UTF-8 or LF;
the whole point is to exercise the real-file path. Editors may offer to
"fix" it — decline.

### 9. Empty quoted string = absent on optional fields
In `validate_field` (`src/parser.rs`), an `""` token in an **optional**
position short-circuits validation — no `bad-date-format` / `bad-amount`
/ `bad-integer` / `bad-enum-value` is emitted. Rationale: SIE fields are
positional with no way to "skip" an intermediate optional field. Real
exporters (notably Fortnox) write `""` as a placeholder so they can reach
later positional fields like `quantity`. Example: `#TRANS 1930 {} -91.05
"" "" 0` — the two `""` placeholders stand in for `transdate` and
`transtext` so `quantity=0` lands in slot 6. The spec (§11 #TRANS.1)
makes those fields optional; §5.10 mandates `YYYYMMDD` for dates but is
silent on the empty-placeholder case. Every SIE reader in the wild
accepts it, so we do too.
- Required fields still error on `""` (e.g. `#VER`'s `verdate`, `#GEN`'s
  `date`). The lenience is gated on `!spec.required`.
- `FieldKind::ObjectList` is excluded — empty there is `{}`, not `""`.
- Tests: `empty_quoted_placeholder_on_optional_field` and
  `empty_quoted_still_errors_on_required_field` in `parser.rs`.

## Adding a new label

Spec amendments or vendor extensions:
1. Add a `LabelSpec` entry in `src/labels.rs` (with `description`,
   `format`, `fields`).
2. Update `labels::tests::has_thirty_six_labels` count.
3. Add a line to `tests/sample.rs` or `tests/broken.rs` if the new label
   exercises a new diagnostic path.
4. No changes to `parser.rs` should be needed — it drives off the schema.

## Adding a new diagnostic code

1. Add a `pub const FOO: &str = "foo";` to `src/diagnostics.rs` and append
   to `ALL`.
2. Emit it from wherever the check lives (usually `parser.rs::build_item`
   or `validate_field`).
3. Add a trigger to `tests/fixtures/broken.se` that hits the new code.
4. The `broken.rs` test uses `dc::ALL` so it auto-includes new codes.

## Deferred / explicitly out of scope

Don't implement these without re-checking with the user:

- **Semantic validation**: verifications balancing to zero, `#TRANS`
  account existing in `#KONTO`, `#RAR` year ranges being non-overlapping,
  `#KSUMMA` CRC-32 verification, `#RTRANS` / `#TRANS` pairing (spec §11
  #RTRANS point 4).
- **Incremental parsing**: never needed for files this small.

## Things that have bitten me

- **Integer fields accept negative numbers** (because `#RAR` uses
  `year_no: 0, -1, -2`). If you add a field that must be non-negative
  (e.g. account numbers), consider a `FieldKind::NonNegativeInteger`
  variant rather than changing `FieldKind::Integer`.
- **Tab inside a quoted string is a control char** per spec §5.7. This
  is caught by the `b < 0x20` check. Real files never have this but the
  broken.se fixture does.
- **`#VER` without a brace block** is silently tolerated (the container
  flag is for detecting the `{` line that follows). If the next line
  isn't `{`, `#VER` becomes a top-level item with no children and no
  diagnostic. That matches real files where `#VER` can appear with its
  sub-entries inline on rare occasions. Don't turn this into an error
  without spec backing.

## Verification checklist

Before considering a change complete:

```sh
just test         # all must pass
just build        # release build
```

For changes that affect the LSP-facing API, also run the sibling
`sie-lsp` repo's tests against this crate via the
`[patch.crates-io]` override.

---
> Source: [t4t5/sie-parser](https://github.com/t4t5/sie-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
