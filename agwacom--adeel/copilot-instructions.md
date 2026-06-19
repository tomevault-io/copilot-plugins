## adeel

> |

<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Pre-check

Check the SessionStart hook output in this conversation context for `ADEEL_AUTO_UPDATE=`.
If it says `ADEEL_AUTO_UPDATE=false`, use AskUserQuestion to ask:
"Auto-updates are off. Run /adeel-update to enable?" If yes, invoke
`/adeel-update`. If `ADEEL_AUTO_UPDATE=true` or not found, proceed directly
without mentioning it.

## Preamble (run first)

```bash
mkdir -p $HOME/.adeel/sessions
touch $HOME/.adeel/sessions/"$PPID"
_SESSIONS=$(find $HOME/.adeel/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find $HOME/.adeel/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get adeel_contributor 2>/dev/null || true)
_PROACTIVE=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(${CLAUDE_PLUGIN_ROOT}/bin/adeel-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f $HOME/.adeel/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p $HOME/.adeel/analytics
echo '{"skill":"adeel","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> $HOME/.adeel/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
```

If `PROACTIVE` is `"false"`, do not proactively suggest adeel skills — only invoke
them when the user explicitly asks. The user opted out of proactive suggestions.

If `LAKE_INTRO` is `no`: Before continuing, introduce the Completeness Principle.
Tell the user: "adeel follows the **Go all the way** principle — always do the complete
Then offer to open the essay in their default browser:

```bash
```

Only run `open` if the user says yes. Always run `touch` to mark as seen. This only happens once.

If `TEL_PROMPTED` is `no` AND `LAKE_INTRO` is `yes`: After the lake intro is handled,
ask the user about telemetry. Use AskUserQuestion:

> Help adeel get better! Community mode shares usage data (which skills you use, how long
> they take, crash info) with a stable device ID so we can track trends and fix bugs faster.
> No code, file paths, or repo names are ever sent.
> Change anytime with `adeel-config set telemetry off`.

Options:
- A) Help adeel get better! (recommended)
- B) No thanks

If A: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry community`

If B: ask a follow-up AskUserQuestion:

> How about anonymous mode? We just learn that *someone* used adeel — no unique ID,
> no way to connect sessions. Just a counter that helps us know if anyone's out there.

Options:
- A) Sure, anonymous is fine
- B) No thanks, fully off

If B→A: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry anonymous`
If B→B: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry off`

Always run:
```bash
touch $HOME/.adeel/.telemetry-prompted
```

This only happens once. If `TEL_PROMPTED` is `yes`, skip this entirely.

## Contributor Mode

If `_CONTRIB` is `true`: you are in **contributor mode**. At the end of each major workflow step, rate your adeel experience 0-10. If not a 10 and there's an actionable bug or improvement — file a field report.

**File only:** adeel tooling bugs where the input was reasonable but adeel failed. **Skip:** user app bugs, network errors, auth failures on user's site.

**To file:** write `$HOME/.adeel/contributor-logs/{slug}.md`:
```
# {Title}
**What I tried:** {action} | **What happened:** {result} | **Rating:** {0-10}
## Repro
1. {step}
## What would make this a 10
{one sentence}
**Date:** {YYYY-MM-DD} | **Version:** {version} | **Skill:** /{skill}
```
Slug: lowercase hyphens, max 60 chars. Skip if exists. Max 3/session. File inline, don't stop.

## Completion Status Protocol

When completing a skill workflow, report status using one of:
- **DONE** — All steps completed successfully. Evidence provided for each claim.
- **DONE_WITH_CONCERNS** — Completed, but with issues the user should know about. List each concern.
- **BLOCKED** — Cannot proceed. State what is blocking and what was tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what you need.

### Escalation

It is always OK to stop and say "this is too hard for me" or "I'm not confident in this result."

Bad work is worse than no work. You will not be penalized for escalating.
- If you have attempted a task 3 times without success, STOP and escalate.
- If you are uncertain about a security-sensitive change, STOP and escalate.
- If the scope of work exceeds what you can verify, STOP and escalate.

Escalation format:
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

## Telemetry (run last)

After the skill workflow completes (success, error, or abort), log the telemetry event.
Determine the skill name from the `name:` field in this file's YAML frontmatter.
Determine the outcome from the workflow result (success if completed normally, error
if it failed, abort if the user interrupted).

**PLAN MODE EXCEPTION — ALWAYS RUN:** This command writes telemetry to
`$HOME/.adeel/analytics/` (user config directory, not project files). The skill
preamble already writes to the same directory — this is the same pattern.
Skipping this command loses session duration and outcome data.

Run this bash:

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f $HOME/.adeel/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

Replace `SKILL_NAME` with the actual skill name from frontmatter, `OUTCOME` with
success/error/abort, and `USED_BROWSE` with true/false based on whether `$B` was used.
If you cannot determine the outcome, use "unknown". This runs in the background and
never blocks the user.

## Plan Status Footer

When you are in plan mode and about to call ExitPlanMode:

1. Check if the plan file already has a `## ADEEL REVIEW REPORT` section.
2. If it DOES — skip (a review skill already wrote a richer report).
3. If it does NOT — run this command:

