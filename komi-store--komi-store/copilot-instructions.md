## komi-store

> Local "recently viewed" history — every repo whose Details screen was opened. **Presentation-only**, backed by `SeenReposRepository` (same pipeline that drives "Hide seen" filter on Home/Search).

# Recently Viewed Feature

Local "recently viewed" history — every repo whose Details screen was opened. **Presentation-only**, backed by `SeenReposRepository` (same pipeline that drives "Hide seen" filter on Home/Search).

## Structure

```
feature/recently-viewed/presentation/
├── RecentlyViewedViewModel / State / Action / Root
├── model/RecentlyViewedRepo, mappers/RecentlyViewedRepoMapper
└── components/RecentlyViewedItem
```

## Navigation

`GithubStoreGraph.RecentlyViewedScreen`.

## Notes

- Visited timestamps come from `seenRepoDao.insert(...)` calls in `DetailsViewModel` on screen open.
- No network. `seenReposRepository.clearAll()` wipes table; `removeFromHistory(repoId)` for single-row.
- Koin in `composeApp/.../app/di/ViewModelsModule.kt` (no `data/di/`).

---
> Source: [komi-store/komi-store](https://github.com/komi-store/komi-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
