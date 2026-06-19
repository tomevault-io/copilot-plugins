## attach-to-browser-skill

> Use when automating the user's already-logged-in Chrome or Edge session — reusing their cookies, SSO, or 2FA — instead of a fresh browser. Primary tool is dev-browser (`dev-browser --connect`); Playwright CLI (`playwright-cli attach`) is the fallback. Triggers on `dev-browser`, `playwright-cli`, CDP attach, or "attach to my open/logged-in browser".


# Attaching to a Browser

Use this skill when browser automation must run inside the user's real, already-logged-in
browser session — reusing their cookies, SSO, and 2FA — rather than a fresh automated browser.

Two tools attach to a browser the user already controls. They each run their OWN background daemon
and open their OWN CDP connection, so pick ONE per task — you cannot connect with one tool and drive
with the other.

| Tool | Use it for | Operating model |
|---|---|---|
| **dev-browser** (`dev-browser --connect`) | **Primary.** The most stable connect to the logged-in DEFAULT profile — it auto-discovers that profile's debug endpoint itself, the step that most often breaks by hand. | One-shot JS scripts (Playwright Page API) piped via heredoc. |
| **playwright-cli** (`playwright-cli attach`) | Fallback. A stateful named session driven by discrete subcommands; also ships a dedicated-debug-profile launcher. | `attach` once, then `tab-list` / `snapshot` / `click` … |

**Why dev-browser is the more stable connection.** Attaching to the logged-in DEFAULT profile means
resolving that profile's `DevToolsActivePort` (line 1 = a RANDOM high port, line 2 =
`/devtools/browser/<GUID>`) and connecting to that exact GUID URL. Doing this by hand is the most
failure-prone part of the playwright-cli path, and channel attach (`--cdp=chrome`) resolves only a
bare path that modern Chrome rejects. dev-browser reads `DevToolsActivePort` and builds the GUID URL
itself (Chrome, Canary, Chromium, Brave), falling back to scanning ports 9222–9229 — so the fragile
manual step disappears.

The safety rules at the bottom apply to BOTH tools.

---

## Primary: dev-browser --connect

### Install

```bash
npm install -g dev-browser
dev-browser install   # one-time: downloads Playwright + Chromium
```

The binary is `dev-browser`. Scripts run in a sandboxed QuickJS runtime (NOT Node.js): no
`require`/`import`, `fs`, `process`, or `fetch`. A background daemon starts automatically and keeps
named pages and CDP connections alive across script runs.

### Enable remote debugging (user action — do not bypass)

In the target Chrome/Edge, open `chrome://inspect/#remote-debugging` and enable **Allow remote
debugging for this browser instance**; keep the logged-in browser open. This is a human step every
time — ask for it, never script it. (Alternatively the user launches Chrome with
`--remote-debugging-port=9222`.) If `--connect` reports it cannot auto-discover Chrome, the
browser-side setup in the playwright-cli "Troubleshooting" section still governs — on Chrome 136+ the
default profile may also need a `--remote-allow-origins=*` relaunch.

### Connect and verify the login

`--connect` with no URL auto-discovers the running Chrome. List the tabs FIRST and confirm a
logged-in signal before any real work:

```bash
dev-browser --connect <<'EOF'
const tabs = await browser.listPages();
console.log(JSON.stringify(tabs, null, 2));
EOF
```

