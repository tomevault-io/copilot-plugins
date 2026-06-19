## snip-skill

> |


# snip: Save Tokens Without Breaking Your Agent

snip is a CLI proxy that sits between your AI coding assistant and the shell.
It filters verbose command output — turning 689 tokens of `go test` into
"10 passed, 0 failed" (16 tokens). **60-90% token savings.**

But there's a catch: filters work silently. If a filter is too aggressive,
your agent gets incomplete information and doesn't know it missed something.
One wrong filter can turn a debugging session into a guessing game.

This guide shows you exactly which filters are safe, which are dangerous,
and — most importantly — **how to escape at any moment**.

---

## Before You Start: The Escape Protocol

Think of this like a plane safety briefing. You don't need it until you do.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OUTPUT LOOKS WRONG?                                           │
│                                                                 │
│  Step 1 — Bypass one command:                                   │
│    snip proxy <same command>                                    │
│    Example: snip proxy curl http://localhost:8585/health        │
│    Forces raw passthrough for that single command.              │
│                                                                 │
│  Step 2 — See what snip hid:                                   │
│    ls ~/.local/share/snip/tee/                                  │
│    Every filtered command's raw output is saved to disk.        │
│    Find the latest file and cat it.                             │
│                                                                 │
│  Step 3 — Disable the bad filter permanently:                  │
│    Edit ~/.config/snip/config.toml                              │
│    [filters.enable]                                             │
│    curl = false                                                 │
│    No restart needed — next command uses the new config.        │
│                                                                 │
│  Step 4 — Nuke from orbit (last resort):                        │
│    Remove "opencode-snip@latest" from opencode.json plugins.    │
│    Close terminal, reopen. snip is gone.                        │
│                                                                 │
│  SAFETY NET: Every filter has on_error: passthrough.            │
│  If a filter crashes or panics — raw output flows through.      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Test the escape now, before you need it:**

```bash
snip proxy echo "this works"     # bypasses snip entirely
snip echo "this goes through snip"  # (if snip has a filter for echo)
```

The plugin skips `cd`, `source`, `.`, `export`, `alias`, `unset`, `set`,
`shopt`, `eval`, and `exec` automatically. Those always run raw.

---

## How snip Filters Actually Work

A filter is a YAML file. The binary is the engine; filters are data.
They evolve independently. You can write one without touching Go.

Here's a real filter, fully annotated:

```yaml
name: "git-log"
version: 1
description: "Condense git log to hash + message + author + date"

match:
  command: "git"
  subcommand: "log"
  exclude_flags: ["--format", "--pretty", "--graph", "--oneline"]
  # ^ Only matches `git log` without these flags

inject:
  args: ["--pretty=format:%h %s (%ar) <%an>", "--no-merges"]
  defaults:
    "-n": "10"
  # ^ Injects arguments. If you run plain `git log`, snip runs:
  #   git log --pretty=format:"%h %s (%ar) <%an>" --no-merges -n 10
  # ^ THIS IS THE KEY INSIGHT — some filters modify your command.

pipeline:
  - action: "keep_lines"
    pattern: "\\S"
  - action: "truncate_lines"
    max: 80
    ellipsis: "..."
  - action: "format_template"
    template: "{{.count}} commits:\n{{.lines}}"
  # ^ Three pipeline stages: keep non-empty, truncate long lines, format

on_error: "passthrough"
  # ^ EVERY filter has this. If ANYTHING goes wrong — raw output passes through.
```

**Three things filters can do:**

| Action | Example | What it does |
|--------|---------|-------------|
| **Remove noise** | `remove_lines`, `strip_ansi` | Strips progress bars, ANSI colors, download logs |
| **Inject args** | `inject.args` | Rewrites your command to produce cleaner output |
| **Condense output** | `head`, `truncate_lines`, `aggregate` | Keeps first N lines, summarizes counts |

**The 19 pipeline actions:** `keep_lines`, `remove_lines`, `truncate_lines`,
`strip_ansi`, `head`, `tail`, `group_by`, `dedup`, `json_extract`,
`json_schema`, `ndjson_stream`, `regex_extract`, `state_machine`,
`aggregate`, `format_template`, `compact_path`, `replace`, `match_output`,
`on_empty`.

Every filter has at least one of these. Understanding which ones go too far
is the difference between safe savings and silent data loss.

### Commands without a matching filter

