## cryptex-oss

> This file is read by AI coding assistants (Claude, Copilot, Cursor, etc.) so they follow the project's conventions. Humans contributing PRs should skim it once too.

# Contributor + AI-Assistant Guide

This file is read by AI coding assistants (Claude, Copilot, Cursor, etc.) so they follow the project's conventions. Humans contributing PRs should skim it once too.

## What this repo is

Cryptex OSS is a SvelteKit 2 + Svelte 5 + shadcn-svelte static-site app for LLM red-teaming techniques. The deliverable is a single-page browser app, served as static files. There is no backend, no database, and no auth; every piece of state lives in `localStorage`. The app exposes 25 tool routes (10 technique workbenches under `/transforms`, `/decode`, etc., plus 15 red-team labs under `/redteam/*`).

A Python CLI (`cryptex-cli`, managed with `uv`) shells out to Node to execute the canonical transformers in `src/transformers/`, so there is one source of truth for transforms: both the SvelteKit app and the CLI import the same 159 transformer files.

The v2.0 milestone hardened every tool surface with a unified `ToolShell` template, typed `CryptexError` taxonomy, Web Worker offloading for inputs at or above 50 KB, a per-tool Vault drawer (309 bundled OSS-licensed seed items + custom-add), and a persistent searchable session history via the v2 store. See the v2.0 conventions section below before adding new surfaces.

## Commands

### Primary (SvelteKit app)

```bash
cd app && npm install              # install deps
cd app && npm run dev              # Vite dev server, http://localhost:5173
cd app && npm run check            # svelte-kit sync + svelte-check (type-check)
cd app && npx vitest run           # run all unit tests
cd app && npx vitest run path/to/file.test.ts   # run a single test
cd app && npm run build            # static build → app/build/
```

### Python CLI (cryptex-cli)

```bash
uv run cryptex-cli list
uv run cryptex-cli inspect caesar --json
uv run cryptex-cli encode --transform base64 --text "Hello"
uv run cryptex-cli /base64 --decode SGVsbG8=          # slash-command form
uv run cryptex-cli auto-decode --text "SGVsbG8="
pytest                                                 # python_tests/
```

### Docker

```bash
docker compose up --build          # → http://localhost:8080
```

The image is `nginx:1.27-alpine` serving `app/build/`. Same `app/build/` as the static deploy path.

## Architecture

### Transformers (the 159 transforms — single source of truth)

