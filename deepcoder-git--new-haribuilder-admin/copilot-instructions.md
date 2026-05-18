## admin-filter-search-ui-consistency

> Enforce same filter/search UI structure across modules


# Admin Filter/Search UI Consistency

Use one consistent header pattern for all module list/report pages.

- Keep search and filters in one top header row.
- Use same visual size for inputs/selects (`height: 38px` preferred).
- Keep dropdown filters aligned to the right, search on the left.
- Reuse existing classes first (`form-control-solid`, `form-select-solid`, `d-flex`, `gap-*`).
- Avoid custom one-off spacing/widths unless required by content.
- No horizontal scroll for filter row.

## Blade Structure (reference)

```blade
<div class="d-flex align-items-center justify-content-between gap-2 flex-nowrap w-100">
  <div class="d-flex align-items-center gap-2 flex-grow-1" style="min-width: 0;">
    {{-- title/badge + search --}}
  </div>
  <div class="d-flex align-items-center gap-1 flex-nowrap ms-2">
    {{-- site/store/type selects --}}
  </div>
</div>
```

## Apply Scope

- Apply to all admin module Blade UIs (especially `resources/views/admin/**/*.blade.php`).
- When updating old screens, refactor toward this structure instead of introducing new patterns.

---
> Source: [deepcoder-git/new_haribuilder_admin](https://github.com/deepcoder-git/new_haribuilder_admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