Then attach a page and snapshot it. A visible "로그인 / Sign in / 회원가입" button or a fresh
new-tab page means you hit the WRONG (empty) profile — stop and have the user confirm the logged-in
window is the one with remote debugging on. Only proceed once a logged-in signal (account menu, the
user's name, a my-page URL) is present.

```bash
dev-browser --connect <<'EOF'
// Attach to an existing tab by its id from listPages(), or open a named page.
const page = await browser.getPage("work");
await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
const snap = await page.snapshotForAI();   // AI-friendly accessibility snapshot
console.log(snap.full);
EOF
```

### Operate the page

Pages are full [Playwright Page objects](https://playwright.dev/docs/api/class-page) — `goto`,
`click`, `fill`, `locator`, `evaluate`, `screenshot`. Three ways to inspect and act:

- `page.snapshotForAI({ depth, timeout })` → `{ full, incremental? }` — accessibility tree for
  picking targets. This is the "look before you act" step (see Safety rule 2).
- `page.domCua.*` — DOM-id tier: `getVisibleDom()` lists interactive elements as `node_id=N`;
  `click` / `scroll` act by node id; `type` / `keypress` act on the focused element.
- `page.cua.*` — vision tier: `screenshot()` returns a JPEG whose pixels map 1:1 to CSS coordinates;
  `click` / `move` / `scroll` / `type` act at those coordinates.

Named pages persist across script runs (the daemon keeps them), so you can navigate in one script and
act in the next. `browser.newPage()` is anonymous and cleaned up when the script exits. File helpers
(`saveScreenshot`, `writeFile`, `readFile`) are restricted to `~/.dev-browser/tmp/`.

Optional — pre-approve the command so Claude Code does not prompt each run (safe: scripts are
sandboxed). Add to `.claude/settings.json` (project) or `~/.claude/settings.json` (global):

```json
{ "permissions": { "allow": ["Bash(dev-browser *)"] } }
```

Full command + script API surface: [reference/dev-browser.md](reference/dev-browser.md).

---

## Fallback: playwright-cli attach

Use this when you want a stateful named session driven by discrete subcommands, or the
dedicated-debug-profile launcher. dev-browser automates the hardest part below (finding the
logged-in profile's endpoint); reach here when you need the subcommand model or `--connect` cannot
discover the browser. Full catalog: [reference/commands.md](reference/commands.md).

### Install

```bash
npm install -g @playwright/cli@latest
```

The binary is `playwright-cli` (invoke it directly, not via `npx`).

### Find the user's logged-in session first

The point of attaching is to reuse the user's EXISTING login — so first find which
running browser actually holds it. Do this BEFORE picking a method, and do NOT assume the
port is 9222.

1. Scan every running Chrome/Edge for its debug port and profile:

   ```bash
   ps aux | grep -iE '[r]emote-debugging-port' \
     | grep -oE -- '--remote-debugging-port=[0-9]+|--user-data-dir=[^ ]+'
   ```

   - The `chrome://inspect` "Allow remote debugging" toggle opens a **random high port**
     (e.g. 55267), not 9222 — read the real port, never hard-code it.
   - A `--user-data-dir` like `…/chrome-debug-profile` is a **dedicated debug profile**:
     EMPTY, not logged in. The user's real session is the **default profile**, which
     typically shows **no** `--user-data-dir` in the listing.

2. For the default profile, get the real port + WS path from its `DevToolsActivePort`
   file (line 1 = port, line 2 = `/devtools/browser/<GUID>`). `/json/version` returning
   empty/404 there is NORMAL (HTTP discovery off) — it is NOT a connection failure, so go
   straight to the file:

   ```bash
   # macOS (Linux: ~/.config/google-chrome/DevToolsActivePort; Edge: …/Microsoft Edge/…)
   F="$HOME/Library/Application Support/Google/Chrome/DevToolsActivePort"
   WS="ws://127.0.0.1:$(sed -n '1p' "$F")$(sed -n '2p' "$F")"
   playwright-cli attach --cdp="$WS" -s=<name>
   ```

3. **Verify the session is logged in before any work.** Right after attach, `tab-list` +
   `snapshot`. A visible "로그인 / Sign in / 회원가입" button or a fresh new-tab page means
   you hit the WRONG (empty) profile — stop and locate the default one. Only proceed once a
   logged-in signal (profile/account menu, the user's name, a my-page URL) is present.

### Choose an attach method

| Method | Command | When to use |
|---|---|---|
| Channel | `playwright-cli attach --cdp=chrome -s=<name>` | Default. The user's normally-launched Chrome/Edge with remote debugging enabled. |
| CDP endpoint | `playwright-cli attach --cdp=http://localhost:9222 -s=<name>` | Browser started with `--remote-debugging-port=9222`. |
| Server endpoint | `playwright-cli attach --endpoint=ws://localhost:3000 -s=<name>` | Connect to a running Playwright server. |
| Extension | `playwright-cli attach --extension -s=<name>` | Playwright browser extension installed. Best for SSO/2FA; reuses the live profile. |

Channel values for `--cdp=<channel>` / `--extension=<channel>`: `chrome`, `chrome-beta`,
`chrome-dev`, `chrome-canary`, `msedge`, `msedge-beta`, `msedge-dev`, `msedge-canary`.

Prefer **channel attach** on older Chrome/Edge. But on **Chrome 136+/148 default profiles**, `--cdp=chrome` and the `chrome://inspect` toggle alone routinely fail with `Timeout 30000ms exceeded`: the default profile needs the `--remote-allow-origins=*` relaunch flag AND the toggle TOGETHER (flag = origin allow-list, toggle = opens the gated port). Do NOT loop on `--cdp=chrome` — if attach times out once, go to "Troubleshooting attach failures" below. If a fresh login is acceptable, the **dedicated debug profile** (step 4) gives a deterministic, toggle-free attach — but that profile is EMPTY, so when the goal is to reuse the user's EXISTING login, find the default profile first (see "Find the user's logged-in session first"). When the default-profile gate keeps biting, prefer the dev-browser primary path above — it resolves the default profile's endpoint automatically.

### Browser setup (channel / CDP method)

In the target Chrome or Edge:

1. Open `chrome://inspect/#remote-debugging`.
2. Enable **Allow remote debugging for this browser instance**.
3. Keep the logged-in browser open.

The **extension** method needs the Playwright browser extension installed from the Chrome Web
Store instead — no `chrome://inspect` step.

### Session flag

Name the session once at attach, then reuse that name on every later command.

- Use `-s=<name>` (equals sign, **no space**). The long form `--session` does not exist.
- Or set `PLAYWRIGHT_CLI_SESSION=<name>` once to skip repeating the flag.

```bash
playwright-cli attach --cdp=chrome -s=work
playwright-cli -s=work tab-list
playwright-cli -s=work snapshot
playwright-cli -s=work goto https://example.com
```

### Operating an attached session

1. `tab-list`, then `tab-select` to land on the correct existing tab.
2. `snapshot` **before** any `click` / `fill` / `type` — never act on an unseen page.
3. Drive the page: `click`, `fill`, `type`, `goto`, etc.
4. Manage sessions when done: `list`, `close-all`, `kill-all`, `delete-data`.

Full command catalog: [reference/commands.md](reference/commands.md).

### Troubleshooting attach failures (Chrome 136+)

`attach --cdp=chrome` can fail after a Chrome update even though the browser runs with remote debugging. Work through these in order.

1. **`403 Forbidden` / `Connection rejected` / "Could not connect to chrome".** Chrome 136+ disables remote debugging on the DEFAULT user-data-dir until you opt in. In the target browser open `chrome://inspect/#remote-debugging` and enable **Allow remote debugging for this browser instance**, then wait until it shows `Server running at: 127.0.0.1:9222` (while it reads `starting…` the port is NOT up yet and attach keeps timing out). This is a user action — ask for it, never bypass it. On the default profile this toggle is REQUIRED in ADDITION to the relaunch flag in step 2 — the two together are what make attach work, so do not expect either one alone to be enough.

2. **`Timeout ... exceeded` right after `<ws connecting>`.** The WebSocket upgrades but no CDP data flows because Chrome 111+ rejects DevTools WS connections whose origin is not allow-listed. Relaunch the browser with `--remote-allow-origins=*` (quote it so the shell does not glob `*`). You cannot add this flag to an already-running instance.

3. **`--cdp=chrome` still fails / connects to a bare path.** Channel attach resolves the default profile's `DevToolsActivePort` to the bare `ws://host:<port>/devtools/browser` (no GUID), which modern Chrome rejects. Build the EXPLICIT endpoint from the `DevToolsActivePort` file instead — and read the port FROM that file, never assume 9222 (the `chrome://inspect` toggle opens a random high port). The exact commands are under "Find the user's logged-in session first" above; `/json/version` returning empty/404 on the default profile is normal (HTTP discovery off), so fall straight back to the file:

   ```bash
   # default profile: port + WS path both come from DevToolsActivePort
   F="$HOME/Library/Application Support/Google/Chrome/DevToolsActivePort"
   WS="ws://127.0.0.1:$(sed -n '1p' "$F")$(sed -n '2p' "$F")"
   playwright-cli attach --cdp="$WS" -s=<name>
   ```

   (This file-based discovery is exactly what dev-browser `--connect` does automatically — if you
   are stuck here, the dev-browser primary path is usually the faster fix.)

4. **Deterministic path (recommended) — dedicated debug profile.** This is the most reliable option and needs NO `chrome://inspect` toggle. Instead of fighting the default-profile gate (flag + toggle, often several rounds), start a separate instance with its OWN `--user-data-dir` on a free remote-debugging port. A non-default profile is not gated by Chrome 136+, so the relaunch flag alone opens CDP and `/json` normally. Note: a fresh profile is NOT logged in, so the user must sign in once (worth it for a one-shot deterministic attach).

   Use the bundled launcher — it resolves the port for you: it **reuses** the profile if it is already serving CDP, falls back to the next **free** port if the default (9333) is held by another process, and accepts a **pinned** port — erroring out (never silently moving) if that exact port is occupied by a non-CDP process. It launches the browser detached and prints the `webSocketDebuggerUrl`.

   ```bash
   # macOS / Linux
   bash scripts/launch-debug-chrome.sh -s=<name>          # auto port (9333, then scan up)
   bash scripts/launch-debug-chrome.sh -p 9500 -s=<name>  # pin a specific port
   ```

   ```powershell
   # Windows (PowerShell)
   pwsh scripts/launch-debug-chrome.ps1 -Session <name>            # auto port
   pwsh scripts/launch-debug-chrome.ps1 -Port 9500 -Session <name> # pin a specific port
   ```

   Extra flags: `-b msedge` / `-Browser msedge` for Edge, `-d` / `-ProfileDir` for a custom profile dir. Omit `-s` / `-Session` to just print `WS=...` and attach yourself.

   **Human-in-the-loop (do not bypass).** The launcher only *opens* the browser and binds a debug port — it never logs in. A person still signs in to the fresh debug profile, and on a default profile a person still flips the `chrome://inspect` "Allow remote debugging" toggle. These are user actions every time; ask for them, wait, and never script or work around them. The launcher also uses an isolated `--user-data-dir`, so it never touches the user's real logged-in default session.

   Manual fallback (no launcher) — 9222 is usually held by the main instance, so use a free port like 9333; if it is taken, pick another:

   ```bash
   open -na "Google Chrome" --args \
     --remote-debugging-port=9333 "--remote-allow-origins=*" \
     --user-data-dir="$HOME/.chrome-debug-profile" --no-first-run
   WS=$(curl -s -m5 http://127.0.0.1:9333/json/version | sed -n 's/.*"webSocketDebuggerUrl": *"\([^"]*\)".*/\1/p')
   playwright-cli attach --cdp="$WS" -s=<name>
   ```

   On macOS use `open -na` (launchd-owned) so the browser survives the calling shell; a plain `nohup ... &` child can be reaped when the command returns. On Linux launch the browser binary directly with the same flags (the `.sh` launcher uses `setsid` for this); on Windows use `Start-Process` (the `.ps1` launcher does this).

Quick diagnosis:
- `curl -s http://127.0.0.1:9222/json/version` → JSON with `webSocketDebuggerUrl` = healthy; `404` = HTTP discovery off (default profile) but the WS may still work after step 1.
- A WS-upgrade probe that TIMES OUT (no 403) means the toggle now allows the upgrade and you are hitting the origin/endpoint issue (steps 2-3), not the gate (step 1).

---

## Safety rules

These hold for every attached session, on any site, with EITHER tool.

1. Never ask for passwords, 2FA codes, payment details, or identity documents. The user logs in manually.
2. Look before acting: capture the page first — `page.snapshotForAI()` / `domCua.getVisibleDom()` (dev-browser) or `snapshot` (playwright-cli) — before every `click` / `fill` / `type`. Never act on an unseen page.
3. Work with existing tabs — `browser.listPages()` / `getPage(id)` (dev-browser) or `tab-list` / `tab-select` (playwright-cli); do not blindly open new ones.
4. Draft any outbound action (message, form submission, post) and show it first.
5. Never submit, send, upload, purchase, accept, or change account/profile settings without explicit approval for that exact target.
6. Prefer the least-privileged path, and clean up when finished — close the page, or `delete-data` / `kill-all` the playwright-cli session.

---
> Source: [cskwork/attach-to-browser-skill](https://github.com/cskwork/attach-to-browser-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
