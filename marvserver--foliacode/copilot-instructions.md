## foliacode

> This file tells a coding assistant (Claude Code, Codex, Cursor, Copilot, or any

# AGENTS.md — driving FoliaCode from an AI assistant

This file tells a coding assistant (Claude Code, Codex, Cursor, Copilot, or any
agent framework) how to operate FoliaCode correctly. It is written to be read by a
machine and acted on without further explanation.

If you are a human, you probably want [README.md](README.md) instead.

---

## The one rule that matters

**`analyze` is always safe. `verify` executes downloaded third-party code — ask the
user before you run it. `transform` writes a new JAR — dry-run it first.**

`foliacode analyze` only reads bytecode. It touches no network, starts no process,
and changes no file unless `--json` is given. Run it freely.

`foliacode verify` downloads a Folia server jar from PaperMC and executes it on the
user's machine. It is gated behind explicit consent (`--yes`, or `enabled: true` in
`foliacode.yml`) precisely so that an agent cannot enable it by accident.

Do not pass `--yes` on your own initiative. Ask first, in plain language:

> "I can also boot a throwaway Folia server to confirm the plugin actually loads.
> That downloads a ~50 MB server jar from PaperMC and runs it locally for about a
> minute, then deletes it. Want me to?"

If the user has already put `enabled: true` in `foliacode.yml`, that is their
standing consent and you may run `verify` without asking again.

`foliacode transform` never modifies the input JAR — it writes a new one beside it —
so it is not destructive. But it produces an artifact the user may deploy, so run
`--dry-run` first and show them what would change before writing anything. Never
describe a transformed JAR as "Folia-ready": say which call sites were rewritten and
which were refused.

---

## Quick reference

```bash
# Static analysis — safe, offline, fast (~0.4s for 2000 classes)
foliacode analyze MyPlugin.jar

# Every JAR in a directory
foliacode analyze plugins/

# Machine-readable output for you to parse
foliacode analyze plugins/ --json report.json

# Gate a CI job on thread-safety issues, not just fatal ones
foliacode analyze plugins/ --fail-on HIGH

# Show every call site instead of the first three per rule
foliacode analyze MyPlugin.jar --verbose

# List the rule set (useful for explaining a finding to a user)
foliacode rules

# Real server boot — ONLY after the user agrees
foliacode verify MyPlugin.jar --yes --memory 2048

# Boot with dependency plugins installed alongside
foliacode verify MyPlugin.jar --yes --with Vault.jar --with ProtocolLib.jar

# Boot and record which flagged calls actually ran, and on which threads
foliacode verify MyPlugin.jar --yes --instrument

# See what a rewrite would change — safe, writes nothing
foliacode transform MyPlugin.jar --dry-run

# Write the rewritten JAR (the original is left alone)
foliacode transform MyPlugin.jar --json transform.json

# Print the -javaagent line for a server the user runs themselves
foliacode agent MyPlugin.jar
```

---

## Exit codes

Branch on these rather than parsing prose. They are stable API.

### `analyze`

| Code | Meaning | What to do |
|------|---------|------------|
| `0` | Nothing at or above `--fail-on` | Report success |
| `1` | Findings at or above the threshold | Report the findings |
| `2` | Execution error (bad args, unreadable file) | Fix the invocation |

### `verify`

| Code | Meaning | What to do |
|------|---------|------------|
| `0` | Server booted, plugin enabled | Report success |
| `1` | **The plugin is at fault** — missing dependency, class error, exception on load | Report it as a plugin problem |
| `2` | Could not run — bad args, file missing, or consent not given | Fix the invocation; ask the user about consent |
| `3` | **The environment is at fault** — out of memory or timeout | Re-run with more `--memory` or a longer `--timeout`. **Do not tell the user their plugin is broken.** |

Exit code `3` exists specifically so you never misattribute an out-of-memory
server to the plugin under test. Treat `1` and `3` as different conclusions.

### `transform`

| Code | Meaning | What to do |
|------|---------|------------|
| `0` | Nothing at `HIGH` or above was refused | Report what was rewritten, and the caveats |
| `1` | Call sites remain that need a person | Report the refusals and why each one was refused |
| `2` | Could not run — bad args, missing file, signed JAR | Fix the invocation |

