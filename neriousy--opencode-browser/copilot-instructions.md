## opencode-browser

> - Read `CONTEXT.md` before structural work. Use its domain language: Browser Plugin, Browser Generation, Managed Browser

# OpenCode Browser Agent Guide

## Engineering

- Read `CONTEXT.md` before structural work. Use its domain language: Browser Plugin, Browser Generation, Managed Browser
  Runtime, Shared Browser Page, Browser Session, Browser Control, MCP Registry Lease, Host Chrome, Browser Surface, and
  Presentation.
- Keep the package specific to the OpenCode V2 TUI. The public TUI plugin boundary remains Promise-based even when
  Effect owns the internal lifecycle.
- Organize runtime code by named domains. Follow the OpenCode V2 shape where it improves clarity: a domain namespace,
  an exported `Interface`, a `Context.Service`, a `layer`, and named `Effect.fn("Domain.operation")` operations.
- Bind yielded services to named variables before using them. Do not hide dependency reads inside nested `yield*`
  expressions.
- Keep synchronous parsing, option construction, state projection, and wire encoding synchronous. Use Effect for
  dependency composition, scoped ownership, interruption, typed failures, and decoding untrusted boundaries.
- Do not introduce Effect into Solid components or Presentation, frame, keyboard, mouse, or scroll hot paths.
- Add a domain module when it owns policy or a lifecycle. Do not create generic `utils`, `helpers`, or `common` modules.
- Reuse OpenCode's domain vocabulary and scoped-service pattern, but do not copy its `LayerNode`,
  `LocationServiceMap`, durable service graph, or monorepo/Turbo structure into this focused package.

## Runtime Model

- Keep one runtime architecture: one background Electron Managed Browser Runtime per Browser Generation, one or more
  Shared Browser Pages in that runtime, and one active Presentation for the selected visible page.
- Do not add system-Chrome discovery, managed Chrome downloads or caches, Chrome launch arguments, CDP screencast
  presentation, popup adoption, or alternate browser backends.
- Electron is an installed package dependency. Chrome DevTools MCP is also a direct installed dependency; resolve its
  package entry and launch it with Node.js. Do not add `npx`, first-run downloads, or an npm command-cache dependency.
- Keep Chrome DevTools MCP out of the pixel path. It connects to the generation's exact loopback Electron DevTools
  endpoint and operates the same Shared Browser Pages that Host Chrome presents.
- Share the generation runtime across every Browser Session tab. Never launch one Electron process per tab or Browser
  Surface. Resource cost scales per runtime, per page, and per active Presentation in different ways.
- Treat `frameRate` and viewport area as the primary Presentation cost controls. Hiding a Browser Surface must release
  its Presentation while leaving the Shared Browser Page alive.

## Ownership

- `Browser Plugin` owns one root scope for a plugin installation.
- `Browser Control` and the `MCP Registry Lease` live for the root scope. Their externally visible names and control
  endpoint stay stable across Managed Browser Runtime recovery.
- `Browser Generation` owns one replaceable child scope: the Electron Managed Browser Runtime, its initial Shared
  Browser Page, Browser Session, page and disconnect listeners, and the OpenCode app slot. Detach and close a
  disconnected generation before recovery. Build every replacement in a fresh child scope, publish it only after it is
  usable, and close a failed candidate.
- `Browser Session` owns logical browser tabs created through explicit session/control lifecycle. A `tabId` is stable
  identity; a CDP `targetId` is replaceable transport identity. Never expose `targetId` as the UI tab identity, and do
  not treat Chrome DevTools MCP's numeric `pageId` as either one.
- `Host Chrome` owns the OpenCode-integrated tab title, navigation controls, and browser tab strip. It owns neither
  Electron nor Presentation resources.
- `Browser Surface` owns only the visual mount. `Presentation` owns only the renderable view's start, focus, and close
  lifecycle; neither owns a Shared Browser Page or Browser Session.

Prefer `Effect.acquireRelease`, `Effect.addFinalizer`, child `Scope`s, and scoped forks over parallel booleans, stored
unsubscribe callbacks, or hand-written cleanup chains. Make cleanup idempotent and order finalizers so the app slot,
listeners, session, pages, and Managed Browser Runtime close before the root-scoped MCP/control resources. Bound
retries, timeouts, queues, and all input-driven work.

## Boundaries And Failures

- Decode plugin options, control requests/responses, and OpenCode MCP responses exactly once at their ingress with
  Effect Schema. Pass typed values inside the package. Do not substitute `isRecord`, `typeof value === "object"` plus
  property probing, or unchecked response casts for a domain schema.
- Use domain-tagged errors at Effect boundaries and preserve the original cause. Translate to rejected Promises only
  in explicit adapters such as the exported TUI boundary and the control handler boundary.
- Recovery must be serialized and generation-aware. A stale disconnect or late completion may not replace the current
  generation. Disposal always wins recovery and must close late acquisitions.
- Registration is transactional: do not publish a generation or MCP configuration until its required resources are
  ready. On partial failure, close the candidate scope. Keep a still-usable current generation when possible; after a
  disconnect, report the browser as unavailable until a candidate succeeds.
- Keep the control server stable and route each request to the current generation at execution time. Never retain a
  poisoned Shared Browser Page or session handler after a generation swap.
- Browser Control owns `new_tab` and `close_tab` lifecycle for agents. Chrome DevTools MCP owns inspection and page
  operations, but must not bypass Browser Session lifecycle and leave Host Chrome stale.

## Tooling And Verification

- Use `$opencode-browser-architecture` from `.opencode/skills/opencode-browser-architecture/SKILL.md` for changes or
  reviews in runtime source and tests.
- Use Bun for dependency management and scripts: `bun install`, `bun run <script>`, and `bun test`.
- Use `oxfmt` as the formatting source of truth (`semi: false`, `printWidth: 120`). Avoid unrelated formatting churn.
- For a bug fix, first add a focused regression test of observable behavior. Test ownership failures: interrupted
  acquisition, partial startup, stale disconnect, bounded recovery, late completion after dispose, cleanup order, and
  repeated disposal.
- Prefer injected domain dependencies and real scoped services over global mocks. Assert that listeners, slots,
  Presentations, Shared Browser Pages, Electron runtimes, control servers, and MCP leases are released.
- Run the narrowest test first, then `bun run typecheck`, `bun test`, `bun run build`, and `bun run test:packed` as
  appropriate. Run `bun run test:mcp` for the real Electron/Chrome DevTools MCP integration. Run `bun run check` before
  handing off a release-sized change.

---
> Source: [neriousy/opencode-browser](https://github.com/neriousy/opencode-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
