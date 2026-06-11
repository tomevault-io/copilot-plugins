## ai-latex

> Monorepo with Next.js frontend and C# backend:

# Ai-LaTex - AI-Powered Collaborative LaTeX Editor

## Project Structure

Monorepo with Next.js frontend and C# backend:

```
src/web/          — Next.js 16 frontend (App Router, TypeScript, Tailwind)
src/server/       — C# Solution (.NET 10)
  ClaTeX.Api/     — ASP.NET Core Web API
  ClaTeX.Core/    — Shared models and interfaces
  ClaTeX.AI/      — OpenCode Go AI integration
  ClaTeX.TexLab/  — texlab LSP WebSocket bridge
  ClaTeX.BuildAgent/  — User-local LaTeX build agent (WS Server on localhost, front-end connects directly)
  ClaTeX.BuildServer/ — Server-side LaTeX build service (receives tasks from Api, runs compiler on server)
```

## Development Commands

### Frontend (src/web/)
```bash
npm run dev       # Start dev server (http://localhost:3000)
npm run build     # Production build
npm run lint      # ESLint
```

### Backend (src/server/)
```bash
dotnet build ClaTeX.slnx              # Build entire solution
dotnet run --project ClaTeX.Api       # Start API server
dotnet run --project ClaTeX.BuildAgent  # Start local build agent (WS server on :18384)
dotnet run --project ClaTeX.BuildServer # Start server-side build service
```

### y-sweet (collaboration server)
```bash
npx y-sweet@latest serve              # Start local y-sweet server
```

## Tech Stack

- **Frontend**: Next.js 16 + Monaco Editor + Yjs + y-sweet
- **Backend**: ASP.NET Core Web API (.NET 10)
- **AI**: OpenCode Go API via OpenAI-compatible and messages-compatible endpoints
- **Auth**: Supabase Auth
- **Database**: Supabase (PostgreSQL)
- **LSP**: texlab (server-side)
- **Collaboration**: Yjs + y-sweet

## Environment Variables

Frontend (.env.local):
- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `Y_SWEET_CONNECTION_STRING` — y-sweet server URL

Backend (appsettings.json / env):
- `OPENCODE_API_KEY` — OpenCode Go API key
- `AI__BaseUrl` — defaults to `https://opencode.ai/zen/go/v1`
- `AI__Model` — defaults to `deepseek-v4-flash`
- `AI__ContextWindowTokens` — defaults to `1000000`
- `AI__ThinkingEffort` — optional `low`, `medium`, `high`, or empty/off for no requested thinking mode
- `Supabase__Url` / `Supabase__Key`

---
> Source: [Summu77/Ai-Latex](https://github.com/Summu77/Ai-Latex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
