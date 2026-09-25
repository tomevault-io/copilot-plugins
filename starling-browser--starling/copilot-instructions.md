## starling

> The repository's agent rules of engagement live in [AGENTS.md](AGENTS.md) — read it first.

# Agent instructions

The repository's agent rules of engagement live in [AGENTS.md](AGENTS.md) — read it first.

The **C# performance policy** is the source of truth in AGENTS.md under
"Coding standards — performance policy for C# code". It applies to all code in
this repo. Follow it for every change.

**Comments and doc comments:** Use them sparingly. A comment should explain a
decision local to the scope of the code when that decision isn't clear from the
code itself. Skip comments that restate what the code already says. Keep them
local too — never send the reader three layers up or into another system, and do
not narrate a problem you hit (no "fixes the issue where..."). See AGENTS.md
under "Coding standards" for the full rule.

**Braces:** Always use braces for conditionals, loops, and other block
statements, even when the body is a single line.

**Testing a demo site:** when asked to test, verify, or check a demo site (the
fixtures in `testdata/sites/...`), always load it through the running Aspire
harness at `http://localhost:8088/<name>/`, never by rendering the files from
disk (`file://`, a loopback, or an in-process stub). Rendering from disk hides
real serving failures — most notably the `sites` YARP container not running when
Docker is down. See AGENTS.md under "Getting traces & telemetry from Aspire" for
the rule and the quick checks when a site will not load.

---
> Source: [starling-browser/starling](https://github.com/starling-browser/starling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
