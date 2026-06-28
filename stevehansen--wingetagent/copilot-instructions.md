## wingetagent

> A **safe, reviewed winget update flow**: scan for upgrades, score each for safety, research the risky ones, and emit a dated HTML report + a curated self-elevating `apply-updates.cmd`. **It never installs anything automatically** — the user runs the `.cmd`.

# WingetAgent — agent guide

A **safe, reviewed winget update flow**: scan for upgrades, score each for safety, research the risky ones, and emit a dated HTML report + a curated self-elevating `apply-updates.cmd`. **It never installs anything automatically** — the user runs the `.cmd`.

## Architecture

Two layers that meet **only through JSON files** in a run directory (`~/.winget-agent/runs/<timestamp>/` by default). Keep this boundary — it's what keeps both sides simple and independently testable.

```
skill (.claude/skills/safe-winget-update)     C# engine (src/WingetAgent, .NET 10 + Spectre.Console)
─ orchestrates + judges                       ─ deterministic, no Claude needed
  scan ───────────────────────────────────►   winget upgrade → enrich → score → updates.json (+ baseline report/cmd)
  read updates.json
  web-search known issues for exact versions
  write annotations.json
  build-report ───────────────────────────►   merge annotations → report.html + apply-updates.cmd
  summarize
```

- `updates.json` — engine output: raw winget fields + derived `category`/`jump`/`publisher`/`trustedPublisher` + enrichment (`releaseDate`, `ageDays`, `recentVersions`, `manifestUrl`) + `score` with a `factors` breakdown.
- `annotations.json` — Claude's per-package judgment: `id`, `adjustedScore?`, `recommendation` (`Approve`|`Review`|`Skip`), `notes`, `sources[]`.

The engine is fully usable standalone (`scan` writes a baseline report/cmd with no annotations). The skill adds researched judgment on top.

## Ubiquitous Language

