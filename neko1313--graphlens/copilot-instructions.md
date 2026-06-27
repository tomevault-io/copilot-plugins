## graphlens

> graphlens/                      ← uv workspace root (also the core library)

# graphlens — Architecture Guide

## Project layout

```
graphlens/                      ← uv workspace root (also the core library)
  src/graphlens/                ← core: models, contracts, registry, exceptions, utils
  packages/
    graphlens-python/           ← Python language adapter
    graphlens-typescript/       ← TypeScript language adapter
    graphlens-go/               ← Go language adapter (structure + go.mod)
    graphlens-rust/             ← Rust language adapter (structure + Cargo.toml)
    graphlens-php/              ← PHP language adapter (tree-sitter + PHPantom)
    graphlens-cli/              ← CLI: analyze / query / visualize / neo4j
  tests/                         ← core library tests
  examples/                      ← standalone usage examples (no CLI dep)
    demo_resolved_graph.py      ← Python: print node/edge stats + find-usages
    demo_resolved_graph_ts.py   ← TypeScript: same, via TypeScript Compiler API
    visualize_graph.py          ← standalone HTML graph viewer (vis.js)
    neo4j_export.py             ← standalone Neo4j export script
```

All packages use the `src/` layout and `uv_build` as build backend.

Each language adapter follows this internal layout:
```
packages/graphlens-<lang>/
  src/graphlens_<lang>/
    __init__.py              ← exports <Lang>Adapter (+ <Lang>Resolver if public)
    _adapter.py              ← LanguageAdapter subclass + _analyze_root()
    _visitor.py              ← ASTVisitor + ImportClassifier + OccurrenceRef
    _resolver.py             ← SymbolResolver subclass (e.g. TyResolver)
    _deps.py                 ← DependencyFileParser implementations + default list
    _project_detector.py     ← is_<lang>_project(), find_<lang>_roots(), detect_project_name()
    _module_resolver.py      ← file→qualified_name, source root detection
```

---

## Core principles

### 1. Adapters are pure data producers
An adapter parses source files and returns a `GraphLens`. It never writes to
any backend, database, or file system. The graph is the only output.

### 2. graphlens core is minimal
`core` lives at the workspace root under `src/graphlens/`. It contains only:
models, contracts (ABCs), registry, exceptions, utils.
No pipeline, no orchestration, no I/O. Orchestration belongs in a separate
package or in user code.

**Non-goals (explicit scope boundary).** graphlens produces a graph IR and
stops there. It does NOT: persist state or own a database (backends are a
separate consuming layer); watch the filesystem or re-index incrementally on
its own (scans are pure; deterministic IDs enable, but the caller drives,
incremental updates); compute embeddings, semantic search, or relevance
ranking; provide a UI or an agent runtime (`visualize` emits static HTML,
`mcp` exposes query tools — neither hosts a long-running service). These belong
to tools built on top of graphlens. See `website/docs/intro.md` → "Scope &
Non-goals".

### 3. SQLAlchemy dialect pattern for adapters
Adapters register themselves via `importlib.metadata` entry points:

```toml
# In the adapter's pyproject.toml
[project.entry-points."graphlens.adapters"]
python = "graphlens_python:PythonAdapter"
```

Callers resolve adapters through the registry — no direct imports needed:

```python
from graphlens import adapter_registry
adapter = adapter_registry.load("python")()
graph = adapter.analyze(project_root)
```

### 4. Tree-sitter + type-aware resolver
Every adapter uses Tree-sitter for structure extraction, occurrence roles
(call/read/write/annotation/base), and spans. A language-specific
`SymbolResolver` handles type-aware resolution — mapping occurrence positions
to definition nodes and emitting CALLS/REFERENCES/HAS_TYPE/INHERITS_FROM
edges. Tree-sitter is no longer the sole engine; it hands off precise position
data that the resolver consumes.

The Python adapter's `TyResolver` spawns a `ty server` LSP subprocess (Astral
ty, Rust-based); files are opened lazily on the first query per file, and
`open_file()` drains `publishDiagnostics` before returning so definition
queries never block on background analysis. The TypeScript
adapter's `TsResolver` is a Node-subprocess Compiler-API resolver that batches
all occurrence queries into a single `resolve_all` call to a bundled
`ts_resolver.js` script, installing typescript on-demand into a cache dir.

Parser setup (one module-level singleton per adapter):
```python
import tree_sitter_<lang> as ts_lang
from tree_sitter import Language, Parser, Node as TSNode

_LANGUAGE = Language(ts_lang.language())
_parser = Parser(_LANGUAGE)

def parse_<lang>(source: bytes) -> tree_sitter.Tree:
    return _parser.parse(source)
```

### 5. Visitor pattern: dispatch by node.type
```python
class <Lang>ASTVisitor:
    def visit(self, node: TSNode) -> None:
        handler = getattr(self, f"_visit_{node.type}", None)
        if handler:
            handler(node)
        else:
            self._visit_children(node)

    def _visit_children(self, node: TSNode) -> None:
        for child in node.children:
            self.visit(child)
```

