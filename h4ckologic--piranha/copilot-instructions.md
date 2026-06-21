## piranha

> This project is a Claude Code skill for autonomous bug bounty hunting.

# BugHunter AI — Claude Code Configuration

This project is a Claude Code skill for autonomous bug bounty hunting.

## Key Files
- `skills/BugBountyFramework/SKILL.md` — Main skill definition (loaded by Claude Code)
- `cli/piranha.ts` — the `piranha` launcher binary (compiled via `bun build --compile`). Classifies a target, drives the hunt-orchestrator state machine, computes the router plan, and launches the harness (omp/pi or Claude Code). Standalone subcommands: `hunt`, `status`, `plan`, `agents`, `engagements`, `tools` (per-domain dynamic tooling/MCP capture + PATH check), `vault`, `install`, `doctor`, `completions`, `update`.
- `skills/BugBountyFramework/Agents/*.md` — 53 specialized agents (51 hunters/specialists + ValidatorAgent + ExploitChainAgent)
- `skills/BugBountyFramework/Tools/*.ts` — TypeScript tools (Bun runtime)
- `skills/BugBountyFramework/Tools/agent-router.ts` — engagement type → ordered, dependency-aware agent deployment plan
- `package.json` — Pi package manifest (`pi.skills`); this repo is the **PiRanha** Pi package and installs the skill as `piranha` via `pi install git:github.com/h4ckologic/PiRanha`

## How It Works
When a user types `hunt <target>`, Claude Code loads SKILL.md which orchestrates a 10-phase hunt:
1. State machine initialization (hunt-orchestrator.ts)
2. Credential loading from vault (credential-vault.ts)
3. Authentication flow (auth-manager.ts)
4. Application profiling via Playwright
5. Reconnaissance
6. Parallel agent deployment (each agent is a .md file)
7. Dynamic testing via Burp + Playwright
8. Vulnerability assessment
9. Learning & pattern update
10. Report generation

## Development
- Tools are TypeScript/Bun. Run with `bun <tool>.ts --help`
- Binary: `bun run build` (this host) or `bun run build:all` (cross-platform → `dist/piranha-<platform>-<arch>`, attached to GitHub releases by `.github/workflows/release.yml`). Install via `curl -fsSL .../install.sh | sh`.
- The three core tools double as importable libraries (guarded by `import.meta.main`) and standalone CLIs — `cli/piranha.ts` bundles them; `bun <tool>.ts` still runs each directly.
- Agents are Markdown files with structured instructions
- Test against your own targets or sanctioned bug bounty programs only

---
> Source: [h4ckologic/PiRanha](https://github.com/h4ckologic/PiRanha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