\`\`\`bash
${CLAUDE_PLUGIN_ROOT}/bin/adeel-review-read
\`\`\`

Then write a `## ADEEL REVIEW REPORT` section to the end of the plan file:

- If the output contains review entries (JSONL lines before `---CONFIG---`): format the
  standard report table with runs/status/findings per skill, same format as the review
  skills use.
- If the output is `NO_REVIEWS` or empty: write this placeholder table:

\`\`\`markdown
## ADEEL REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/adeel:plan-ceo-review\` | Scope & strategy | 0 | — | — |
| Codex Review | \`/adeel:codex review\` | Independent 2nd opinion | 0 | — | — |
| Eng Review | \`/adeel:plan-eng-review\` | Architecture & tests (required) | 0 | — | — |
| Design Review | \`/adeel:plan-design-review\` | UI/UX gaps | 0 | — | — |

**VERDICT:** NO REVIEWS YET — run \`/adeel:autoplan\` for full review pipeline, or individual reviews above.
\`\`\`

**PLAN MODE EXCEPTION — ALWAYS RUN:** This writes to the plan file, which is the one
file you are allowed to edit in plan mode. The plan file review report is part of the
plan's living status.

If `PROACTIVE` is `false`: do NOT proactively suggest other adeel skills during this session.
Only run skills the user explicitly invokes. This preference persists across sessions via
`adeel-config`.

If `PROACTIVE` is `true` (default): suggest adjacent adeel skills when relevant to the
user's workflow stage:
- Brainstorming → /adeel:office-hours
- Strategy → /adeel:plan-ceo-review
- Architecture → /adeel:plan-eng-review
- Design → /adeel:plan-design-review or /adeel:design-consultation
- Auto-review → /adeel:autoplan
- Debugging → /adeel:investigate
- QA → /adeel:qa
- Code review → /adeel:review
- Visual audit → /adeel:design-review
- Shipping → /adeel:ship
- Docs → /adeel:document-release
- Retro → /adeel:retro
- Second opinion → /adeel:codex
- Prod safety → /adeel:careful or /adeel:guard
- Scoped edits → /adeel:freeze or /adeel:unfreeze
- Upgrades → /adeel-upgrade

If the user opts out of suggestions, run `adeel-config set proactive false`.
If they opt back in, run `adeel-config set proactive true`.

# adeel browse: QA Testing & Dogfooding

Persistent headless Chromium. First call auto-starts (~3s), then ~100-200ms per command.
Auto-shuts down after 30 min idle. State persists between calls (cookies, tabs, sessions).

## SETUP (run this check BEFORE any browse command)

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/adeel/browse/dist/browse" ] && B="$_ROOT/.claude/skills/adeel/browse/dist/browse"
[ -z "$B" ] && B=${CLAUDE_PLUGIN_ROOT}/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If `NEEDS_SETUP`:
1. Tell the user: "adeel browse needs a one-time build (~10 seconds). OK to proceed?" Then STOP and wait.
2. Run: `cd <SKILL_DIR> && ./setup`
3. If `bun` is not installed: `curl -fsSL https://bun.sh/install | bash`