If you run a command that has no filter, snip passes it through unchanged
with ~10ms overhead. Over 70% of commands hit no filter in a typical session.

---

## Installation (3 ways)

```bash
# Option 1: Quick install script (macOS/Linux) — needs curl + tar
curl -fsSL https://raw.githubusercontent.com/edouard-claude/snip/master/install.sh | sh

# Option 2: Go (if you already have Go)
go install github.com/edouard-claude/snip/cmd/snip@latest

# Option 3: Homebrew (macOS only)
brew install edouard-claude/tap/snip
```

Verify:

```bash
snip --version
snip config               # shows current settings
```

### OpenCode plugin

Add to `~/.config/opencode/opencode.json`:

```json
{
  "plugin": ["opencode-snip@latest"]
}
```

The plugin uses the `tool.execute.before` hook to prefix every `bash` tool
call with `snip`. It skips commands already starting with `snip`, so
`snip proxy` works even with the plugin active.

### Other assistants

| Tool | Command |
|------|---------|
| Claude Code | `snip init` |
| Cursor | `snip init --agent cursor` |
| Copilot | `snip init --agent copilot` |
| Gemini CLI | `snip init --agent gemini` |

---

## The Safe Zone: Filters You Can Trust

These filters remove noise while preserving the information your agent needs.
They strip progress bars, download logs, and boilerplate — but errors,
warnings, results, and structure survive intact.

### How to read this section

Each entry shows:
- What the filter removes
- What the agent still sees
- A before/after example

### Git (except diff/show)

```yaml
git-log:
  removes: full commit metadata (author email, timestamp, parent hashes)
  keeps:   "hash message (ago) <author>", defaults to 10 entries
  safe:    ✅ You see commit history without the bloat

git-status:
  removes: verbose "On branch main\nYour branch is up to date..." header
  keeps:   porcelain filenames + status summary like "M: 2, ??: 1"
  safe:    ✅ All filenames preserved, just the greeting removed

git-branch:
  removes: nothing important
  keeps:   branch list, head 20 branches
  safe:    ✅ If you have >20 branches, add `-a` to skip the filter

git-push / git-pull / git-fetch / git-commit / git-stash / git-add / git-worktree:
  safe:    ✅ They strip remote progress noise and keep results/errors
```

### Test runners

```yaml
go-test:
  inject:  --json flag
  removes: verbose per-test JSON
  keeps:   "10 passed, 0 failed"
  safe:    ✅ Pass/fail is the signal

cargo-test:
  removes: Compiling/Downloading progress
  keeps:   test results (pass/fail/ignored) + failure details
  safe:    ✅

pytest:
  inject:  --tb=line -q
  removes: progress dots, stack trace context lines
  keeps:   "FAILED file::test" + "N passed, M failed"
  safe:    ✅

jest / vitest:
  removes: stack traces, source context, console.log
  keeps:   PASS/FAIL per suite + summary line
  safe:    ✅
```

### Linters and type checkers

```yaml
tsc:
  removes: source context lines, underline markers
  keeps:   "file.ts:line:col - error TS2304: message"
  safe:    ✅ The error location + code is the signal

eslint / ruff / mypy / basedpyright / biome / oxlint:
  removes: context lines, blank lines
  keeps:   "file:line:col error rule" + problem count
  safe:    ✅

shellcheck / hadolint / markdownlint / yamllint / pre-commit:
  removes: blank lines, ANSI
  keeps:   issues by file + "ok" on clean
  safe:    ✅
```

### Build tools

```yaml
make:
  removes: "make[N]: Entering/Leaving directory", "Nothing to be done"
  keeps:   ALL actual build output (commands, errors, warnings), head 50
  safe:    ✅ Only directory navigation noise removed

gradle / mvn:
  removes: > Task, Downloading, INFO boilerplate
  keeps:   BUILD result, errors, warnings
  safe:    ✅

cargo-build / cargo-check / cargo-clippy:
  removes: Compiling/Checking/Fresh/Downloading progress
  keeps:   errors, warnings, "Finished" line
  safe:    ✅

go-build / go-vet / golangci-lint:
  removes: compilation progress, blank lines
  keeps:   errors only, "ok (compiled)" on success
  safe:    ✅
```

### Package managers

