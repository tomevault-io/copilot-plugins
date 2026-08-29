## webview-conventions

> Structure and messaging conventions for the Nyx webview UI


# Webview conventions

- **Module boundaries**: `dom.ts` owns the HTML skeleton + element refs,
  `state.ts` owns shared state + `post()`, `transcript.ts` renders messages and
  cards, `composer.ts` owns input/queue/attachments, `main.ts` is only the
  host-message dispatcher. New UI belongs in the matching module, not main.ts.
- **All host↔webview messages are typed** in `src/types.ts`
  (`HostToWebview` / `WebviewToHost`). Add the variant there first; the
  dispatcher switches must stay exhaustive (`never` check in default).
- **Escape everything user- or model-provided** with `escapeHtml` before
  `innerHTML`; markdown goes through `renderMarkdown`/`renderMarkdownFinal`.
- **Streaming renders are throttled** (see `scheduleStreamingRender`): never
  re-parse full markdown on every delta — that froze the UI on long
  generations once already.
- **No external resources**: the CSP allows only bundled assets; libraries
  (marked, highlight.js) are bundled via esbuild into `media/main.js`.
- **Verify UI changes in the harness**: `python3 -m http.server 8321` +
  `http://127.0.0.1:8321/.harness/index.html?scene=chat|approval|machines`
  renders the real bundle with demo data (also used for README screenshots).

---
> Source: [sthamann/nyx-local-ai](https://github.com/sthamann/nyx-local-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
