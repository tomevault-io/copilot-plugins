## parrot-db

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Parrot-db is a toy LMDB-inspired key-value storage engine in Python. It supports MVCC transactions with snapshot isolation, and is being developed in stages toward full WAL-based durability and B+ tree indexing.

**Current Stage:** 5 (Freelist, page reclamation, and compaction) - Complete

See `storage-engine-spec.md` for the full staged development plan and `progress.md` for current status.

## Commands

```bash
uv sync                      # Install dependencies
uv run pytest                # Run all tests
uv run pytest tests/test_models.py  # Run model tests only
uv run pytest tests/test_pager.py   # Run storage tests only
uv run ruff check .          # Lint
uv run ruff format .         # Format
uv run python client.py      # Interactive REPL (legacy)
uv run python tools/db_inspect.py --db ./tmp/dev.db --summary  # Inspect database
uv run python tools/db_copy.py source.db dest.db              # Copy database
uv run python tools/db_copy.py source.db dest.db -c           # Copy with compaction
```

## Project Structure

```
parrot-db/
├── models/                  # Pydantic models for on-disk structures
│   ├── storage.py          # KeyValue, PageHeader, PageType
│   ├── wal.py              # WALEntry, WALOperation
│   └── metadata.py         # DBMetadata, MAGIC
├── storage/                 # Page-based storage layer
│   ├── pager.py            # Pager: page I/O, allocation, file layout
│   ├── pages.py            # Page types: Header, Meta, Leaf, Branch, Freelist
│   ├── freelist.py         # In-memory freelist for page reuse
│   └── btree.py            # Copy-on-write B+ tree implementation
├── tools/
│   ├── db_inspect.py       # Database inspection CLI tool
│   └── db_copy.py          # Database copy with optional compaction
├── txn/                     # Transaction layer
│   └── transaction.py      # ReadTransaction, WriteTransaction
├── tests/
│   ├── test_models.py      # Model serialization tests
│   ├── test_pager.py       # Storage layer tests
│   ├── test_btree.py       # B+ tree tests
│   ├── test_transactions.py # Transaction and MVCC tests
│   ├── test_freelist.py    # Freelist and page reclamation tests
│   └── test_db.py          # Database tests (legacy)
├── parrot_db.py            # ParrotDB: main database class
├── storage-engine-spec.md  # Staged development plan
└── progress.md             # Current progress tracker
```

## Models (Stage 1)

All models use Pydantic with `struct`-based binary serialization. Each model file contains:
- Struct format reference in the module docstring
- Format constants (e.g., `KEY_VALUE_FMT = "<IIQ"`)
- `to_bytes()` / `from_bytes()` methods

| Model | File | Purpose |
|-------|------|---------|
| `KeyValue` | storage.py | Key-value entry with MVCC version |
| `PageHeader` | storage.py | 9-byte page header (type, id, checksum) |
| `WALEntry` | wal.py | WAL record (op, key, value, txn_id, timestamp) |
| `DBMetadata` | metadata.py | 32-byte file header (magic, version, page_size, etc.) |

## Storage Layer (Stage 2)

Page-based storage with dual meta pages for atomic commits.

**File Layout:**
- Page 0: Header page (magic `PRRT`, version, page_size)
- Page 1: Meta page 0 (txn_id, root_page_id, freelist_page_id)
- Page 2: Meta page 1 (alternating meta page)
- Page 3+: Data pages (freelist, branch, leaf)

**Key Components:**

| Component | File | Purpose |
|-----------|------|---------|
| `Pager` | pager.py | Page I/O, allocation, file management |
| `HeaderPage` | pages.py | Database file header with magic bytes |
| `MetaPage` | pages.py | Root pointers and txn_id for atomic commits |
| `LeafPage` | pages.py | B+ tree leaf with key-value cells |
| `BranchPage` | pages.py | B+ tree internal node with separators |
| `FreelistPage` | pages.py | Persisted list of free page IDs |
| `Freelist` | freelist.py | In-memory free page tracking with MVCC support |

**Checksum:** All pages use CRC32 checksums computed over the entire page content.

## Freelist and Page Reclamation (Stage 5)

MVCC-aware page management with deferred freeing for snapshot isolation.

**Deferred Freeing:**
Pages orphaned by writes can't be immediately freed if readers might still reference them. The freelist tracks:
- **Free pages**: immediately available for allocation
- **Pending-free pages**: keyed by txn_id, released when oldest reader advances past that txn_id

**Page Reclamation Flow:**
1. Write transaction modifies tree, creating new pages (CoW)
2. On commit, orphaned pages (old tree - new tree) are marked pending-free
3. When readers complete, `release_pending()` moves pages to free set
4. Future allocations reuse free pages before extending file

**Database Copy and Compaction:**

```python
# Raw copy (preserves layout, includes holes)
db.copy("backup.db")

# Compact copy (contiguous pages, smaller file)
db.copy("compact.db", compact=True)
```

CLI tool:
```bash
uv run python tools/db_copy.py source.db backup.db        # Raw copy
uv run python tools/db_copy.py source.db compact.db -c    # Compact copy
```

**Key Components:**

| Method | Location | Description |
|--------|----------|-------------|
| `mark_pending_free` | Freelist | Mark pages for deferred release |
| `release_pending` | Freelist | Move releasable pages to free set |
| `collect_page_ids` | BTree | Collect all page IDs reachable from root |
| `copy` | ParrotDB | Copy database with optional compaction |
| `freelist_count` | ParrotDB | Number of immediately free pages |
| `pending_free_count` | ParrotDB | Number of pages pending free |

## B+ Tree (Stage 3)