```yaml
npm-install:
  removes: deprecation warnings, funding nags, "npm notice"
  keeps:   "added 245 packages" or "up to date"
  safe:    ✅ Errors are preserved (ERESOLVE, peer dep conflicts)

pip-install:
  removes: Downloading/Using cached progress, pip version notice
  keeps:   "Successfully installed ..." or "ERROR: ..."
  safe:    ✅

pnpm-install / brew-install / composer-install / poetry-install:
  removes: progress bars, download logs
  keeps:   result summary + errors
  safe:    ✅
```

### Docker (except logs)

```yaml
docker-build:
  removes: sub-step sha256, transferring context timing
  keeps:   step names (FROM, RUN, COPY), ERROR lines, final result
  safe:    ✅ Build output is 90% noise, errors fit in 30 lines

docker-ps / docker-images:
  removes: nothing significant
  keeps:   all output, head 30 entries
  safe:    ✅

docker-compose:
  removes: Pulling/Extracting/Waiting progress
  keeps:   service status
  safe:    ✅
```

### Infrastructure

```yaml
terraform / tofu:
  removes: blank lines
  keeps:   plan changes (+/-/~ lines), errors
  safe:    ✅

kubectl-get:
  removes: blank lines
  keeps:   resources with status, head 30
  safe:    ✅

gh-pr / gh-issue / gh-run:
  removes: blank lines
  keeps:   all content, head 30 entries
  safe:    ✅

helm / ansible-playbook / gcloud / aws:
  removes: progress/WARNING noise
  keeps:   results, errors, status
  safe:    ✅
```

---

## The Danger Zone: Filters to Disable

These filters are the reason this guide exists. They strip or truncate
information that your agent may need. **Disable them all.**

### Critical: curl and wget

**curl** — the most dangerous filter in the set.

```yaml
# ~/.config/snip/filters/curl.yaml (simplified)
pipeline:
  - action: "remove_lines"
    pattern: "^\\s*(\\*|>|\\{|\\}) "
    # ^ Removes lines STARTING with *, >, {, }
  - action: "head"
    n: 50
    overflow_msg: "... response truncated"
```

**What this breaks:**

```
Without snip (curl http://localhost:8585/health):
{
  "status": "healthy",
  "llm_loaded": true,
  "gpu": { "cuda": true, "vram": "3.5/6GB" }
}

With snip (curl filter active):
... response truncated
```

The filter removes lines starting with `{`. JSON objects start with `{`.
**Your entire API response is deleted.** The agent sees "truncated" — but
the actual content was removed before truncation even applied.

Similarly, `>` lines are HTTP request headers. Gone for debugging.

**Disable it:** `curl = false` in config.

---

**wget** — less destructive but still strips useful info.

```yaml
pipeline:
  - action: "keep_lines"
    pattern: "(Saving|saved|Resolving|Connecting|HTTP|ERROR|failed|Downloaded|Length:)"
```

Keeps only these specific patterns. Any custom output (--output-document,
post-download processing, webpage content) is discarded.

**Disable it:** `wget = false`.

---

### Critical: git-diff and git-show

These inject `--stat`, replacing actual code changes with file-level stats.

**git-diff:**

```
Without snip (git diff):
diff --git a/src/main.rs b/src/main.rs
index abc..def 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -10,6 +10,8 @@ fn handle_request(req: Request) -> Response {
     let user = req.user();
+    if !user.is_authenticated() {
+        return Response::unauthorized();
+    }
     // ...

With snip (git-diff filter active):
3 files changed, 18 insertions(+), 5 deletions(-)
```

You see *which* files changed and *how many* lines. You don't see
*what changed*. For code review, debugging, or commit analysis —
this is useless.

**git-show** has the same injection. `git show <hash>` becomes
`git show --stat <hash>` — file stats only, no patch content.

**Workaround:** Use `git diff --name-only` or `git show -p` (the `-p` flag
is in `skip_if_present` so it bypasses the filter). But you have to
remember to do this every time. Better to disable.

**Disable them:** `git-diff = false`, `git-show = false`.

---

### Critical: systemctl

```yaml
pipeline:
  - action: "keep_lines"
    pattern: "(Loaded:|Active:|Main PID:|Status:|\\u25cf|failed|running|inactive|enabled|disabled|dead)"
```

Keeps only 4 fields: Loaded, Active, Main PID, Status. Everything else
is stripped — including the service file contents, journal output, and
dependency information.

**Example:**

