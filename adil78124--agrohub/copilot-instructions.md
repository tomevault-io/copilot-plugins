## url-state

> This project mandates that any user-controlled state that affects page content, navigation, or data fetching must be synchronized with the URL. Use shared hooks from `src/shared/hooks` to read and write this state. This ensures shareable links, back/forward navigation, SSR compatibility, and consistent behavior across modules.


## URL state and shared hooks usage

This project mandates that any user-controlled state that affects page content, navigation, or data fetching must be synchronized with the URL. Use shared hooks from `src/shared/hooks` to read and write this state. This ensures shareable links, back/forward navigation, SSR compatibility, and consistent behavior across modules.

### Must

- Use `use-url-filter` for any filters, pagination, sorting, and view toggles that change what is displayed on the page.
- Use `use-search-query` for search inputs that affect the dataset or view.
- Reflect all such state in the URL query parameters; never keep a divergent internal copy.
- Derive component state from the URL (source of truth), and update the URL on user actions.
- Keep behavior consistent across modules; prefer shared hooks over custom ad-hoc implementations.

### Must not

- Do not keep filter/search/sort/pagination state only in component or store state without syncing to URL.
- Do not implement custom URL parsing/writing in modules when a shared hook exists.
- Do not debounce or throttle in ways that cause URL and UI to get out of sync.

### Allowed exceptions

- Purely ephemeral UI state that does not affect navigation or fetched data (e.g., input focus, open/close of local dropdowns, temporary form drafts before submit) may remain local.
- Sensitive values that must never appear in the URL (e.g., passwords, tokens) must not be synced; design such flows accordingly.

### Recommended hooks

- `src/shared/hooks/use-url-filter.ts`
- `src/shared/hooks/use-search-query.ts`

---
> Source: [Adil78124/AgroHub](https://github.com/Adil78124/AgroHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
