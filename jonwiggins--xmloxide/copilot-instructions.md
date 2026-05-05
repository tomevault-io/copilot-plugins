## xmloxide

> **xmloxide** is a pure Rust reimplementation of libxml2 — the de facto standard XML/HTML parsing library in the open-source world. libxml2 became officially unmaintained in December 2025 with known security issues. xmloxide is a memory-safe, high-performance replacement that passes the same conformance test suites.

# CLAUDE.md — xmloxide project guide

## Project Overview

**xmloxide** is a pure Rust reimplementation of libxml2 — the de facto standard XML/HTML parsing library in the open-source world. libxml2 became officially unmaintained in December 2025 with known security issues. xmloxide is a memory-safe, high-performance replacement that passes the same conformance test suites.

**Version:** 0.4.1
**License:** MIT
**MSRV:** Rust 1.81+

### Achieved Goals

- Full conformance with W3C XML 1.0 (Fifth Edition) and Namespaces in XML 1.0
- **1727/1727** W3C XML Conformance Test Suite tests passing (100%)
- **119/119** libxml2 regression tests passing (100%)
- Feature parity with libxml2's core: XML/HTML parsing, DOM, SAX2, XPath 1.0+, XmlReader, push parser, DTD/RelaxNG/XSD/Schematron validation, C14N, XInclude, XML Catalogs
- **WHATWG HTML5 parser** — full HTML Living Standard tokenizer (§13.2.5) and tree builder (§13.2.6) with 7032/7032 tokenizer tests + 1778/1778 tree construction tests passing (100% html5lib-tests)
- **CSS selector engine** — query elements with familiar CSS syntax including combinators and pseudo-classes
- **Serde integration** — optional XML (de)serialization to/from Rust types via serde
- **Async parsing** — optional async parsing from `tokio::io::AsyncRead` sources
- Zero `unsafe` in public API surface (`unsafe_code = "deny"` in Cargo.toml)
- No system dependencies — pure Rust (uses `encoding_rs` for character encoding)
- C/C++ FFI layer with header file (`include/xmloxide.h`)
- WASM bindings (`xmloxide-wasm`) and Python bindings (`pyxmloxide`)
- `xmllint` CLI tool
- Performance competitive with libxml2 (serialization 1.5-2.3x faster)

### Non-Goals