```
Without snip (systemctl status nginx):
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Tue 2026-05-05 14:22:31 UTC; 2 days ago
       Docs: man:nginx(8)
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
   Main PID: 1234 (code=exited, status=1/FAILURE)
        CPU: 5ms

With snip (systemctl filter):
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Tue 2026-05-05 14:22:31 UTC; 2 days ago
   Main PID: 1234 (code=exited, status=1/FAILURE)
```

Looks similar, right? But look what's missing: **the Process line**.
"ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)" —
the actual reason the service failed. The filter kept "Main PID" but
discarded the ExecStartPre line that explains *why* it failed.

**Disable it:** `systemctl = false`.

---

### High risk: search and data tools

```yaml
find:
  head: 50 files
  # Agent runs find . -name "*.rs" — 200 matches. Sees 50. Thinks there are 50.

grep / rg:
  head: 50 matches
  # Agent runs grep -r "TODO" src/ — 500 matches. Sees 50. Misses 450 TODOs.

jq:
  head: 50 lines
  # Agent runs jq '.' data.json — JSON has 200 lines. Sees 50. Gets incomplete data.

psql:
  head: 40 rows
  # Agent runs SELECT * FROM users — 1000 rows. Sees 40. Missing 960.

ps:
  head: 30 processes
  # Agent runs ps aux to find a runaway process. Sees 30. Process is #31.
```

**The head limit pattern:** These filters use `head` with overflow messages.
The overflow message tells you data was cut — but by then, the agent has
already acted on incomplete information.

**Disable them:** `find = false`, `grep = false`, `rg = false`, `jq = false`,
`psql = false`, `ps = false`.

---

### Medium risk: system commands

```yaml
ssh:
  head: 30 lines
  # Agent SSHs into a server, runs a diagnostic. Output: 100 lines. Sees 30.

ping:
  keep: only statistics
  # "ping google.com -c 10" — only shows "5 packets transmitted, 5 received"
  # Agent sees success but misses timing details.

df:
  head: 20 lines, removes tmpfs/devtmpfs
  # If disk is full, agent processes first 20 filesystems. The full one is #21.

du:
  head: 30 entries
  # "du -sh /*" — 50 directories. Sees largest 30. Misses 20.

diff:
  head: 60 lines
  # Comparing two large files. 200 lines differ. Sees 60. Misses 140.

rsync:
  keeps: only sent/received/speedup/error
  # File sync report stripped to 4 metrics. Agent can't verify individual files.

stat:
  head: 20 lines
  # "stat largefile.bin" output truncated.

iptables:
  head: 30 rules
  # 100 iptables rules. Sees 30. Misses the DROP rule at position 42.

fail2ban:
  head: 20 lines
  # Ban status truncated.
```

**Disable them:** `ssh = false`, `ping = false`, `df = false`, `du = false`,
`diff = false`, `rsync = false`, `stat = false`, `iptables = false`,
`fail2ban = false`.

---

### Medium risk: logs

```yaml
docker-logs:
  dedup: 30 patterns, normalizes timestamps and IPs
  # 500 lines of container logs. Dedup finds 30 "unique" patterns.
  # An intermittent error that appears 3 times in 500 lines? Collapsed into
  # one. Agent sees the error once, doesn't know it's recurring.

kubectl-logs:
  dedup: 30 patterns (same as docker-logs)
  # Same problem: deduplication hides frequency and distribution.
```

These use `dedup` which normalizes timestamps and IPs then keeps the top N
patterns. It's useful for finding unique log lines in massive output, but
dangerous when frequency matters.

**Disable them:** `docker-logs = false`, `kubectl-logs = false`.

---

### Low risk: other

```yaml
sops:
  head: 30 lines
  # Secrets decryption output truncated.

ollama:
  removes: pulling/verifying/writing progress
  head: 20 lines
  # Model download progress stripped, list truncated.
```

**Disable them:** `sops = false`, `ollama = false`.

---

## Recommended Config: Full TOML

Copy this to `~/.config/snip/config.toml`:

