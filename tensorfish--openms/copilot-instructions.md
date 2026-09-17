## openms

> [`docs/`](docs/) is the source of truth for project guidance, contracts, findings, and validation. This file is only an index.

# Agent navigation

[`docs/`](docs/) is the source of truth for project guidance, contracts, findings, and validation. This file is only an index.

Target desktop-sized applications with a minimum viewport of **800×600**. Mobile support is not required.

## Operating rules

Never commit generated validation reports (including JSON reports and their raw runtime/test logs) to `docs/validation/` or anywhere else in the repository. Keep those artifacts outside the repository, such as `/tmp/openms-...`, or in ignored `artifacts/` storage. Commit only concise Markdown findings, reproduction commands and relevant source/catalog identities. Do not force-add ignored reports or add documentation links to untracked report files. See [validation artifact policy](docs/validation-method.md#artifact-policy).

Use explicit flags for CLI tool configuration; follow [CLI configuration](docs/coding-style.md#cli-configuration). Do not introduce environment-variable inputs for one-shot tools.

Follow [change-scoped validation](docs/validation-method.md#validation-scope): use the smallest check relevant to the edit, then stop. Comprehensive smoke/end-to-end runs are opt-in, not a completion requirement. Documentation and Compose/environment edits do not require browser, login, gameplay or persistence checks.

Apply [shorten the loop](docs/validation-method.md#shorten-the-loop) before expanding online integration: prove native input → transaction → recipient update → reconnect, in small file-disjoint batches with fixed interfaces and immediate cheap checks. Prioritize one domain-scoped executable check over more agents or speculative integration. The [iteration-loop findings](docs/validation.md#iteration-loop-investigation) distinguish measured costs, current reuse/restart constraints and tooling still to implement.

If a bottleneck can be removed by a tool, or a tool would shorten the iteration/feedback loop, build the tool instead of grinding through repeated manual work. Extend the existing tooling where it fits — [asset decoding](client/src/assets/), [extraction](client/tools/extract.js), [archive scanning](client/tools/scan.js), [development server](client/tools/dev-online.js), [online scenarios](client/tools/scenarios/), [analysis scripts](docs/tools/) — rather than starting a parallel convention. Keep every tool deterministic, bounded, and reproducible, and record what it establishes in `docs/`.

When browser verification is in scope, parallelize independent checks when useful. Give each worker an isolated browser context, account/session state, and evidence directory; never share a mutable game session. Verify the same source/catalog identity across workers and consolidate their results before delivery.

For changes requiring browser checks, use the [measured feedback loop](docs/validation-method.md#timing-the-feedback-loop), not repeated manual browser narration. Finish a file-disjoint edit batch, then run applicable formatting and cheap targeted tests/static checks before any necessary extraction; changing an extraction recipe mid-run invalidates that work. Use `client/tools/scenarios/` for affected input/readiness/geometry/audio checks, and reserve image review for appearance. Do not run all scenarios or serial-versus-concurrent comparisons for routine edits. Reuse an existing validation browser when available and retain isolated contexts.

For explicitly requested performance work, profile before optimizing: retain preflight/extraction, server identity stages, browser acquisition, readiness, action and teardown timings. Compare serial and `--concurrency 2` only when evaluating concurrency; do not run competing performance probes concurrently or sum hierarchical timings. Full conversion remains an explicit release gate. Record measurements and failures from requested performance/release work in [validation results](docs/validation.md), not for routine documentation/configuration edits.

Keep static extraction reusable across sessions. Do not delete the extraction cache or re-run extraction for browser-only edits. Use the retained measurements and generated catalog identity rather than assuming every source edit requires rebuilding assets.

## Documentation

- [Quick Start: documentation homepage and local setup](docs/index.md)
- [Client: setup, controls, code, and coverage](docs/development.md#client)
- [Server: database, settings, deployment, and authority](docs/development.md#server)
- [Required coding style for all coding agents](docs/coding-style.md)
- [Original inputs and provenance](docs/inputs.md)
- [Original file hashes](docs/input-manifest.json)
- [Asset formats and decoder evidence](docs/asset-evidence.md)
- [Original client and rendering evidence](docs/client-evidence.md)
- [Scene format and browser API contract](docs/scene-contract.md)
- [In-game inventory and priority boundaries](docs/ingame-inventory.md)
- [UI](docs/ingame-ui.md), [portals](docs/ingame-portals.md), [life](docs/ingame-life.md), and [audio/effects](docs/ingame-audiovisual.md)
- [Validation procedure](docs/validation-method.md)
- [Validation results](docs/validation.md)
- [Extraction report](docs/extraction.json) and [archive scan](docs/archive-scan.json)
- [Windows reference requests](docs/windows-reference-captures.md)
- [Asset Ghidra evidence](docs/ghidra-assets/), [client Ghidra evidence](docs/ghidra-client/), and [analysis scripts](docs/tools/)

## Implementation

- [Workspace scripts and packages](package.json)
- [Asset decoding](client/src/assets/)
- [Scene extraction](client/tools/extract.js) and [archive scanning](client/tools/scan.js)
- [Browser entry](client/src/browser/online/main.js), [online integration](client/src/online/), and [animation](client/src/rendering/animation.js)
- [Shared visual ownership](client/src/rendering/visual-resources.js) and [native audio](client/src/audio/audio-engine.js)
- [Browser markup](client/index.html) and [styles](client/style.css)
- [Development server](client/tools/dev-online.js) and [online scenarios](client/tools/scenarios/)
- [Regression tests](client/test/)
- [Authoritative server workspace](server/)

---
> Source: [tensorfish/openms](https://github.com/tensorfish/openms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