Exit `1` is the normal outcome for a real plugin. It is not a failure; it means the
tool did what it could and is telling you what it could not.

---

## Parsing the JSON

`--json <file>` writes one object for a single JAR, or an array of those objects
when the target was a directory.

```json
{
  "jar": "MyPlugin.jar",
  "plugin": { "name": "MyPlugin", "version": "1.0.0",
              "main": "com.example.MyPlugin", "foliaSupported": null },
  "classesScanned": 2149,
  "classesFailed": 0,
  "nestedJarsScanned": 0,
  "verdict": "NOT_READY",
  "summary": { "CRITICAL": 7, "HIGH": 60, "MEDIUM": 28, "INFO": 131 },
  "findings": [
    {
      "severity": "CRITICAL",
      "category": "SCHEDULER",
      "api": "BukkitScheduler.runTask",
      "ruleOwner": "org/bukkit/scheduler/BukkitScheduler",
      "calleeOwner": "org/bukkit/scheduler/BukkitScheduler",
      "class": "com.example.MyPlugin",
      "method": "onEnable",
      "descriptor": "()V",
      "line": 42,
      "callKind": "DIRECT_CALL",
      "viaSubtype": false,
      "reason": "...",
      "remedy": "..."
    }
  ]
}
```

Field notes that matter when you summarise for a user:

- **`verdict`** — `READY` | `NEEDS_REVIEW` | `NOT_READY`. Lead with this.
- **`severity`** — `CRITICAL` means Folia throws and the plugin certainly breaks.
  `HIGH` means a thread-safety violation. `INFO` means static analysis could not
  see through (reflection); it is a note, not a defect.
- **`callKind`** — `METHOD_REFERENCE` means it was found inside an `invokedynamic`
  bootstrap (e.g. `Block::setType`), not a plain call instruction.
- **`viaSubtype`** — `true` means the call site's owner was a subtype of the rule's
  owner. Worth mentioning: exact-name matching would have missed it.
- **`line`** — `-1` when the JAR was compiled without debug info.

---

## How to summarise findings for a human

Do not dump the whole list. A 226-finding report is not useful raw.

1. Lead with `verdict` and the `summary` counts.
2. Explain every `CRITICAL` — these are certain breakage, and there are usually few.
3. Group `HIGH` by `category` and give the count plus one example each.
4. Mention `INFO`/reflection once, as a limitation, not as a list of defects.
5. Quote `remedy` when the user asks how to fix something. It is written to be shown.

A clean report is **not** proof of safety. If `verdict` is `READY`, say that static
analysis found nothing *and* that reflection cannot be followed.

---

## Typical workflows

**"Is this plugin Folia-ready?"**
```bash
foliacode analyze MyPlugin.jar --json /tmp/r.json
```
Read the JSON, lead with the verdict, explain the CRITICALs.

**"Check all my server's plugins."**
```bash
foliacode analyze plugins/ --json /tmp/r.json
```
The JSON is an array. Sort by verdict and report the broken ones first.

**"Will it actually run?"**
Ask for consent, then:
```bash
foliacode verify MyPlugin.jar --yes --memory 2048
```
On exit `1`, read the `Missing dependency plugins` section — it names the plugins
to install. On exit `3`, raise the heap and retry; say nothing about the plugin.

**"Add it to CI."**
```bash
foliacode analyze plugins/ --fail-on CRITICAL
```
Use `--fail-on HIGH` only when the project has already dealt with its CRITICALs,
otherwise the build is red from day one and people stop reading it.

---

## Things not to do

- Do not pass `--yes` without asking.
- Do not report exit code `3` as a plugin defect.
- Do not describe a `READY` verdict as "safe" or "verified" — it means "nothing
  detected by static analysis".
- Do not run `verify` in CI without the user having configured it deliberately;
  it needs network access and a couple of GB of RAM.
- Do not run `transform` without dry-running it first and showing the user the result.
- Do not present a transformed JAR as finished work when the exit code was `1`.
- Do not read "this call site never executed" from a runtime report as "this call site
  is safe". It means the run never reached it.

---
> Source: [MARVserver/Foliacode](https://github.com/MARVserver/Foliacode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
