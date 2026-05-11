## genternet-chrome

> This folder is the public open-source staging repo for the Genternet visual

# Genternet Chrome Repo

This folder is the public open-source staging repo for the Genternet visual
browser runtime. Treat the parent workspace as private context and keep changes
inside this folder unless the user explicitly asks to update the private
workspace too.

## Repo Boundary

- Runtime code lives in `extension/`.
- Runtime logic that can be tested without Chrome lives in `extension/runtime/`.
- Node tests live in `tests/`.
- Local QA fixtures live in `test-stand/`.
- Public architecture and release-track docs live in `docs/`.
- The Chrome Web Store publishing track is separate. Do not add listing copy,
  screenshots, packed release artifacts, account notes, or channel rollout
  metadata here unless they are intentionally public.

Do not copy private workspace folders into this repo. In particular, exclude
`txt2img/`, generated image runs, ranking pages, candidate data, `.env`, `.venv`,
local browser profiles, and any checked-in secrets.

## Subsystem Guides

Read the closest guide before editing a subsystem:

- `extension/AGENTS.md` for manifest, content script, options page, overlay, and
  background service worker changes.
- `extension/runtime/AGENTS.md` for prompts, schemas, provider adapters, session
  state, and OpenAI-compatible LLM behavior.
- `test-stand/AGENTS.md` for local fixture and extension-flow QA.
- `tests/AGENTS.md` for Node checks and package-boundary tests.
- `docs/AGENTS.md` for public documentation and publishing-track notes.

When a behavior changes, update the relevant subsystem guide in the same patch
so future agents do not rely on stale architecture notes.

## Runtime Rules

- Keep the shipped runtime JavaScript-only. Do not reintroduce a required local
  Python server for normal extension use.
- User keys belong only in `chrome.storage.local` through the options page or in
  developer-local `.env` files for tests. Never sync, hardcode, document, or
  fixture real keys.
- Provider calls must run through the extension background service worker, not
  page-origin content-script fetches.
- Keep provider/model settings explicit and editable. Do not silently switch
  providers when a selected provider fails.
- Preserve the disabled grounding status until a real grounding pass exists.
- Generated infographic prompts must keep the 18 px minimum text rule and favor
  dense, source-grounded cards over sparse labels.

## Checks

Run this before finishing code or prompt changes:

```sh
npm run check
```

If a change affects extension behavior, also reload the unpacked extension from
`extension/` in Chrome and exercise the visual flow on `test-stand/`.

---
> Source: [DenisSergeevitch/genternet-chrome](https://github.com/DenisSergeevitch/genternet-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
