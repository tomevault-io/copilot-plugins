## localsite-ai

> Generates a Svelte Playground link with the provided code.

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available Svelte MCP Tools:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---

## Project Overview

LocalSite-AI generates complete, self-contained HTML/CSS/JS web pages from
natural language prompts using AI. Users pick a provider and model, describe
what they want, and get a live preview with an editable Monaco code editor.

**Tech stack:** SvelteKit 2, Svelte 5 (runes), Deno 2, Tailwind CSS 3,
Vercel AI SDK v5, Monaco Editor, Paneforge, svelte-sonner, @lucide/svelte.

**Runtime:** Deno 2 (`deno task dev/build/check/lint`). `package.json` retains
npm scripts for compatibility but Deno is the primary toolchain.

## Architecture

```
src/
  routes/
    +page.svelte                  Main page: WelcomeView -> GenerationView
    +layout.svelte                Global layout, Toaster
    +layout.ts                    SSR disabled (ssr = false)
    +error.svelte                 Error boundary
    api/
      generate-code/+server.ts    POST: streams NDJSON (text + reasoning)
      get-models/+server.ts       GET: models for provider; POST: list providers
      get-default-provider/+server.ts  GET: default provider from env
  lib/
    state/
      code-generation.svelte.ts   CodeGeneration class (rune-based state)
    server/providers/
      config.ts                   LLMProvider enum, config registry, env helpers
      provider.ts                 9 provider client classes, generateCodeStream()
      prompts.ts                  DEFAULT_SYSTEM_PROMPT, THINKING_SYSTEM_PROMPT
    components/
      [see README for full component list]
    ui/                           8 hand-rolled primitives (Button, Select, etc.)
```

### Data Flow

1. WelcomeView loads providers (POST /api/get-models) and models (GET
   /api/get-models?provider=X). Models cached in sessionStorage.
2. User clicks GENERATE -> CodeGeneration.generateCode() POSTs to
   /api/generate-code.
3. Server streams NDJSON: `{type: "text", content}` and
   `{type: "reasoning", content}` lines.
4. Client accumulates chunks reactively. GenerationView debounces preview
   updates (1s throttle during streaming).
5. PreviewPanel uses double-buffered iframes with z-index/opacity crossfade.

### Key Patterns

- **Svelte 5 runes everywhere.** No stores. State is `$state()`, computed
  is `$derived()`, side effects are `$effect()`, props are `$props()` with
  `$bindable()` for two-way binding.
- **Server/client boundary:** `$lib/server/` files use `$env/dynamic/private`
  for env vars. Never import these from client code.
- **SSR is disabled** (`+layout.ts`). The app renders entirely client-side
  because Monaco, sessionStorage, and streaming fetch require browser APIs.
- **NDJSON streaming.** The generate-code endpoint returns
  `application/x-ndjson`. Each line is a JSON object. The client reads via
  `ReadableStream` reader.

## Development Commands

```bash
deno task dev        # Start dev server (localhost:5173)
deno task build      # Production build to build/
deno task check      # TypeScript + Svelte type checking
deno task lint       # Deno linter
deno task start      # Run production server (build/index.js)
```

## Important Gotchas

These are non-obvious behaviors discovered during development. Read before
making changes in these areas.

### Monaco Editor

- **`executeEdits()` does NOT work in read-only mode.** Monaco silently
  returns `false`. During streaming the editor is always read-only. Use
  `setValue()` for external sync. See `CodeEditor.svelte` `syncEditorContent()`.
- **`setValue()` resets the undo stack.** This is acceptable during streaming
  (read-only) but be careful in edit mode.
- **Monaco loads asynchronously** via dynamic import. Code that arrives during
  loading must be picked up after init — see the `syncEditorContent(code)` call
  at the end of `onMount`.

### Preview Panel

- **Double-buffered iframes** swap via `setTimeout(400ms)` + `onload`. Both
  timeouts MUST be cleaned up in `onDestroy` to avoid leaks.
- **`allow-same-origin` in sandbox is required.** The parent reads
  `contentWindow.scrollX/Y` for scroll sync between the two iframes.
  Removing it breaks the seamless preview swap.
- **Fast-forward CSS** (`animation-duration: 0.001s`) is injected during
  streaming to prevent entrance animations from trapping elements invisible.
  Removed when generation completes.

### Provider System

- **All 9 providers** must implement `LLMProviderClient` (getModels + getModel).
- **Cloud provider `*_API_BASE` env vars** are passed as `baseURL` to the SDK.
  If you add a new provider, pass `getProviderBaseUrl()` to the factory.
- **`isProviderConfigured()`** checks DISABLED_PROVIDERS and required env vars.
  Always call it before allowing generation.
- **Ollama uses `createOllama()`** (not the bare `ollama()`) so that
  `baseURL` is respected for generation, not just model listing.
- **Reasoning middleware** uses `tagName: "think"` (no angle brackets). The
  thinking system prompt must use matching `<think>` tags.

### State Management

- **`CodeGeneration` class** holds all generation state as `$state()` fields.
  It has `abort()` and `reset()` methods for cancellation and restart.
- **AbortController** prevents mixed output from parallel generations. Always
  abort the previous request before starting a new one.
- **`stripFences()`** removes markdown code fences. Run it once at the END of
  streaming, not on every chunk (incomplete fences corrupt the output mid-stream).

### Tailwind / Styling

- **Dark mode only.** `class="dark"` on `<html>` in `app.html`. No toggle yet.
- **Fonts:** Inter (body) and Space Mono (headings) loaded in `app.html`.
  Do not add additional font imports.
- **CSS variables** defined in `app.css` (`--background`, `--foreground`, etc.)
  and consumed by Tailwind via `hsl(var(--...))` in `tailwind.config.ts`.

### Docker

- **Dockerfile uses `COPY`** (not `git clone`). Local builds use local code.
- **Container runs as non-root** (`appuser`). Add files before the `USER`
  directive.
- **`BODY_SIZE_LIMIT`** is set via environment variable (100K default), not
  in `svelte.config.js` (SvelteKit 2 does not support `kit.bodySizeLimit`).
- **`host.docker.internal:host-gateway`** in `docker-compose.yml` enables
  access to Ollama/LM Studio on the host machine.

---
> Source: [weise25/LocalSite-ai](https://github.com/weise25/LocalSite-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