## IMPORTANT

- Use the compiled binary via Bash: `$B <command>`
- NEVER use `mcp__claude-in-chrome__*` tools. They are slow and unreliable.
- Browser persists between calls — cookies, login sessions, and tabs carry over.
- Dialogs (alert/confirm/prompt) are auto-accepted by default — no browser lockup.
- **Show screenshots:** After `$B screenshot`, `$B snapshot -a -o`, or `$B responsive`, always use the Read tool on the output PNG(s) so the user can see them. Without this, screenshots are invisible.

## QA Workflows

> **Credential safety:** Use environment variables for test credentials.
> Set them before running: `export TEST_EMAIL="..." TEST_PASSWORD="..."`

### Test a user flow (login, signup, checkout, etc.)

```bash
# 1. Go to the page
$B goto https://app.example.com/login

# 2. See what's interactive
$B snapshot -i

# 3. Fill the form using refs
$B fill @e3 "$TEST_EMAIL"
$B fill @e4 "$TEST_PASSWORD"
$B click @e5

# 4. Verify it worked
$B snapshot -D              # diff shows what changed after clicking
$B is visible ".dashboard"  # assert the dashboard appeared
$B screenshot /tmp/after-login.png
```

### Verify a deployment / check prod

```bash
$B goto https://yourapp.com
$B text                          # read the page — does it load?
$B console                       # any JS errors?
$B network                       # any failed requests?
$B js "document.title"           # correct title?
$B is visible ".hero-section"    # key elements present?
$B screenshot /tmp/prod-check.png
```

### Dogfood a feature end-to-end

```bash
# Navigate to the feature
$B goto https://app.example.com/new-feature

# Take annotated screenshot — shows every interactive element with labels
$B snapshot -i -a -o /tmp/feature-annotated.png

# Find ALL clickable things (including divs with cursor:pointer)
$B snapshot -C

# Walk through the flow
$B snapshot -i          # baseline
$B click @e3            # interact
$B snapshot -D          # what changed? (unified diff)

# Check element states
$B is visible ".success-toast"
$B is enabled "#next-step-btn"
$B is checked "#agree-checkbox"

# Check console for errors after interactions
$B console
```

### Test responsive layouts

```bash
# Quick: 3 screenshots at mobile/tablet/desktop
$B goto https://yourapp.com
$B responsive /tmp/layout

# Manual: specific viewport
$B viewport 375x812     # iPhone
$B screenshot /tmp/mobile.png
$B viewport 1440x900    # Desktop
$B screenshot /tmp/desktop.png

# Element screenshot (crop to specific element)
$B screenshot "#hero-banner" /tmp/hero.png
$B snapshot -i
$B screenshot @e3 /tmp/button.png

# Region crop
$B screenshot --clip 0,0,800,600 /tmp/above-fold.png

# Viewport only (no scroll)
$B screenshot --viewport /tmp/viewport.png
```

### Test file upload

```bash
$B goto https://app.example.com/upload
$B snapshot -i
$B upload @e3 /path/to/test-file.pdf
$B is visible ".upload-success"
$B screenshot /tmp/upload-result.png
```

### Test forms with validation

```bash
$B goto https://app.example.com/form
$B snapshot -i

# Submit empty — check validation errors appear
$B click @e10                        # submit button
$B snapshot -D                       # diff shows error messages appeared
$B is visible ".error-message"

# Fill and resubmit
$B fill @e3 "valid input"
$B click @e10
$B snapshot -D                       # diff shows errors gone, success state
```

### Test dialogs (delete confirmations, prompts)

```bash
# Set up dialog handling BEFORE triggering
$B dialog-accept              # will auto-accept next alert/confirm
$B click "#delete-button"     # triggers confirmation dialog
$B dialog                     # see what dialog appeared
$B snapshot -D                # verify the item was deleted

# For prompts that need input
$B dialog-accept "my answer"  # accept with text
$B click "#rename-button"     # triggers prompt
```

### Test authenticated pages (import real browser cookies)

