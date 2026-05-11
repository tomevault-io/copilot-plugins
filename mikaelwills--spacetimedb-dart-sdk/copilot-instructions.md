## spacetimedb-dart-sdk

> 1. **Multiple rows per request** (`table_cache.dart`)

# SpacetimeDB Dart SDK

## Offline Storage Implementation

### Completed Fixes

1. **Multiple rows per request** (`table_cache.dart`)
   - Changed `_optimisticChanges` from `Map<String, _OptimisticChange<T>>` to `Map<String, List<_OptimisticChange<T>>>`
   - Rollback now iterates in reverse order to properly undo changes

2. **`loadFromSerializable` clearing optimistic state** (`table_cache.dart`)
   - Now preserves optimistic changes map during load
   - Added warning log for failed deserialization

3. **Table snapshot locking** (`json_file_storage.dart`)
   - Added per-table `_AsyncLock` via `_getTableLock()`
   - `saveTableSnapshot` and `loadTableSnapshot` now synchronized

4. **Backup file cleanup** (`json_file_storage.dart`)
   - Added `_cleanupBackup()` method
   - Backups cleaned after successful read/write operations

5. **`clearAll` race condition** (`json_file_storage.dart`)
   - Now acquires `_globalLock`, `_mutationsLock`, and `_syncTimesLock` before clearing
   - Clears `_tableLocks` map after directory deletion

6. **`clearTableSnapshot` locking** (`json_file_storage.dart`)
   - Now uses table lock and cleans up backup files

7. **`failedCount` logic** (`subscription_manager.dart`)
   - Changed from `lastError != null` to `hasExceededMaxRetries()`
   - Failed count now accurately reflects permanently failed mutations

8. **Cached pending counts** (`subscription_manager.dart`)
   - Added `_cachedPendingCount` and `_cachedFailedCount`
   - `_decrementPendingCount()` and `_incrementFailedCount()` for efficient updates
   - No longer re-reads full queue for every status update

### Design Notes

- SpacetimeDB is server-authoritative with ACID transactions - conflict resolution happens at reducer level
- Optimistic UI is speculative; server's `TransactionUpdate` provides authoritative state
- SDK's job is to queue and replay mutations; server decides validity

---
> Source: [mikaelwills/spacetimedb-dart-sdk](https://github.com/mikaelwills/spacetimedb-dart-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
