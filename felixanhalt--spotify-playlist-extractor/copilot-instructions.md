## spotify-playlist-extractor

> - **Frontend dev**: `cd frontend && pnpm run dev` | **build**: `pnpm run build` | **lint**: `pnpm run lint`

# Agent Guidelines - Spotify Playlist Extractor

## Commands
- **Frontend dev**: `cd frontend && pnpm run dev` | **build**: `pnpm run build` | **lint**: `pnpm run lint`
- **Backend dev**: `cd backend && ./start.sh` or `uvicorn main:app --reload`
- **Test backend**: `cd backend && pytest` | **single test**: `pytest tests/test_auth.py::test_login_redirect`
- **Test frontend**: No pnpmscript configured. Run tests with `npx vitest` if needed.

## Code Style

### TypeScript/React
- **Imports**: (1) React, (2) third-party libs, (3) router, (4) local API/utils, (5) components, (6) context
- **Types**: Always use explicit interfaces for props/state. Union types for nullables (`string | null`). Avoid `any` except for untyped external data.
- **Naming**: PascalCase components/interfaces, camelCase functions/vars, UPPER_SNAKE_CASE constants
- **Components**: Presentational use props, containers use hooks/context. State → handlers → JSX.
- **Error handling**: `try-catch` with `err: any`, optional chaining (`?.`), user-facing alerts
- **The project uses Tailwind v4 for styling. DO NOT USE ANY v3 RELATED styling!**
- **Seldom use inline styles. Prefer Tailwind classes.**

### Python/FastAPI
- **Imports**: (1) stdlib, (2) FastAPI/third-party, (3) local relative imports
- **Types**: Use type hints on all functions. `Optional[T]` for nullables, `List[Dict[str, Any]]` for complex structures.
- **Naming**: snake_case functions/vars, UPPER_SNAKE_CASE constants, `_prefix` for private
- **Error handling**: `HTTPException` for API errors, print statements for debugging, early returns for validation
- **Endpoints**: Docstring → parse body → validate → retrieve session → business logic → JSONResponse

## Standards
- Max 20 lines per method (logic). Separate concerns. No speculative abstractions.
- Test with accessibility queries (`getByRole`, `getByLabelText`). Mock external dependencies.
- Use `async`/`await` consistently. FastAPI endpoints always `async def`.

Agents & rules:

- **For bug reports**, invoke `@bug-hunter` to investigate and fix bugs by finding root causes, not just symptoms. The bug hunter will trace the issue, identify the underlying problem, and implement a proper fix.
- **Start each feature** by invoking `@architect` to plan the implementation: identify files to change, steps to take, and potential issues.
- **After making major code changes**, automatically invoke `@code-reviewer` to review modified changes for quality, correctness, security, and best practices.
- **After code review passes**, automatically invoke `@quality-assurance` to run tests (`pnpm test`), lint (`pnpm run lint`), and build (`pnpm build`) — all must pass before the task is complete.
- **Never skip QA** — always run quality-assurance after code changes, even for small changes.
- If you need to search the codebase for patterns or understand architecture, use `@explore` for fast file/code searches.
- **For creating OpenCode skills**, invoke `@skill-contributor` to create skills or identify skill opportunities using `@explore`.
- **Never document something if not explicitly asked** — only provide documentation when requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felixAnhalt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/felixAnhalt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