```bash
# Import cookies from your real browser (opens interactive picker)
$B cookie-import-browser

# Or import a specific domain directly
$B cookie-import-browser comet --domain .github.com

# Now test authenticated pages
$B goto https://github.com/settings/profile
$B snapshot -i
$B screenshot /tmp/github-profile.png
```

> **Cookie safety:** `cookie-import-browser` transfers real session data.
> Only import cookies from browsers you control.

### Compare two pages / environments

```bash
$B diff https://staging.app.com https://prod.app.com
```

### Multi-step chain (efficient for long flows)

```bash
echo '[
  ["goto","https://app.example.com"],
  ["snapshot","-i"],
  ["fill","@e3","$TEST_EMAIL"],
  ["fill","@e4","$TEST_PASSWORD"],
  ["click","@e5"],
  ["snapshot","-D"],
  ["screenshot","/tmp/result.png"]
]' | $B chain
```

## Quick Assertion Patterns

```bash
# Element exists and is visible
$B is visible ".modal"

# Button is enabled/disabled
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"

# Checkbox state
$B is checked "#agree"

# Input is editable
$B is editable "#name-field"

# Element has focus
$B is focused "#search-input"

# Page contains text
$B js "document.body.textContent.includes('Success')"

# Element count
$B js "document.querySelectorAll('.list-item').length"

# Specific attribute value
$B attrs "#logo"    # returns all attributes as JSON

# CSS property
$B css ".button" "background-color"
```

## Snapshot System

The snapshot is your primary tool for understanding and interacting with pages.

```
-i        --interactive           Interactive elements only (buttons, links, inputs) with @e refs
-c        --compact               Compact (no empty structural nodes)
-d <N>    --depth                 Limit tree depth (0 = root only, default: unlimited)
-s <sel>  --selector              Scope to CSS selector
-D        --diff                  Unified diff against previous snapshot (first call stores baseline)
-a        --annotate              Annotated screenshot with red overlay boxes and ref labels
-o <path> --output                Output path for annotated screenshot (default: <temp>/browse-annotated.png)
-C        --cursor-interactive    Cursor-interactive elements (@c refs — divs with pointer, onclick)
```

All flags can be combined freely. `-o` only applies when `-a` is also used.
Example: `$B snapshot -i -a -C -o /tmp/annotated.png`

**Ref numbering:** @e refs are assigned sequentially (@e1, @e2, ...) in tree order.
@c refs from `-C` are numbered separately (@c1, @c2, ...).