```toml
[display]
color = true
emoji = true
quiet_no_filter = true

[tee]
enabled = true
mode = "failures"
max_files = 20

[filters.enable]
# ===================== DISABLED =====================
# These filters strip context the agent needs.
# See "The Danger Zone" section above for details.

# Network — curl removes { and > lines, wget strips non-HTTP output
curl = false
wget = false

# Search — head limits hide results
find = false
grep = false
rg = false

# Data tools — head limits truncate structured data
jq = false
psql = false

# System — head limits + aggressive stripping hide debugging info
ps = false
systemctl = false
df = false
du = false
stat = false
ping = false
ssh = false
rsync = false
diff = false
iptables = false
fail2ban = false

# Logs — dedup hides recurring errors
docker-logs = false
kubectl-logs = false

# Git — --stat injection hides code changes
git-diff = false
git-show = false

# Other
sops = false
ollama = false

# ===================== ENABLED =====================
# These filters remove noise and preserve signal.

# Git (safe subcommands)
git-log = true
git-status = true
git-branch = true
git-commit = true
git-push = true
git-pull = true
git-fetch = true
git-stash = true
git-add = true
git-worktree = true

# Go
go-test = true
go-build = true
go-vet = true
golangci-lint = true

# Rust
cargo-test = true
cargo-build = true
cargo-check = true
cargo-clippy = true
cargo-install = true
cargo-nextest = true

# Python
pytest = true
ruff = true
mypy = true
basedpyright = true
pip-install = true
poetry-install = true

# JavaScript / TypeScript
jest = true
vitest = true
tsc = true
eslint = true
biome = true
oxlint = true
prettier = true
npm-install = true
npx = true
pnpm-install = true
pnpm-list = true
next-build = true
turbo = true
nx = true

# Docker (except logs)
docker-build = true
docker-ps = true
docker-images = true
docker-compose = true

# Kubernetes
kubectl-get = true

# GitHub CLI
gh-pr = true
gh-issue = true
gh-run = true

# Build tools
make = true
gradle = true
mvn = true
just = true
task = true

# Infrastructure
terraform = true
tofu = true
helm = true
ansible-playbook = true
gcloud = true
aws = true

# Ruby
rubocop = true
rspec = true
rake = true
bundle-install = true
rails-migrate = true
rails-routes = true

# .NET
dotnet-build = true
dotnet-test = true
dotnet-format = true

# Linters
shellcheck = true
hadolint = true
markdownlint = true
yamllint = true
pre-commit = true

# Elixir
mix-compile = true
mix-format = true

# Other safe filters
jira = true
prisma = true
playwright = true
liquibase = true
quarto = true
```

**70 filters enabled, 24 disabled.** The disabled ones are disabled for a
specific, documented reason — not "just in case." Read the Danger Zone
section above to understand each one.

---

## How to Customize

### Disable a built-in filter you disagree with

```toml
[filters.enable]
pytest = false    # maybe you want full test output
cargo-build = false  # maybe you want to see compilation progress
```

### Write a custom filter

Create a YAML file in `~/.config/snip/filters/`:

```yaml
# ~/.config/snip/filters/my-tool.yaml
name: "my-tool"
version: 1
description: "Custom filter for my-tool"

match:
  command: "my-tool"

pipeline:
  - action: "keep_lines"
    pattern: "(error|warning|result)"
  - action: "head"
    n: 20
    overflow_msg: "... more output"

on_error: "passthrough"
```

User filters override built-in ones with the same name.

### Test a filter without affecting your agent

```bash
snip git log -5           # see filtered output
git log -5                # compare with raw output
```

If they differ more than expected, disable or adjust the filter.

---

## Debugging Commands

```bash
snip config                # show effective config
snip gain                  # token savings dashboard
snip gain --daily          # today only
snip gain --json           # machine-readable
snip gain --top 10         # top commands by savings
snip discover              # find commands that could benefit from filters
snip discover --since 30   # scan last 30 days
```

---

## Summary

| Concept | Takeaway |
|---------|----------|
| **Escape** | `snip proxy <cmd>` bypasses everything |
| **Safety net** | `on_error: passthrough` on every filter |
| **Raw output** | `~/.local/share/snip/tee/` saves every filtered command |
| **Safe filters** | Test runners, linters, build tools, package managers, git (except diff/show) |
| **Dangerous filters** | curl, systemctl, git-diff, git-show, find, grep, jq, psql — disable all |
| **Config** | Copy the recommended config above |
| **Customize** | Disable any filter you disagree with, or write your own |

snip saves a lot of tokens. The safe filters alone (70 of them) cover
most of the noise in a typical AI coding session. The 24 disabled filters
would cost more in lost context than they'd save in tokens.

Install it. Use the config above. And remember the escape protocol —
you'll probably never need it, but you'll be glad it's there.

---

## Limitations: Loops & Complex Commands