All state lives on three stacks pushed/popped as scope changes:
- `_scope_stack: list[str]` — qualified name prefix
- `_container_stack: list[str]` — current parent node ID
- `_kind_stack: list[NodeKind]` — to distinguish METHOD vs FUNCTION

The visitor receives an `ImportClassifier` (see §9) and must set
`metadata["origin"]` on every `IMPORT` node.

The visitor also records `metadata["name_span"]` on every structural node
(CLASS, FUNCTION, METHOD, VARIABLE, ATTRIBUTE, TYPE_ALIAS, PARAMETER) so
the `SpanIndex` can map a definition position back to a node ID. During
traversal the visitor collects `OccurrenceRef` objects for every use-site,
each carrying a role (`call` / `read` / `write` / `annotation` / `base`),
the 1-based position of the name token, and the enclosing node ID. The
visitor does **not** emit CALLS, INHERITS_FROM, REFERENCES, or HAS_TYPE
directly — those edges are produced by the adapter's post-visit resolution
pass (see §9).

### 6. Deterministic node IDs
```python
# src/graphlens/utils/ids.py
def make_node_id(project_name: str, qualified_name: str, kind: str) -> str:
    key = f"{project_name}::{kind}::{qualified_name}"
    return hashlib.sha256(key.encode()).hexdigest()[:16]
```

Same inputs → same ID across re-scans. Enables incremental updates.

### 7. Multi-language / monorepo support
`LanguageAdapter.can_handle(root)` must return `True` for multi-language
projects even when language-specific marker files live in sub-directories.

`analyze(root)` without explicit `files` must:
1. Call `find_<lang>_roots(root)` to locate actual project sub-roots
2. Detect project name and source roots **relative to each sub-root**
3. Create one `PROJECT` node per sub-root in the shared `GraphLens`

Root discovery must not stop just because `root` itself has a language marker.
If `root` is both a project and a monorepo, return `root` **and** every valid
nested project root for the same language. While analyzing a parent root,
exclude files that belong to nested project roots so child projects are not
also modeled as modules of the parent.

This ensures module qualified names and import mappings are correct when
`root` is a monorepo containing multiple independent projects.

### 8. Spans are 1-based
Tree-sitter positions are 0-based `(row, col)`. Convert to 1-based when
constructing `Span(start_line, start_col, end_line, end_col)`:
```python
Span(
    start_line=node.start_point[0] + 1,
    start_col=node.start_point[1] + 1,
    end_line=node.end_point[0] + 1,
    end_col=node.end_point[1] + 1,
)
```

### 9. Import origin classification and occurrence-driven resolution

Every `IMPORT` node must have `metadata["origin"]` set to one of:

| Value | Meaning |
|-------|---------|
| `"stdlib"` | Language standard library (os, sys, path, …) |
| `"internal"` | Module declared within the same project |
| `"third_party"` | Package listed in a dependency manifest |
| `"unknown"` | None of the above (transitive dep, missing, etc.) |

Full analysis pipeline in `_analyze_root()`:

**Before visiting any file (pre-pass):**

1. **Internal** — derive top-level module names from file paths via the module
   resolver (no source parsing needed).
2. **Third-party** — run all `DependencyFileParser` instances that `can_parse()`
   the project root, union results.
3. **Stdlib** — language built-ins (e.g. `sys.stdlib_module_names` for Python).

Build an `ImportClassifier(stdlib, third_party, internal)` and pass it to the
visitor. Relative imports are always `"internal"` regardless of the classifier.

For `"internal"` imports: resolve `RESOLVES_TO` to the existing `MODULE` node
when it is present in the graph; fall back to `EXTERNAL_SYMBOL` so the edge
is never missing when the target file hasn't been processed yet.

**After visiting all files (resolution pass):**

4. Build a `SpanIndex` from the completed graph — this is the
   location→node bridge that maps any `(file_path, line, col)` to the node
   whose `name_span` contains that position.
5. Call `SymbolResolver.prepare(project_root, files)` to initialise the
   type-aware engine (e.g. ty for Python, tsc for TypeScript).
6. For each `OccurrenceRef` collected by the visitor, call
   `SymbolResolver.definition_at(file, line, col)` to resolve the use-site to
   its definition. The current resolution pass uses `definition_at` for every
   occurrence role; `infer_type_at()` is available on the contract for type
   inference but is not invoked by this pass. The returned `ResolvedRef.origin`
   field carries the same four-value vocabulary (`"stdlib"` / `"internal"` /
   `"third_party"` / `"unknown"`) — now derived from the resolver, not from the
   import manifest.