After snapshot, use @refs as selectors in any command:
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # cursor-interactive ref (from -C)
```

**Output format:** indented accessibility tree with @ref IDs, one element per line.
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

Refs are invalidated on navigation — run `snapshot` again after `goto`.

## Command Reference

### Navigation
| Command | Description |
|---------|-------------|
| `back` | History back |
| `forward` | History forward |
| `goto <url>` | Navigate to URL |
| `reload` | Reload page |
| `url` | Print current URL |

### Reading
| Command | Description |
|---------|-------------|
| `accessibility` | Full ARIA tree |
| `forms` | Form fields as JSON |
| `html [selector]` | innerHTML of selector (throws if not found), or full page HTML if no selector given |
| `links` | All links as "text → href" |
| `text` | Cleaned page text |

### Interaction
| Command | Description |
|---------|-------------|
| `click <sel>` | Click element |
| `cookie <name>=<value>` | Set cookie on current page domain |
| `cookie-import <json>` | Import cookies from JSON file |
| `cookie-import-browser [browser] [--domain d]` | Import cookies from installed Chromium browsers (opens picker, or use --domain for direct import) |
| `dialog-accept [text]` | Auto-accept next alert/confirm/prompt. Optional text is sent as the prompt response |
| `dialog-dismiss` | Auto-dismiss next dialog |
| `fill <sel> <val>` | Fill input |
| `header <name>:<value>` | Set custom request header (colon-separated, sensitive values auto-redacted) |
| `hover <sel>` | Hover element |
| `press <key>` | Press key — Enter, Tab, Escape, ArrowUp/Down/Left/Right, Backspace, Delete, Home, End, PageUp, PageDown, or modifiers like Shift+Enter |
| `scroll [sel]` | Scroll element into view, or scroll to page bottom if no selector |
| `select <sel> <val>` | Select dropdown option by value, label, or visible text |
| `type <text>` | Type into focused element |
| `upload <sel> <file> [file2...]` | Upload file(s) |
| `useragent <string>` | Set user agent |
| `viewport <WxH>` | Set viewport size |
| `wait <sel|--networkidle|--load>` | Wait for element, network idle, or page load (timeout: 15s) |

### Inspection
| Command | Description |
|---------|-------------|
| `attrs <sel|@ref>` | Element attributes as JSON |
| `console [--clear|--errors]` | Console messages (--errors filters to error/warning) |
| `cookies` | All cookies as JSON |
| `css <sel> <prop>` | Computed CSS value |
| `dialog [--clear]` | Dialog messages |
| `eval <file>` | Run JavaScript from file and return result as string (path must be under /tmp or cwd) |
| `is <prop> <sel>` | State check (visible/hidden/enabled/disabled/checked/editable/focused) |
| `js <expr>` | Run JavaScript expression and return result as string |
| `network [--clear]` | Network requests |
| `perf` | Page load timings |
| `storage [set k v]` | Read all localStorage + sessionStorage as JSON, or set <key> <value> to write localStorage |

### Visual
| Command | Description |
|---------|-------------|
| `diff <url1> <url2>` | Text diff between pages |
| `pdf [path]` | Save as PDF |
| `responsive [prefix]` | Screenshots at mobile (375x812), tablet (768x1024), desktop (1280x720). Saves as {prefix}-mobile.png etc. |
| `screenshot [--viewport] [--clip x,y,w,h] [selector|@ref] [path]` | Save screenshot (supports element crop via CSS/@ref, --clip region, --viewport) |

### Snapshot
| Command | Description |
|---------|-------------|
| `snapshot [flags]` | Accessibility tree with @e refs for element selection. Flags: -i interactive only, -c compact, -d N depth limit, -s sel scope, -D diff vs previous, -a annotated screenshot, -o path output, -C cursor-interactive @c refs |

### Meta
| Command | Description |
|---------|-------------|
| `chain` | Run commands from JSON stdin. Format: [["cmd","arg1",...],...] |
| `frame <sel|@ref|--name n|--url pattern|main>` | Switch to iframe context (or main to return) |
| `inbox [--clear]` | List messages from sidebar scout inbox |
| `watch [stop]` | Passive observation — periodic snapshots while user browses |

### Tabs
| Command | Description |
|---------|-------------|
| `closetab [id]` | Close tab |
| `newtab [url]` | Open new tab |
| `tab <id>` | Switch to tab |
| `tabs` | List open tabs |

### Server
| Command | Description |
|---------|-------------|
| `connect` | Launch headed Chromium with Chrome extension |
| `disconnect` | Disconnect headed browser, return to headless mode |
| `focus [@ref]` | Bring headed browser window to foreground (macOS) |
| `handoff [message]` | Open visible Chrome at current page for user takeover |
| `restart` | Restart server |
| `resume` | Re-snapshot after user takeover, return control to AI |
| `state save|load <name>` | Save/load browser state (cookies + URLs) |
| `status` | Health check |
| `stop` | Shutdown server |

## Tips

1. **Navigate once, query many times.** `goto` loads the page; then `text`, `js`, `screenshot` all hit the loaded page instantly.
2. **Use `snapshot -i` first.** See all interactive elements, then click/fill by ref. No CSS selector guessing.
3. **Use `snapshot -D` to verify.** Baseline → action → diff. See exactly what changed.
4. **Use `is` for assertions.** `is visible .modal` is faster and more reliable than parsing page text.
5. **Use `snapshot -a` for evidence.** Annotated screenshots are great for bug reports.
6. **Use `snapshot -C` for tricky UIs.** Finds clickable divs that the accessibility tree misses.
7. **Check `console` after actions.** Catch JS errors that don't surface visually.
8. **Use `chain` for long flows.** Single command, no per-step CLI overhead.

---
> Source: [agwacom/adeel](https://github.com/agwacom/adeel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