snip proxies a single command at a time — it matches `command` and
`subcommand` from the first word of the shell invocation. Shell parser
keywords like `for`, `while`, `until`, `if`, and `case` are **not** binaries;
they are part of the shell's syntax. Passing them as the "command" to snip
will always fail.

### The Problem

```bash
# THIS DOES NOT WORK:
snip for ip in 192.168.100.{101..110}; do
  ssh root@$ip "hostname"
done
# → bash: syntax error near unexpected token 'do'
```

When snip receives `for` as the command, bash sees it as a literal argument
rather than a loop keyword. The `do` keyword has no matching `for` to pair
with, producing a syntax error. The same applies to `while`, `until`,
`if/then/else`, and `case/esac`.

### Why

Snip's `unproxyableReason` in `internal/cli/cli.go` currently lists 40+
shell builtins (cd, export, alias, etc.) but is missing shell **parser
keywords**: `for`, `while`, `until`, `select`, `if`, `case`, `fi`, `esac`,
`then`, `elif`, `else`, `done`.

These are not binaries in `$PATH` — they are understood only by the shell's
parser. Running them as external commands through snip always fails.

### Upstream Issue

Filed for the snip project (`edouard-claude/snip`):

- **Issue**: [#66](https://github.com/edouard-claude/snip/issues/66) — reported
- **Pull Request**: [#68](https://github.com/edouard-claude/snip/pull/68) — fix + corporate config submitted (replaces closed #67)
- **Missing**: `for`, `while`, `until`, `select`, `if`, `case`, and their
  closing keywords in `unproxyableReason`
- **Fix**: ~6 lines added to the `switch` block in `internal/cli/cli.go`

**After the fix lands** (snip version containing this patch):
- The workarounds below are no longer needed — snip will reject loops with a
  clear error instead of producing a cryptic bash syntax error.
- If `snip --version` returns a version newer than the fix, skip workarounds
  1-4 and let snip handle it naturally.
- This skill section will be updated to mark the workarounds as deprecated.

Note: a companion design proposal for corporate/project-level `.snip/config.toml`
was submitted as a response to [#65](https://github.com/edouard-claude/snip/issues/65).

### Workarounds

**Workaround 1 — Put snip inside the loop body**

```bash
# Add snip to each command inside the loop
for ip in 192.168.100.{101..110}; do
  snip ssh root@$ip "systemctl status nginx" 2>&1
done
```

**Workaround 2 — Wrap as a function**

```bash
# Let snip run the entire function
scan_hosts() {
  for ip in 192.168.100.{101..110}; do
    ssh root@$ip "systemctl status nginx" 2>&1
  done
}
snip scan_hosts
```

**Workaround 3 — snip proxy inside loops**

```bash
# Bypass filters but keep agent integration
for ip in 192.168.100.{101..110}; do
  snip proxy ssh root@$ip "systemctl status nginx" 2>&1
done
```

**Workaround 4 — Patch UNPROXYABLE_COMMANDS in opencode-snip plugin**

If you run snip via the OpenCode plugin (`opencode-snip@latest`), its
`UNPROXYABLE_COMMANDS` list also needs these keywords:

```typescript
// node_modules/opencode-snip/src/index.ts
const UNPROXYABLE_COMMANDS = new Set([
  "cd", "source", ".", "export", "alias", "unset", "set", "shopt", "eval", "exec",
  // ADD:
  "for", "while", "until", "select",
  "if", "case", "fi", "esac", "then", "elif", "else",
])
```

### What the Safe Config Does

Our recommended config (`config.toml` with 24 disabled + 70 enabled filters)
is unaffected by this limitation. The loop issue lives in snip's CLI routing
layer (`unproxyableReason`), not in the filter pipeline. Even if all filters
are perfectly configured, a `snip for ...` invocation still breaks at the
parsing stage.

---

## Corporate & Team Deployment

snip can be configured at the **project level** so every developer on a team
gets the same filter behavior — without touching their personal config.

### How It Works

```
Team lead creates:  .snip/config.toml  (committed to repo)
Developer clones:   git clone team-repo
Developer works:    snip auto-detects .snip/config.toml → applies team settings
No per-developer:   No snip trust, no .config edits, no env vars needed
```

### .snip/config.toml — Full Reference

```toml
# ── Mode ────────────────────────────────────────────────────
mode = "project"          # "user" (default) or "project"
                          # "project" = team config wins over user config

# ── Global Parameters ──────────────────────────────────────
[filters.global]
max_lines = 0             # 0 = unlimited. Caps all filtered output.
max_line_length = 0       # 0 = unlimited. Caps per-line length.
stream_mode = "filter"    # "filter" | "full" | "truncate_only"

# ── Per-Filter Enable/Disable (team policy) ────────────────
[filters.enable]
curl = false              # Always disabled for this org
docker-logs = false

# ── Per-Filter Parameter Overrides ─────────────────────────
# Each [filters.override.<name>] targets one built-in filter.
# Keys map to pipeline action parameters:
#
#   head = N           → head action, param "n"
#   truncate_lines = N → truncate_lines action, param "max"
#   keep_lines = "rx"  → keep_lines action, param "pattern"
#   stream_mode="full" → skip entire pipeline for this filter

[filters.override.ls]
head = 0                # Never truncate ls output

[filters.override.dotnet-test]
stream_mode = "full"    # Always show full test output

[filters.override.make]
stream_mode = "full"    # CI needs full build output

[filters.override.pytest]
head = 0                # Full pytest output always

[filters.override.go-test]
head = 0

[filters.override.docker-build]
head = 200

[filters.override.npm-install]
keep_lines = "(error|warn|deprecated|notice|added|removed|packages)"

# ── Bypass (always skip filtering for these commands) ─────
[filters.bypass]
commands = ["curl", "wget", "jq", "psql"]

# ── Tee (raw output cache) ─────────────────────────────────
[tee]
enabled = true
mode = "always"           # "always" | "failures" | "never"
max_files = 100
max_file_size = 10485760  # 10MB for large build outputs
```

### Trust Model

| File | Trust Required | Why |
|------|---------------|-----|
| `.snip/config.toml` | **No** — auto-loaded | Parameter overrides only (head limits, enable/disable). Cannot execute code. |
| `.snip/filters/*.yaml` | **Yes** — `snip trust` | Custom filter YAMLs can inject args and run pipeline actions. Must be verified. |

### Two Personas

| | Vibe Coder | Team Developer |
|---|---|---|
| Setup | `curl install.sh \| sh` | `git clone` + `.snip/config.toml` in repo |
| Config | None needed | Auto-detected from repo root |
| Trust | Never needed | Only for custom filter YAMLs |
| Escape | `snip proxy <cmd>` | Same |
| Who controls | Themselves | Team lead via committed `.snip/config.toml` |

### When to Use Each Mode

- **`mode = "project"`**: Corporate/team repos. Team lead's config is authoritative.
  Individual developers cannot override. Use for: security policies, CI consistency,
  per-project output limits.
- **`mode = "user"`** (default): Personal projects. The developer can override any
  project config setting with their own `~/.config/snip/config.toml`.

### Design Status

This feature was proposed in response to
[edouard-claude/snip#65](https://github.com/edouard-claude/snip/issues/65)
with a full implementation design (~180 lines of Go code in `config.go`,
`pipeline.go`, and a new `projectconfig.go`). Awaiting maintainer review.

### Upstream Tracker

| Issue | Topic | Status |
|-------|-------|--------|
| [#66](https://github.com/edouard-claude/snip/issues/66) | Loops/complex commands break snip | PR [#68](https://github.com/edouard-claude/snip/pull/68) submitted |
| [#65](https://github.com/edouard-claude/snip/issues/65) | Corporate `.snip/config.toml` | PR [#68](https://github.com/edouard-claude/snip/pull/68) submitted |

---

## Credits

- **[edouard-claude](https://github.com/edouard-claude)** — creator of [snip](https://github.com/edouard-claude/snip), the CLI proxy and all 126 built-in YAML filters. Inspired by [rtk](https://github.com/rtk-ai/rtk).
- **[VincentHardouin](https://github.com/VincentHardouin)** — creator of [opencode-snip](https://github.com/VincentHardouin/opencode-snip), the OpenCode plugin that auto-wraps `bash` tool calls.
- **[DeepSeek](https://deepseek.com)** — AI model provider (deepseek-v4-pro via Anthropic-compatible API) powering the agent behind this skill's analysis and curation.
- This skill analyzes and curates their work for safe agent use — filter audit, escape protocol, and recommended config are original to this project.

---
> Source: [EnRaiha/snip-skill](https://github.com/EnRaiha/snip-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