7. Use `SpanIndex.at(file_path, line, col)` to look up the target declaration
   node. If found, emit the appropriate edge (CALLS, REFERENCES, HAS_TYPE,
   INHERITS_FROM) between the `enclosing_id` from the occurrence and the
   resolved node. If not found, fall back to an `EXTERNAL_SYMBOL` node.

`DependencyFileParser` rules:
- One parser per file format — compose via a `<LANG>_DEFAULT_DEP_PARSERS` list.
- Different package managers (poetry vs pip-tools; pnpm vs yarn) get separate
  parsers or configurable key-paths within one parser.
- Include dev/test groups so test imports classify as `third_party`.
- Return `frozenset()` on any error — never raise.
- Use `normalize_pkg_name()` from `graphlens` for consistent comparison:
  lowercase, hyphens→underscores, strip extras/version specifiers.

Adapters expose `dep_parsers` as a constructor parameter so callers can inject
custom parsers for non-standard setups without subclassing the adapter.

### 10. File collection belongs to the adapter
`LanguageAdapter` provides `file_extensions()` and a default `collect_files()`
implementation. Callers must not build file lists manually. Pass explicit
`files=` only for incremental updates or custom filtering.

---

## Graph model

```
NodeKind:     PROJECT  MODULE  FILE  CLASS  METHOD  FUNCTION
              PARAMETER  IMPORT  EXTERNAL_SYMBOL  DEPENDENCY
              VARIABLE  ATTRIBUTE  TYPE_ALIAS

RelationKind: CONTAINS  DECLARES  IMPORTS  RESOLVES_TO
              CALLS  REFERENCES  INHERITS_FROM  DEPENDS_ON  HAS_TYPE
```

Typical hierarchy built by an adapter:

```
PROJECT
  └─(CONTAINS)─ MODULE (top-level)
                  └─(CONTAINS)─ MODULE (nested)
                                  └─(CONTAINS)─ FILE
                                                  └─(DECLARES)─ CLASS
                                                                  └─(DECLARES)─ METHOD
                                                  └─(DECLARES)─ FUNCTION
                                                  └─(DECLARES)─ VARIABLE / ATTRIBUTE / TYPE_ALIAS
                                                  └─(DECLARES)─ IMPORT ─(RESOLVES_TO)─ MODULE (internal)
                                                                        └─(RESOLVES_TO)─ EXTERNAL_SYMBOL (stdlib/third_party/unknown)
FUNCTION/METHOD ─(CALLS)─ FUNCTION/METHOD (resolved) or EXTERNAL_SYMBOL
CLASS ─(INHERITS_FROM)─ CLASS (resolved) or EXTERNAL_SYMBOL
FUNCTION/METHOD/PARAMETER ─(HAS_TYPE)─ CLASS (resolved) or EXTERNAL_SYMBOL
FILE/FUNCTION/METHOD ─(REFERENCES)─ VARIABLE/ATTRIBUTE (resolved) or EXTERNAL_SYMBOL
```

EXTERNAL_SYMBOL always carries `metadata["origin"]` = `"stdlib"` | `"third_party"` |
`"internal"` (fallback when MODULE node not yet in graph) | `"unknown"`.

---

## Adding a new language adapter

Key checklist:
1. `packages/graphlens-<lang>/` with `src/` layout
2. `tree-sitter>=0.24` + `tree-sitter-<lang>` + type-aware resolver dep in dependencies
3. Entry point `"graphlens.adapters"` → `"<lang>"`
4. `LanguageAdapter` subclass: `language()`, `can_handle()`, `file_extensions()`, `analyze()`
5. `find_<lang>_roots()` for monorepo support
6. Root discovery includes both root and nested same-language projects, and
   parent analysis excludes files from nested roots
7. `_deps.py`: `DependencyFileParser` implementations + `<LANG>_DEFAULT_DEP_PARSERS`
8. `ImportClassifier` pre-pass in `_analyze_root()`, `origin` on every IMPORT node
9. Adapter accepts `dep_parsers` constructor param for custom override
10. Visitor: dispatch by `node.type`, three stacks, `make_node_id` for IDs; records
    `metadata["name_span"]` on structural nodes; collects `OccurrenceRef`s (does NOT
    emit CALLS/INHERITS_FROM/REFERENCES/HAS_TYPE directly)
11. `_resolver.py`: `SymbolResolver` subclass; never raises — all errors return None/[]
12. Post-visit resolution pass: build `SpanIndex`, call `resolver.prepare()`, then
    for each occurrence call `definition_at()` / `infer_type_at()` and emit edges
13. Tests mirror `packages/graphlens-python/tests/` structure including `test_<lang>_deps.py`

## Adding a dependency parser for an existing adapter

Add a new `DependencyFileParser` subclass in the adapter's `_deps.py`, append
it to the `<LANG>_DEFAULT_DEP_PARSERS` list. Return `frozenset()` on any error.

---
> Source: [Neko1313/graphlens](https://github.com/Neko1313/graphlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