- Live in `src/transformers/<category>/<name>.js` — **category = directory name** (ancient, case, cipher, encoding, fantasy, format, special, technical, unicode, visual).
- Each file does `export default new BaseTransformer({...})` from `src/transformers/BaseTransformer.js`.
- **Auto-discovered.** Adding a new file is sufficient; the registry picks it up.
- `src/transformers/loader-node.js` is the **Node-side** loader used by `scripts/cli_bridge.js` (the CLI's subprocess entry point). The web app uses Vite `import.meta.glob` instead. Same modules, two front-doors.
- `priority` (1–310) controls the universal decoder's ranking when auto-detecting format. See the priority guide at the bottom of `BaseTransformer.js`. Unique character sets (Binary, Morse, Braille) sit at 300; Unicode lookalikes default to 85; ciphers at 60; invisible-text at 1.

### Multi-provider AI gateway (BYOK)

All AI calls route through `app/src/lib/ai/gateway.ts`. It exposes:

- `chat(req) → ChatResponse`
- `streamChat(req)` — async iterator of stream chunks
- `fetchModels(signal)` — aggregates catalogs across every enabled provider
- `validateKey(providerId, candidate, opts)` — per-provider key probe
- `resolve(modelId)` — routes qualified ids (`openrouter:…`, `anthropic:…`, `openai-compat:<instance>/…`) to the right adapter; unqualified ids default to OpenRouter.

Provider config lives in `app/src/lib/ai/providers.svelte.ts` and is persisted under `cryptex.providers` in `localStorage`. Adapters in `app/src/lib/ai/adapters/` are lazy-imported.

Supported providers:
- **OpenRouter** (default, CORS-open)
- **Anthropic direct** (uses `anthropic-dangerous-direct-browser-access` header)
- **OpenAI-compatible endpoints** (Groq, Together, Fireworks, DeepInfra, Cerebras, SambaNova, custom)

Direct OpenAI / Google Gemini are **not supported** from the browser — no CORS. Users route those models through OpenRouter.

In dev mode (`npm run dev`), `app/vite.config.ts` defines a `/api/_proxy/<providerId>` proxy that forwards model-list and chat-completions requests server-side to each provider — sidestepping browser CORS on `/v1/models`. The dev-vs-prod URL resolution lives in `app/src/lib/ai/proxy-url.ts` (`effectiveBaseURL` / `effectiveDirectBaseURL`) and is consumed by the three adapters. Production static deploys do not include the proxy; direct fetches go to provider URLs and the per-preset `defaultModels` lists in `presets.ts` cover any `/models` endpoints that block CORS.

### Technique registry

`app/src/lib/techniques/registry.ts` aggregates everything available to the workbenches:

- 159 transformers (re-exported from the `src/transformers/` registry)
- **36 mutators** in `from-mutators.ts` — single-prompt rewriters (rephrase, obfuscate, roleplay, multilingual, fragment, custom, red_team_persona, step_back, chain_of_verification, ctf_framing, rfc_style, payload_split, hypothetical_world, cipher_encode_bypass, pap_logical, pap_authority, many_shot, tap_seeder, plus E1-E5 expansion entries)
- **8 classifier-evasion techniques** in `from-classifier.ts` (circumlocution, metonymy, semantic_decomposition, technical_register, academic_framing, temporal_displacement, perplexity_raise, plus one expansion entry)
- **3 conversation modes** in `modes/` (creative, intelligent, adaptive)
- **4 composites** in `from-composites.ts` (base64_smuggle, grammar_constrained_output, layered_mutation, multi_layer_attack)

### Tool state

- Per-tool form state and outputs are persisted via `app/src/lib/tools/repo.ts`, which writes to `localStorage` under the `cryptex.toolStates` key.
- All access is browser-guarded — there is no SSR for stateful pages.
- No tool component reads `localStorage` directly; everything goes through `repo.ts`.

### Routes

- File-based SvelteKit routing under `app/src/routes/`.
- 25 tool routes: 10 technique routes (`/transforms`, `/decode`, `/emoji`, `/gibberish`, `/tokenizer`, `/tokenade`, `/bijection`, `/fuzzer`, `/promptcraft`, `/anticlassifier`) and 15 redteam routes (`/redteam/adv-suffix`, `/redteam/glitch-tokens`, `/redteam/ocr-injection`, `/redteam/markdown-exfil`, `/redteam/probe-lab`, `/redteam/cross-model-diff`, `/redteam/replayer`, `/redteam/tool-result-lab`, `/redteam/indirect-injection`, `/redteam/harmbench`, `/redteam/strongreject`, `/redteam/jbb`, `/redteam/fingerprinter`, `/redteam/watermark`, `/redteam/pdf-injection`).
- Plus `/`, `/about`, `/guide/*`, `/privacy`, `/terms`, `/settings`.

### Two-layer access to transforms (web vs CLI)

- **Web**: SvelteKit app loads transformers via `app/src/lib/transformers/registry.ts` (Vite `import.meta.glob` over `src/transformers/`).
- **CLI**: Python (`cryptex_cli/bridge.py`) spawns `node scripts/cli_bridge.js`, which `require`s `src/transformers/loader-node.js` and exchanges JSON over stdio. This is why changes to a transform are instantly reflected in the CLI — no separate build step beyond the regular `npm run build`.

## Generated / gitignored paths — do not edit

- `app/build/` — SvelteKit static output (the deployable artifact).
- `app/.svelte-kit/` — Vite/SvelteKit cache.

If you see stale state after editing transforms, re-run `npm run build` from the repo root before debugging.

## Deployment

Static site, deploys anywhere. The repo ships a `Dockerfile` + `nginx.conf` for container deploys (Dokploy-friendly out of the box). Build runs `cd app && npm run build`; image is `nginx:1.27-alpine` serving the static output.

Build args plumbed through `Dockerfile` + `docker-compose.yml`:

- `BASE_PATH` — subpath like `/cryptex`. Empty for root-domain deploys.

That is the only build-time env.

## v2.0 conventions (applies to every new tool surface)

### `ToolShell` template — every tool wraps in it

`app/src/lib/components/shell/ToolShell.svelte` is the keystone. Every route under `/transforms`, `/decode`, `/emoji`, `/gibberish`, `/tokenizer`, `/tokenade`, `/bijection`, `/fuzzer`, `/promptcraft`, `/anticlassifier`, and every `/redteam/*` lab passes its main content as the `children` snippet:

```svelte
<ToolShell
  toolId="my-tool"
  title="My Tool"
  accent="tool"
  description="One-line tool summary."
  usage={{ title: 'My Tool · Usage', bullets: ['…', '…'] }}
>
  <!-- main content -->
  {#snippet vault()}
    <VaultSection toolId="my-tool" …>…</VaultSection>
  {/snippet}
</ToolShell>
```

ToolShell auto-renders the `<svelte:head><title>`, the `<h1>` with optional accent word, the status badge (Ready/Running/Done/Error/Cancelled) with built-in Cancel, the optional Vault slot, and the `HistoryFooter`. It also surfaces any current `CryptexError` from `activeRuns` via `ErrorPanel` above the children.

### Vault drawers — `cryptex.vault.<toolId>` + bundled seeds

Tools with curated payload libraries get a Vault drawer in their `vault` snippet:

- Bundled seeds live under `app/src/lib/vault/seeds/<toolId>.json` (loaded via Vite `import.meta.glob`, eager).
- Each item carries `schemaVersion: 1`, `source: 'bundled' | 'user'`, plus a typed `payload` field.
- User customs persist under `cryptex.vault.<toolId>` in `localStorage`.
- License posture is hard-locked: **MIT / CC0 / CC-BY-4.0 / Apache 2.0 only — NO GPL, AGPL, CC-BY-SA, or NC**. Every new seed file gets a provenance row added to `app/src/lib/vault/LICENSES.md`.

### Typed errors — `CryptexError` + `errorLogger.report()`

`app/src/lib/errors/types.ts` exports a discriminated `CryptexError` union and `Errors.*` factories (`Errors.network`, `Errors.badInput`, `Errors.rateLimit`, `Errors.storageQuota`, `Errors.tool`, `Errors.worker`, etc.). **No silent catches**:

```ts
try { … } catch (err) {
  errorLogger.report(isCryptexError(err) ? err : Errors.tool(err));
  activeRuns.fail(TOOL_ID, msg);  // ToolShell will surface via ErrorPanel
}
```

### Web Worker offload — `runInWorker` for ≥50 KB

Any transformer call where input may exceed 50 KB goes through `runInWorker`:

```ts
import { runInWorker } from '$lib/workers/runInWorker';
const out = await runInWorker({
  task: 'encode',                      // or 'decode'
  transformerName: 'base64',
  args: { text, options: {} },
  signal: controller.signal
});
```

`runInWorker` dispatches in-thread under 50 KB, uses a worker pool of 4 between 50 KB and 1 MB, and throws `Errors.badInput` over 1 MB. Cancellation via `AbortController.signal` → `worker.terminate()`.

### Heuristic benchmark caveat — yellow banner is mandatory

Every benchmark page (HarmBench, StrongREJECT, JBB, Fingerprinter, Watermark, Anti-Classifier) renders this banner above results:

> ⚠️ Heuristic scoring — not paper-accurate

Copy the exact markup from `app/src/routes/redteam/harmbench/+page.svelte`.

### Session history v2 — `history.record()` (sessionLog auto-funnels)

Legacy `sessionLog.record({...})` calls automatically fan out to `history.record({...})` via the compat shim in `app/src/lib/stores/sessionLog.svelte.ts`. New tools can call `history.record()` directly:

```ts
import { history } from '$lib/history/store.svelte';
await history.record({
  toolId: TOOL_ID,
  startedAt: t0,
  status: 'done',
  input, output,
  params: { … }
});
```

The store uses a hybrid layout (localStorage index + IndexedDB payloads, with a 50-entry localStorage fallback) and auto-prunes at 4 MB with a one-shot quota toast.

## When adding things

- **New transformer**: create `src/transformers/<category>/<name>.js`, run `npm run build`. Pick `priority` using the guide in `BaseTransformer.js`. The SvelteKit registry auto-discovers it; no manifest update needed.
- **New tool surface**: create the Svelte component in `app/src/lib/components/tools/<tool>/` (or `app/src/lib/components/redteam/<tool>/`), add the route at `app/src/routes/<tool>/+page.svelte` wrapping content in `<ToolShell toolId="…" title="…">`, and register the tab in `app/src/lib/components/shell/TabRail.svelte`. Add a Vault drawer if the tool has a curated payload library; mandatory 1 MB input cap via `Errors.badInput`; route any error through `errorLogger.report()`.
- **New mutator/classifier/composite**: add an entry to `app/src/lib/techniques/from-mutators.ts` (or the relevant sibling). It will surface in PromptCraft and the technique registry automatically.
- **New benchmark or scoring module**: heuristic only (no paper-accurate trained classifiers shipped). Render the yellow caveat banner. Use the LLM-judge prompt + JSON-tolerant parser + regex fallback pattern from `app/src/lib/redteam/strongreject-scorer.ts`.
- **New seed bundle**: add `app/src/lib/vault/seeds/<toolId>.json`, register provenance + license in `app/src/lib/vault/LICENSES.md`, bump the total count footer line.

---
> Source: [m4xx101/cryptex-oss](https://github.com/m4xx101/cryptex-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
