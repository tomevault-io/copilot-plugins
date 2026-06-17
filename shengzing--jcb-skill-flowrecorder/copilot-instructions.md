## jcb-skill-flowrecorder

> Use when you need to record a website's menu navigation flow, capture screenshots, save rendered HTML, download page resources, and summarize network/API calls during a semi-automatic browser session. Best for documenting public sites, docs sites, and logged-in console menus by manually clicking through them while Playwright records the flow.


# jcb-skill-flowRecorder

Author: Chengbin Jia

Use this skill when the user wants to:

- record website menu jumps
- preserve the actual clicked navigation path
- capture screenshots and rendered HTML after each navigation
- save static resources needed to reconstruct the page
- inspect API traffic triggered by page transitions
- produce site maps, Mermaid diagrams, or interface summaries from the captured output

This skill is optimized for **semi-automatic capture**:

1. Launch the site in a visible browser
2. Let the user log in manually if needed
3. Start recording after login
4. Let the user click menus manually
5. Persist screenshots, HTML, resources, and API logs for each captured page

## Use This Skill For

- public marketing websites
- docs portals
- login-gated cloud consoles
- reverse mapping page flows to backend interfaces
- creating navigation diagrams from real browsing history

## Do Not Use This Skill For

- high-volume crawling
- stress testing
- unauthorized data collection
- bypassing authentication
- large-scale scraping without explicit permission

## Workflow

### 1. Verify prerequisites

The workspace should have:

- Node.js available
- `playwright` installed
- browser runtime installed, typically Chromium

If missing, install:

```bash
npm install -D playwright
npx playwright install chromium
```

### 2. Run the recorder

Use the bundled script:

```bash
node skills/jcb-skill-flowRecorder/scripts/record-menu-flow.mjs <url>
```

Common examples:

```bash
node skills/jcb-skill-flowRecorder/scripts/record-menu-flow.mjs https://example.com
node skills/jcb-skill-flowRecorder/scripts/record-menu-flow.mjs https://example.com --output docs/menu-flow
node skills/jcb-skill-flowRecorder/scripts/record-menu-flow.mjs https://example.com --include-sensitive
```

### 3. Operate the capture session

- Log in manually if needed
- Return to terminal and press Enter to begin recording
- Click menus in the browser manually
- Stop with `Ctrl+C`

### 4. Review outputs

The output directory contains:

- `navigation-log.json`
- `navigation-map.csv`
- `navigation-flow.md`
- `captures/<capture_id>/screenshot.png`
- `captures/<capture_id>/page.html`
- `captures/<capture_id>/network.json`
- `captures/<capture_id>/api-calls.json`
- `captures/<capture_id>/resources/`

See [references/output-format.md](references/output-format.md) when you need the output schema.

### 5. Summarize the capture

When the user asks for analysis:

- read `navigation-log.json` first
- deduplicate edges by `(source_url, clicked_text, target_url)` when drawing a clean flow chart
- separate business APIs from analytics / telemetry
- prefer grouping interfaces by backend service boundary, such as `iam-server`, `walletd-server`, `biz-server`, `api-server`, `panel-server`
- clearly state that the result only covers pages and APIs actually visited during the capture session

## Safety And Privacy

- Default mode should keep sensitive values redacted
- Only use `--include-sensitive` when the user explicitly wants full local capture
- Warn users not to publish raw captures containing tokens, personal information, or private business data
- Use only on systems and sites where the user is authorized to record traffic

## Notes

- This recorder works best on modern web apps where DOM clicks and route changes are observable
- Some SPA transitions may still need follow-up review if the route change is partial or highly dynamic
- Some endpoints may return text or framework transport payloads rather than clean JSON; summarize them accordingly rather than forcing a JSON interpretation

---
> Source: [shengzing/jcb-skill-flowRecorder](https://github.com/shengzing/jcb-skill-flowRecorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