Copy-on-write B+ tree with path copying for MVCC support.

**Key Characteristics:**
- All mutations return a new root_page_id; original tree remains valid
- Leaf nodes store key-value pairs; branch nodes store separator keys
- Uses in-order traversal for range scans (sibling pointers are not maintained due to CoW complexity)
- Automatic splitting when nodes overflow

**BTree API:**

| Method | Signature | Description |
|--------|-----------|-------------|
| `get` | `(root_page_id, key) -> value \| None` | Look up key from given root |
| `insert` | `(root_page_id, key, value) -> new_root_page_id` | Insert/update, returns new root |
| `delete` | `(root_page_id, key) -> new_root_page_id` | Delete key, returns new root (0 if empty) |
| `scan` | `(root_page_id, start, end) -> Iterator` | Iterate keys in sorted order |
| `tree_height` | `(root_page_id) -> int` | Return tree height |
| `count_keys` | `(root_page_id) -> int` | Count total keys |
| `collect_page_ids` | `(root_page_id) -> set[int]` | Collect all reachable page IDs |

**Usage Pattern:**
```python
from storage import Pager, BTree
from storage.pages import MetaPage

with Pager(path, create=True) as pager:
    btree = BTree(pager)
    root = 0  # Start with empty tree

    # Insert returns new root
    root = btree.insert(root, b"key", b"value")

    # Get using root
    value = btree.get(root, b"key")

    # Commit by writing to meta page
    meta = MetaPage(page_id=pager.get_inactive_meta_id(),
                    txn_id=1, root_page_id=root)
    pager.write_meta_page(meta)
    pager.sync()
```

## Transactions (Stage 4)

MVCC transactions with snapshot isolation via CoW B+ tree.

**Key Components:**

| Component | File | Purpose |
|-----------|------|---------|
| `ParrotDB` | parrot_db.py | Database entry point, transaction lifecycle |
| `ReadTransaction` | txn/transaction.py | Read-only snapshot at transaction start |
| `WriteTransaction` | txn/transaction.py | Accumulates changes, commits atomically |

**Transaction API:**

```python
from parrot_db import ParrotDB

db = ParrotDB(path="./data.db")

# Convenience methods (auto-transaction)
db.put(b"key", b"value")
value = db.get(b"key")

# Explicit transactions
with db.begin(write=True) as txn:
    txn.put(b"key1", b"value1")
    txn.put(b"key2", b"value2")
    txn.commit()

# Read transaction (snapshot isolation)
with db.begin() as txn:
    for key, value in txn.scan():
        print(key, value)

db.close()
```

**Concurrency Model:**
- Single writer: only one WriteTransaction can be active (enforced via lock)
- Multiple readers: ReadTransactions run concurrently
- Snapshot isolation: readers see consistent state from transaction start
- Atomic commit: meta page swap + fsync makes writes durable

## Code Style (Python 3.12+)

**Type Hints:**
- Use built-in generics: `list[int]`, `dict[str, Any]`, not `List[int]`, `Dict[str, Any]`
- Use `X | None` instead of `Optional[X]`
- Use `X | Y` instead of `Union[X, Y]`
- Prefer `Self` from `typing` for return type of methods returning the class instance
- Use `type` keyword for simple type aliases: `type PageId = int`

**Modern Python:**
- Use `match` statements for multi-branch conditionals on types/values
- Use walrus operator `:=` where it improves clarity
- Prefer `pathlib.Path` over `os.path`
- Use `contextlib.suppress()` over bare `try/except/pass`
- Use f-strings, never `.format()` or `%`

**Comments:**
- No comments explaining what code does—the code should be self-documenting
- Only comment on *why* something non-obvious is done a certain way
- No inline comments restating the line (e.g., `x += 1  # increment x`)
- No commented-out code

**Docstrings:**
- Public API: one-line summary of intent
- Classes/functions with database semantics (B+ trees, WAL, MVCC, page management): include a brief high-level description of *what* the component does and its role in the system—database internals benefit from context
- Skip docstrings for obvious helpers and private methods

**Structure:**
- Imports: stdlib → third-party → local, separated by blank lines
- One class per file when the class is substantial
- Keep functions short; extract helpers with clear names instead of adding comments
- Use `__all__` to define public API in modules

## Slash Commands

Custom skills available in this project:

| Command | Description |
|---------|-------------|
| `/next-stage` | Implement the next stage from `storage-engine-spec.md` |
| `/make-recommendations` | Review code and generate recommendations (interactive approval) |
| `/fix-next` | Process the next code recommendation from `recommendations.md` |

## Code Recommendations Workflow

Code improvements are tracked in `recommendations.md`. Each recommendation has:
- **ID**: Unique identifier (REC-001, etc.)
- **Priority**: Critical / High / Medium / Low
- **Status**: Pending / Done
- **Ambiguous**: Whether user clarification is needed

**Creating recommendations** with `/make-recommendations`:
1. Specify what to review (file, directory, or git changes)
2. Receive explained recommendations ranked by priority
3. Choose which ones to keep (all, by priority, or individually)
4. Selected items are persisted to `recommendations.md`

**Processing recommendations** with `/fix-next`:
1. Find the next pending recommendation (highest priority first)
2. Ask for clarification if marked ambiguous
3. Implement the fix
4. Run tests and linting
5. Move the recommendation to the Completed section

## Development Workflow

After completing each stage:
1. Run tests: `uv run pytest -q`
2. Run linting: `uv run ruff check .`
3. Update `progress.md` with completion status
4. Update this file (`CLAUDE.md`) with new components
5. Commit with format: `component: short description`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Souloist) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
