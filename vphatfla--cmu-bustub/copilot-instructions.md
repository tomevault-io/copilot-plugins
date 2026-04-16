## cmu-bustub

> > **CLAUDE CODE DIRECTIVE:** Automatically update this context file as you work. Add new findings, code patterns, gotchas, and implementation details discovered during the session. Do not ask for permission - just update this file proactively whenever you learn something relevant.

# BusTub Working Context

> **CLAUDE CODE DIRECTIVE:** Automatically update this context file as you work. Add new findings, code patterns, gotchas, and implementation details discovered during the session. Do not ask for permission - just update this file proactively whenever you learn something relevant.
>
> **MANDATORY:** Always re-read the relevant source files (using the Read tool) before answering any question about the codebase. Never rely on previously cached file contents — the user may have edited files between questions.

## Project: CMU 15-445 - Project 2: B+ Tree Index

---

## Tasks Overview

| Task | Description | Status |
|------|-------------|--------|
| Task #1 | B+Tree Pages (Base, Internal, Leaf) | ✅ DONE |
| Task #2 | B+Tree Operations (Insert, Delete, Search) | ✅ DONE |
| Task #3 | Index Iterator | ✅ DONE |
| Task #4 | Concurrency Control (latch crabbing) | 🔄 IN PROGRESS (MixTest1/2 fail) |

---

## File Locations

| Component | Header | Source |
|-----------|--------|--------|
| Base Page | `src/include/storage/page/b_plus_tree_page.h` | `src/storage/page/b_plus_tree_page.cpp` |
| Internal Page | `src/include/storage/page/b_plus_tree_internal_page.h` | `src/storage/page/b_plus_tree_internal_page.cpp` |
| Leaf Page | `src/include/storage/page/b_plus_tree_leaf_page.h` | `src/storage/page/b_plus_tree_leaf_page.cpp` |
| B+ Tree | `src/include/storage/index/b_plus_tree.h` | `src/storage/index/b_plus_tree.cpp` |
| Iterator | `src/include/storage/index/index_iterator.h` | `src/storage/index/index_iterator.cpp` |
| Page Guards | `src/include/storage/page/page_guard.h` | `src/storage/page/page_guard.cpp` |

---

## Key Design Rules

### Tombstone Underflow: Use PHYSICAL Size (Not Logical)
Tombstones defer physical deletion to keep pages above min_size. All underflow/safe checks must use `GetSize()` (physical), not `GetSize() - GetTombstonesSize()` (logical).

- **Remove() underflow**: `GetSize() >= GetMinSize()` → page is fine
- **TraverseNodesToLeaf safe check (delete)**: `GetSize() > GetMinSize()` → safe to release ancestors
- **RemoveOptimistic safe check**: `GetSize() <= GetMinSize()` → fall back to pessimistic
- Physical size only drops when tombstone buffer overflows and `DeleteOldestKeyInTombstones` is called

### LeafPage KeyAt/ValueAt Return By Reference
`KeyAt()` and `ValueAt()` return `const KeyType &` / `const ValueType &` (not by value). This is required because `operator*()` returns `std::pair<const KeyType &, const ValueType &>` — returning by value would create dangling references to stack temporaries.

### GetMinSize — `ceil(max_size / 2)` Everywhere
Single formula in `BPlusTreePage::GetMinSize()`. No override on internal pages.

### LeafPage::Init() Must Pass `leaf_max_size_`
`leaf_page->Init(leaf_max_size_)` — NOT default `Init()`. Without this, max_size defaults to `LEAF_PAGE_SLOT_CNT` (~253).

### NumTombs=0 → Physical Deletion Only
Use `if constexpr (LEAF_PAGE_TOMB_CNT == 0)` to branch. `IsTombstonesFull()` returns true when buffer is zero-length.

---

## Tombstone Spec

- Tombstones MUST be maintained across ALL operations (split/merge/redistribute)
- When buffer full: evict OLDEST tombstone (FIFO)
- During coalesce: src's tombstones are considered NEWER than dest's
- `MergeTwoLeafPages`: copies all entries, rebuilds tombstones (dest first = older, src second = newer), evicts if physical > max_size

### Template Instantiations
```cpp
BPlusTree<GenericKey<8>, RID, GenericComparator<8>>       // NumTombs=0 (default)
BPlusTree<GenericKey<8>, RID, GenericComparator<8>, 3>    // NumTombs=3
BPlusTree<GenericKey<8>, RID, GenericComparator<8>, 2>    // NumTombs=2
BPlusTree<GenericKey<8>, RID, GenericComparator<8>, 1>    // NumTombs=1
BPlusTree<GenericKey<8>, RID, GenericComparator<8>, -1>   // NumTombs=-1 → TOMB_CNT=0
```