`UBIQUITOUS_LANGUAGE.md` (repo root) is the canonical domain glossary — the agreed vocabulary for the Scan workflow, Update assessment, and agentic review. Use these terms in code, comments, and the Report; consult its "Flagged ambiguities" before naming new concepts (notably: **Review** means three different things — Band vs Recommendation vs the user's act; **Update** is the domain noun, "upgrade" only quotes winget; **Safety Score** vs **Adjusted Score**). Update the glossary when introducing or renaming a domain concept.

## Commands

Requires the **.NET 10 SDK**. First `dotnet run` auto-restores + builds.

```powershell
dotnet run --project src/WingetAgent -c Release -- scan                 # enumerate + enrich + score
dotnet run --project src/WingetAgent -c Release -- scan --no-enrich     # skip GitHub (fast, no dates)
dotnet run --project src/WingetAgent -c Release -- scan --max-enrich 5  # cap enrichment (rate limits)
dotnet run --project src/WingetAgent -c Release -- build-report -i <rundir>
dotnet run --project src/WingetAgent -c Release -- token                # show / set up GitHub token
dotnet run --project src/WingetAgent -c Release -- open                 # open latest report (--folder for Explorer)
```

Runs are written to `~/.winget-agent/runs/<timestamp>/` by default (override with `scan -o`).

## Code map

| Area | Files |
|------|-------|
| Entry / CLI | `Program.cs`, `Commands/{ScanCommand,BuildReportCommand,TokenCommand,OpenCommand}.cs` |
| Winget | `Winget/WingetClient.cs` (CLI shell-out + column-offset table parser) |
| Enrichment | `Enrichment/WingetPkgsClient.cs` (GitHub manifests), `VersionAnalyzer.cs` (semver jump), `CategoryClassifier.cs` |
| Scoring | `Scoring/SafetyScorer.cs` (deterministic 0–100 + factors) |
| Baseline | `Baseline/BaselineComparer.cs` (run-over-run diff: New/Updated/Pending + FirstSeen→PendingDays) |
| Output | `Output/{HtmlReportWriter,CmdWriter,JsonIo,ReportRenderer}.cs` |
| Config / paths | `GitHub/GitHubToken.cs` (user-secrets + env token), `Paths.cs` (run directory location) |
| Models | `Models/Models.cs` |

## Invariants & gotchas (learned the hard way)

- **Quote AND validate every emitted `--id`/`--version`.** winget version strings can contain spaces/parens (e.g. `Zoom.Zoom.EXE` → `7.0.5 (38856)`), so both args are quoted. `CmdWriter.IsSafeArg` additionally rejects `" % \r \n` and emits a `BLOCKED` (commented) line — defense against command injection from a hostile source (STRIDE T1). Don't remove either.
- **`apply-updates.cmd` is ASCII-only.** Batch consoles default to an OEM code page — no em dashes, arrows, or other non-ASCII.
- **The apply script is self-labeling and tallies failures.** Each enabled install gets an `echo [n/N] id  cur -> new` banner plus an `if errorlevel 1` line that prints `OK`/`** FAILED` and increments `%FAIL%`; a summary prints at the end. winget emits *no* package banner when an upgrade fails (e.g. "No applicable installer found"), so this is how a failure gets attributed. Banner text echoes id/version **unquoted**, so `CmdWriter.EchoSafe` escapes batch metacharacters (`& < > | ^ ( )`) — keep it paired with `IsSafeArg` (T1).
- **Redirected winget output is wide and untruncated.** Because stdout is redirected, `winget upgrade` emits the full table (no `…`), so the header column-offset parser is safe. Don't "fix" it to handle ellipsis.
- **Token resolution.** `GitHubToken.Resolve()` prefers .NET user-secrets (`GitHub:Token`) over the `GITHUB_TOKEN` env var; the recommended token has **no scopes** (public read). Never log it or write it to an artifact.
- **GitHub rate limit.** Enrichment is ~2 API calls/package. Unauthenticated = 60/hr → engine caps at 25 packages and warns (pointing at `wingetagent token`). Enrichment degrades to "age unknown", never fails.
- **Never run `apply-updates.cmd` yourself.** Applying updates is the user's explicit action. The skill generates and explains; the user runs.
- **Safety scoring** starts at 70; factors: age, version jump (patch +10 / minor 0 / major −20 / unknown −8), category (driver −20 / system −15 / dev −5 / browser +5), publisher trust (±), unknown installed version −10, baseline (new/changed → −5 *only when release date unknown*, to avoid double-counting age; pending across runs → up to +8 maturity). Bands: Safe ≥75, Review ≥50, Risky <50.
- **Baseline = the previous run in the same parent folder.** `BaselineComparer` runs before scoring and compares against the most recent sibling run (so `-o` runs chain within their own folder, default runs within `~/.winget-agent/runs/`). `FirstSeen` is carried forward so `PendingDays` reflects true pending-age; missing `FirstSeen` on older runs falls back to that run's date. No prior run → everything `NoBaseline`, nothing penalized.

## Conventions

- **Conventional Commits** (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, …).
- Keep changes surgical; match surrounding style. The engine must stay runnable without Claude.
- Runs live under `~/.winget-agent/runs/` (outside the repo, so the shared MIT repo stays clean) — they're the local update history.
- A COM-API winget backend (`Microsoft.Management.Deployment`) is the documented hardening path if the CLI parser ever proves fragile; intentionally not in v1 to stay small.

## Security Documentation

### STRIDE.md Threat Model

This repository includes a STRIDE threat model (`STRIDE.md`) for security analysis.

**When to update STRIDE.md:**
- Adding new authentication/authorization mechanisms
- Changing data storage, encryption, or secrets handling
- Adding new external integrations or API endpoints
- Modifying trust boundaries (new external connections, database access)
- After security incidents or penetration test findings
- When addressing security recommendations from the document

**How to update:**
1. Add new threats to the relevant STRIDE category (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
2. Assess likelihood (Very Low → High) and impact (Low → Critical)
3. Document existing mitigations or add recommendations
4. Link GitHub issues for unresolved findings
5. Update the Review History table
6. Update version if using frontmatter

**Tracking critical findings:**
- Critical/High risk findings should have a linked GitHub issue with `security` label
- Review STRIDE.md annually or after major releases

---
> Source: [stevehansen/WingetAgent](https://github.com/stevehansen/WingetAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