- XSLT (that's libxslt, a separate project)
- XML 1.1 support (rarely used, can add later)
- Full libxml2 API-level compatibility (we design an idiomatic Rust API)

---

## Architecture

### Tree Representation (the critical design decision)

libxml2 uses a web of raw C pointers (parent, children, next, prev, doc, ns). We use **arena allocation with typed indices**:

- `Document` owns a `Vec<NodeData>` — all nodes live here
- `NodeId` is a `#[repr(transparent)]` newtype over a `NonZeroU32` index
- Navigation (parent, first_child, next_sibling, prev_sibling) stored as `Option<NodeId>` fields on each `NodeData`
- This gives us O(1) node access, cache-friendly layout, no reference counting overhead, no borrow checker fights, and safe freeing (drop the Document, everything is freed)
- Trade-off: individual node removal requires a free-list or tombstone approach

**Why not `Rc<RefCell<>>`:** Reference cycles between parent/child require weak refs, runtime borrow panics are possible, and per-node allocation is slow.

**Why not raw pointers behind unsafe:** We want to minimize unsafe surface area. Arena indices give us the same performance with compile-time safety.

### Module Map

```
src/
├── lib.rs              # Public API re-exports, crate docs
├── bin/
│   └── xmllint.rs      # xmllint CLI (behind "cli" feature)
├── tree/
│   ├── mod.rs          # Document, NodeId, NodeData, tree navigation, id_map
│   └── node.rs         # NodeKind enum (element/text/comment/PI/doctype/etc)
├── parser/
│   ├── mod.rs          # ParseOptions, top-level parse functions
│   ├── xml.rs          # XML 1.0 parser (core state machine)
│   ├── push.rs         # Push/incremental parser wrapper
│   └── input.rs        # Parser input stack (entity expansion, includes)
├── html/
│   ├── mod.rs          # HTML 4.01 parser (error-tolerant, auto-closing, void elements)
│   └── entities.rs     # HTML named character references
├── html5/
│   ├── mod.rs          # WHATWG HTML5 parser public API and module docs
│   ├── tokenizer.rs    # HTML5 tokenizer (all 80 states per §13.2.5)
│   ├── tree_builder.rs # HTML5 tree construction (all insertion modes per §13.2.6)
│   ├── sax.rs          # HTML5 SAX-like streaming API (no DOM tree)
│   └── entities.rs     # HTML5 named character references (§13.5, 2231 entries)
├── css/
│   ├── mod.rs          # CSS selector public API (select, select_with)
│   ├── parser.rs       # CSS selector parser
│   ├── types.rs        # Selector AST types
│   └── eval.rs         # Selector evaluator against document trees
├── sax/
│   └── mod.rs          # SAX2 handler trait and streaming parser
├── reader/
│   └── mod.rs          # XmlReader pull-based parsing API
├── encoding/
│   └── mod.rs          # Encoding detection, BOM sniffing, encoding_rs bridge
├── xpath/
│   ├── mod.rs          # XPath public API, evaluate() entry point
│   ├── ast.rs          # XPath AST types (Expr, Step, Axis, etc.)
│   ├── lexer.rs        # XPath expression tokenizer
│   ├── parser.rs       # XPath expression parser → AST
│   ├── eval.rs         # Expression evaluator against a node tree
│   ├── types.rs        # XPath value types (NodeSet, String, Number, Boolean)
│   └── regex.rs        # XPath regex support (matches, replace, tokenize)
├── validation/
│   ├── mod.rs          # ValidationResult, ValidationError
│   ├── dtd.rs          # DTD parsing and validation (populates id_map)
│   ├── relaxng.rs      # RelaxNG schema parsing and validation
│   ├── xsd.rs          # XML Schema (XSD) parsing and validation
│   └── schematron.rs   # ISO Schematron (ISO/IEC 19757-3) rule-based validation
├── serial/
│   ├── mod.rs          # Serialization options and entry points
│   ├── xml.rs          # XML serializer
│   ├── html.rs         # HTML 4.01 + HTML5 serializers (void elements, attribute rules)
│   └── c14n.rs         # Canonical XML (C14N 1.0 / Exclusive C14N)
├── xinclude/
│   └── mod.rs          # XInclude 1.0 document inclusion
├── catalog/
│   └── mod.rs          # OASIS XML Catalogs for URI resolution
├── serde_xml/
│   ├── mod.rs          # Serde XML module root (feature-gated behind "serde")
│   ├── de.rs           # XML deserializer
│   ├── ser.rs          # XML serializer
│   └── error.rs        # Serde error types
├── async_xml.rs        # Async parsing via tokio::io::AsyncRead (feature-gated "async")
├── ffi/
│   ├── mod.rs          # FFI module root (feature-gated behind "ffi")
│   ├── document.rs     # Document parsing FFI
│   ├── tree.rs         # Tree navigation FFI
│   ├── serial.rs       # Serialization FFI
│   ├── xpath.rs        # XPath evaluation FFI
│   ├── validation.rs   # DTD/RelaxNG/XSD/Schematron validation FFI
│   ├── css.rs          # CSS selector FFI
│   ├── sax.rs          # SAX2 streaming FFI
│   ├── reader.rs       # XmlReader FFI
│   ├── push.rs         # Push parser FFI
│   ├── c14n.rs         # Canonical XML FFI
│   ├── catalog.rs      # XML Catalogs FFI
│   ├── html5.rs        # HTML5 parsing FFI
│   ├── xinclude.rs     # XInclude FFI
│   └── strings.rs      # String memory management FFI
├── error/
│   └── mod.rs          # Error types, ParseError, ParseDiagnostic
└── util/
    ├── mod.rs
    ├── dict.rs          # String interning dictionary
    └── qname.rs         # QName (prefix:localname) handling
```

### Key Design Decisions

- **DTD `validate()` takes `&mut Document`** — because it populates the `id_map` on `Document` when ID-type attributes are found. This enables `element_by_id()` and the XPath `id()` function.
- **The FFI module is feature-gated** behind `#[cfg(feature = "ffi")]` to avoid pulling in C interop code unless needed.
- **The CLI is feature-gated** behind `cli` (which is a default feature) using `dep:clap`.

---

## Coding Standards

### Formatting

All code is formatted with `rustfmt`. Configuration is in `rustfmt.toml` at the project root. Run before every commit:

```sh
cargo fmt --all
```

CI will reject unformatted code.

### Linting

All code must pass `clippy` at the **pedantic** level with zero warnings. Configuration is in `Cargo.toml` under `[lints]`.

```sh
cargo clippy --all-targets --all-features -- -D warnings
```

Key clippy policies:
- `pedantic` is enabled globally — we selectively `allow` specific pedantic lints only with justification
- `unwrap()` and `expect()` are **warned** in library code (use `?` or return `Result`)
- `unwrap()` is permitted in tests (via `#[allow(clippy::unwrap_used)]` on test modules)
- `unsafe_code` is **denied** crate-wide

### Naming Conventions

- Types: `PascalCase` — `NodeId`, `Document`, `ParseOptions`
- Functions/methods: `snake_case` — `parse_str`, `root_element`, `first_child`
- Constants: `SCREAMING_SNAKE_CASE` — `MAX_ENTITY_DEPTH`, `DEFAULT_BUF_SIZE`
- Feature flags: `kebab-case` in Cargo.toml, `snake_case` in `#[cfg(feature = "...")]`
- Module files: `snake_case.rs`
- Private helpers: prefixed with `_` only if needed to avoid dead-code warnings; prefer `pub(crate)` visibility over private-with-underscore
- NodeId: always use the newtype, never raw `usize` or `u32`

### API Design Principles

- **Parse functions return `Result<Document, ParseError>`** — never panic
- **Navigation returns `Option<NodeId>`** — missing children/siblings are None, not errors
- **Iterators for traversal** — `children()`, `ancestors()`, `descendants()`, `attributes()` all return iterators
- **`Document` is the owner** — all tree mutation goes through `&mut Document` methods
- **Builder pattern for options** — `ParseOptions::default().recover(true).no_blanks(true)`
- **Zero-copy where possible** — string interning via `Dict`, return `&str` references into the arena
- **No global state** — unlike libxml2, no `xmlInitParser()` / `xmlCleanupParser()` global init. Each `Document` is self-contained and `Send + Sync`.

### Error Handling

- Define domain-specific error enums, not stringly-typed errors
- Hand-implement error traits (no `thiserror` dependency)
- The parser supports **error recovery mode**: collect errors into a `Vec<ParseDiagnostic>` while still producing a (possibly partial) tree — this is critical because it's how most real-world users consume libxml2
- Errors carry **source location** (line, column, byte offset) matching libxml2's error reporting
- Error severity levels: `Warning`, `Error`, `Fatal` (matching libxml2's `xmlErrorLevel`)

### Documentation

- Every public type, trait, method, and function has a doc comment
- Doc comments include a brief one-line summary, then a blank line, then details
- Include `# Examples` sections for key API entry points
- Reference the relevant spec section where applicable (e.g., "See XML 1.0 §2.3 Common Syntactic Constructs")
- Module-level docs (`//!` comments in `mod.rs`) explain the module's role and design rationale
- Use backtick-wrapped names for identifiers and abbreviations in doc comments (e.g., `XPath`, `id_map`) to satisfy `clippy::doc_markdown`

### Testing

- Unit tests go in the same file as the code, in a `#[cfg(test)] mod tests { ... }` block
- Integration tests go in `tests/`
- Test names follow `test_<function>_<scenario>` — e.g., `test_parse_str_empty_document`, `test_node_append_child_to_leaf`
- Use `pretty_assertions` for comparing large strings/structures in tests
- Every parser feature needs a roundtrip test: parse → serialize → parse again → assert tree equality
- Prefer `let...else { panic!("expected X, got {:?}", actual) }` over `if let...else { panic!("expected X") }` for better failure diagnostics

### Performance

- The string dictionary (`Dict`) is load-bearing — hot-path string comparisons must go through interned `SymbolId` equality, not `str` comparison
- The parser should be zero-copy for attribute values and text content where possible (reference into the input buffer)
- Benchmark critical paths: `benches/parser_bench.rs` compares against known documents
- `benches/comparison_bench.rs` provides head-to-head comparison with libxml2 (requires `--features bench-libxml2`)
- Profile before optimizing — don't prematurely complicate the architecture

### Git Conventions

- Commit messages: `<module>: <imperative summary>` — e.g., `tree: add NodeId newtype with NonZeroU32`, `parser: implement element start tag parsing`
- One logical change per commit
- Feature branches: `feature/<module>` — e.g., `feature/tree`, `feature/parser`

---

## Key Specifications & References

These are the specs we implement against:

- [XML 1.0 (Fifth Edition)](https://www.w3.org/TR/xml/) — the core spec
- [Namespaces in XML 1.0](https://www.w3.org/TR/xml-names/) — namespace processing
- [XPath 1.0](https://www.w3.org/TR/xpath-10/) — XPath query language (with selected XPath 2.0 functions)
- [ISO Schematron](https://www.iso.org/standard/40833.html) — ISO/IEC 19757-3 rule-based validation
- [XML Inclusions (XInclude) 1.0](https://www.w3.org/TR/xinclude/) — document merging
- [Canonical XML 1.0](https://www.w3.org/TR/xml-c14n/) — canonical serialization
- [RelaxNG](https://relaxng.org/spec-20011203.html) — schema language
- [W3C XML Schema](https://www.w3.org/TR/xmlschema-1/) — XSD 1.0
- [HTML 4.01](https://www.w3.org/TR/html401/) — libxml2-compatible HTML parser
- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/) — HTML5 tokenizer (§13.2.5), tree construction (§13.2.6), named character references (§13.5)
- [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986) — URI syntax
- [W3C XML Conformance Test Suite](https://www.w3.org/XML/Test/) — 1727 applicable test files
- [libxml2 source](https://gitlab.gnome.org/GNOME/libxml2) — the reference implementation

---

## Build & Test Commands

```sh
# Format everything
cargo fmt --all

# Lint everything (must pass with zero warnings)
cargo clippy --all-targets --all-features -- -D warnings

# Run all tests
cargo test --all-features

# Run tests for a specific module
cargo test --all-features -- tree::
cargo test --all-features -- parser::
cargo test --all-features -- validation::dtd
cargo test --all-features -- serial::html

# Run benchmarks
cargo bench

# Run benchmarks against libxml2
cargo bench --features bench-libxml2

# Check without building (fast feedback)
cargo check --all-features

# Build docs
cargo doc --all-features --no-deps --open

# Run conformance suite (requires download first)
./scripts/download-conformance-suite.sh
cargo test --test conformance

# Run libxml2 compatibility suite (requires download first)
./scripts/download-libxml2-tests.sh
cargo test --test libxml2_compat

# Run html5lib test suites (requires download first)
./scripts/download-html5lib-tests.sh
cargo test --test html5lib_tokenizer
cargo test --test html5lib_tree_construction

# Run fuzz targets
cargo +nightly fuzz run fuzz_xml_parse
cargo +nightly fuzz run fuzz_html_parse
cargo +nightly fuzz run fuzz_html5_parse
cargo +nightly fuzz run fuzz_html5_fragment
cargo +nightly fuzz run fuzz_xpath
cargo +nightly fuzz run fuzz_roundtrip
cargo +nightly fuzz run fuzz_sax
cargo +nightly fuzz run fuzz_reader
cargo +nightly fuzz run fuzz_push
cargo +nightly fuzz run fuzz_validation
cargo +nightly fuzz run fuzz_schematron
```

---

## Common Patterns

### Adding a new node type

1. Add variant to `NodeKind` enum in `tree/node.rs`
2. Add constructor method on `Document` in `tree/mod.rs`
3. Add serialization case in `serial/xml.rs` (and `serial/html.rs` if applicable)
4. Add parsing case in `parser/xml.rs`
5. Add tests for parse → serialize roundtrip

### Adding a new parser feature

1. Reference the exact spec section in a comment
2. Implement in `parser/xml.rs` (or appropriate sub-module)
3. Add unit tests covering valid input, malformed input (error recovery), and edge cases
4. Add a conformance test if the W3C suite covers it
5. Run the full test suite to check for regressions

### Error recovery pattern

```rust
// The parser supports recovering from errors. When `ParseOptions::recover` is
// set, the parser logs the error and attempts to continue. This pattern is
// used throughout:
fn parse_something(&mut self) -> Result<NodeId, ParseError> {
    match self.try_parse_something() {
        Ok(node) => Ok(node),
        Err(e) if self.options.recover => {
            self.push_diagnostic(e.into());
            self.skip_to_recovery_point();
            // Return a best-effort node or skip
            Ok(self.create_error_placeholder())
        }
        Err(e) => Err(e),
    }
}
```

---

## Dependencies Policy

- **Minimize dependencies.** This is a foundational library — every dep is a supply chain risk.
- **Required:** `encoding_rs` (character encoding — replaces iconv/ICU, well-audited Mozilla crate)
- **CLI only:** `clap` (behind `cli` feature, default-on)
- **Benchmarks only:** `libxml` (behind `bench-libxml2` feature, off by default)
- **Dev only:** `criterion` (benchmarks), `pretty_assertions` (test output)
- **Explicitly rejected:** `thiserror` (we hand-implement error types), `serde` (not needed in core — add as optional feature later), `regex` (we implement our own pattern matching for XPath/validation), `nom`/`pest`/`winnow` (we write a hand-rolled recursive descent parser for performance and error recovery, matching libxml2's approach)

The parser is hand-rolled recursive descent (not combinator-based) because:
1. libxml2's parser is recursive descent and we need identical behavior for conformance
2. Error recovery requires fine-grained control over the parse state
3. Push/incremental parsing requires suspendable state, which combinators make awkward
4. Performance — no abstraction overhead

---

## CI/CD

Three GitHub Actions workflows:

- **ci.yml** — runs on every push/PR: formatting, clippy, tests (stable + MSRV 1.81), doc build
- **compat.yml** — weekly: downloads libxml2 test suite, runs full compatibility tests
- **bench.yml** — on every PR: runs parser benchmarks, tracks regression (fails at >115%)

Pre-commit hooks available via `./scripts/install-hooks.sh` (runs fmt, clippy, tests, doc build).

---

## Test Suites

| Suite | Location | Tests | Notes |
|-------|----------|-------|-------|
| Unit tests | `src/**/*.rs` (inline) | 1020+ | All modules |
| W3C Conformance | `tests/conformance.rs` | 1727/1727 | Requires `download-conformance-suite.sh` |
| libxml2 Compat | `tests/libxml2_compat.rs` | 119/119 | Requires `download-libxml2-tests.sh` |
| html5lib Tokenizer | `tests/html5lib_tokenizer.rs` | 7032/7032 | Requires `download-html5lib-tests.sh` |
| html5lib Tree Construction | `tests/html5lib_tree_construction.rs` | 1778/1778 | Requires `download-html5lib-tests.sh` |
| HTML5 Integration | `tests/html5_integration.rs` | 22 | Parsing, fragment, entities, serialization |
| Real-world XML | `tests/real_world_xml.rs` | 22 | Atom, SVG, XHTML, Maven, SOAP, validation |
| Security/DoS | `tests/security.rs` | 15 | Billion laughs, deep nesting, etc. |
| Entity resolver | `tests/entity_resolver.rs` | 14 | Entity expansion edge cases |
| FFI | `tests/ffi_tests.rs` | 138+ | C API surface tests (SAX, Schematron, CSS) |
| Schematron | `tests/schematron.rs` | 11 | PO schema, phases, firing rules, namespaces |
| Property tests | `tests/property_tests.rs` | — | Proptest roundtrip invariants |
| UBL Validation | `tests/ubl_validation.rs` | 2 | Real-world UBL 2.4 XSD (requires download) |
| Fuzz targets | `fuzz/` | 11 targets | XML, HTML, HTML5, XPath, roundtrip, SAX, reader, push, validation, schematron |

---
> Source: [jonwiggins/xmloxide](https://github.com/jonwiggins/xmloxide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