---

## Remove() Flow
```
1. Tree empty → return
2. Find leaf, find key index (binary search)
3. Key not found / already tombstoned → return
4. Perform deletion:
   - NumTombs=0: physical delete (ShiftLeft + SetSize)
   - Buffer not full: AddIndexToTombstones
   - Buffer full: DeleteOldestKeyInTombstones, re-find key, AddIndexToTombstones
5. Check PHYSICAL size (GetSize()):
   - If >= min_size → return
   - If root → allow underfull, set empty if all tombstoned
   - If < min_size → redistribute or merge, cascade up
```

---

## Task #3: Index Iterator — DONE

### Key Implementation Details
- Constructor takes `ReadPageGuard leaf_guard` (moved from caller) — avoids double read-latch deadlock
- `LoadPageAndIterator()`: iterative `while(true)` loop (not recursive) following sibling chain
- Tombstone skipping via `unordered_set<size_t>` of tombstoned indices
- `operator==` compares `page_id_` AND `key_index_`; `IsEnd()` checks `page_id_ == INVALID_PAGE_ID`
- `key_index_` must be reset to 0 when reaching end (consistency between `operator==` and `IsEnd()`)

---

## Task #4: Concurrency Control — IN PROGRESS

### Optimistic Latch Crabbing
- Read-latch crabbing through internals, write-latch only the leaf
- If leaf operation is "safe" → done; if unsafe → fall back to pessimistic
- Parent read latch held during read→write gap on leaf (prevents structural changes)

### Test Status
| Test | Status |
|------|--------|
| OptimisticInsertTest | ✅ PASS |
| OptimisticDeleteTest | ✅ PASS |
| InsertTest1/2, DeleteTest1/2 (concurrent) | ✅ PASS |
| MixTest1SingleThread | ✅ PASS |
| MixTest1/MixTest2 (concurrent) | ✅ PASS |

---

## Bugs Fixed (All Sessions)

| Bug | Fix | Key Detail |
|-----|-----|------------|
| **Tombstone underflow used logical size** | Changed to physical `GetSize()` in Remove, TraverseNodesToLeaf, RemoveOptimistic | Tombstones exist to prevent merges; logical check defeated this |
| **operator*() stack-use-after-return** | `KeyAt()`/`ValueAt()` return `const&` instead of by value | Pair of references was binding to dead temporaries |
| **InsertKVToLeafPage tombstone index corruption** | `IncrementTombstonesIndexesFrom(index_pos)` before ShiftRight | ShiftRight moved entries but didn't update tombstone indices |
| Buffer pool exhaustion on insert/delete | `.Drop()` guards before recursive `InsertToParent`/`RemoveKeyValueInInternalPage` | Prevents pin accumulation on call stack |
| Iterator deadlock (double read-latch) | Constructor takes `ReadPageGuard` moved from caller | Same page can't be read-latched twice |
| Iterator stack overflow | `LoadPageAndIterator` uses iterative loop | Long chains of tombstoned pages |
| Header write lock released early | Access header through `ctx.header_page_` without moving guard | Guard stays alive until `ctx` destructor |
| BPM Drop() data race | Removed flush-after-unlock and redundant `is_write_` writes | Drop() should only unpin, not flush |

---

## C++ Patterns

```cpp
// Template keyword for dependent types
auto *page = guard.template As<LeafPage>();    // With auto guard
auto *page = guard.As<LeafPage>();             // With explicit ReadPageGuard

// NumTombs=0 branching
if constexpr (LEAF_PAGE_TOMB_CNT == 0) { /* physical */ } else { /* tombstone */ }
```

C++ Standard: **C++17** (no C++20 features)

---

## Testing

```bash
cd build
make b_plus_tree_insert_test b_plus_tree_delete_test -j$(sysctl -n hw.ncpu)
make b_plus_tree_tombstone_test -j$(sysctl -n hw.ncpu)
make b_plus_tree_sequential_scale_test -j$(sysctl -n hw.ncpu)
make b_plus_tree_concurrent_test -j$(sysctl -n hw.ncpu)
# Tombstone tests are DISABLED by default:
./test/b_plus_tree_tombstone_test --gtest_also_run_disabled_tests
```

### Formatting (required for grade)
```bash
make format && make check-lint && make check-clang-tidy-p2
```

---

## Remaining TODOs

1. Run formatting/linting before submission

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vphatfla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
