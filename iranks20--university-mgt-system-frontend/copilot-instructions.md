## university-mgt-system-frontend

> Frontend API integration — services, client, types (KCU)


# Frontend API Integration (KCU)

- **No Comments**: Do not write comments in code. Code should be self-documenting through clear naming and structure.
- **Single API client**: Use one place (e.g. `src/lib/api.ts` or `src/services/api.client.ts`) for all backend calls. Base URL from `import.meta.env.VITE_API_BASE_URL`. Attach JWT from auth context to requests. Handle 401 (e.g. clear auth and redirect to login) and parse error body for user feedback.
- **Services**: Keep existing service layer. Replace mock data with API client calls (e.g. `api.get('/api/qa/lecture-records')`, `api.post('/api/qa/lecture-records', body)`). Preserve existing return types from `src/types/` (qa, student, etc.) so screens and components stay unchanged.
- **Types**: Align request/response shapes with backend DTOs. Reuse or mirror types in `src/types/`; do not duplicate with different names. Use the same domain types for Excel/CSV export where applicable.
- **Errors**: Surface API errors to the user (toast or inline message). Do not log tokens or sensitive data. Use async/await and try/catch in services; return or throw consistently so callers can handle loading and error state.
- **Auth**: After login, store token (e.g. from `AuthContext`); send it on every request. If backend is unavailable, show a clear message and do not assume data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Iranks20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
