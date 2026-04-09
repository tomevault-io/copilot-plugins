## devvalue

> VS Code Extension. Not a web app. No express/fastify.

# DevValue — The VibeCoder's Cognitive & Value Tracker

VS Code Extension. Not a web app. No express/fastify.

## WHAT THIS DOES
Tracks the TRUE cost of AI-era software engineering per git branch:
Cost = (Human Focus Time × Hourly Rate) + (AI Agent Token Cost)

"Active time" isn't just typing — it's reading, debugging, reviewing AI output, running tests.

## KEY IDEAS
- **PTA Flow Detection**: If a developer is rapidly switching files, reading logs, running builds — they're in flow, not idle. Dynamically extend the AFK timeout (5min → 20min) instead of stopping the clock.
- **Token Sniffing**: Parse Claude Code's local JSONL logs (~/.claude/projects/) to capture token costs per branch. Distinguish direct user interactions from background jobs (context compaction, summarization). Design the sniffer interface so JSONL parsing can be swapped for an OpenTelemetry (OTLP) listener later. Stream-parse logs — never load entire files into memory (Sonnet 4.6 logs can be very large).
- **Branch = Business Context**: All data keyed by git branch name. A feature branch's total cost tells you what that feature really cost to build.

## ARCHITECTURE CONSTRAINT
Core logic (algorithms, parsers, calculators) must be pure TypeScript with NO `vscode` imports — enabling future reuse in JetBrains, CLI, or other IDEs. VS Code-specific code goes in a separate adapter layer.

## EXTENSION SETTINGS (these names are the contract — do not rename)
- devvalue.hourlyRate (number, default 75)
- devvalue.maxIdleTimeout (number, default 300)
- devvalue.flowThreshold (number, default 3 events/min)
- devvalue.claudeLogGlob (string, default ~/.claude/projects/*/session_logs/*.jsonl)
- devvalue.enableStatusBar (boolean, default true)

## EXTENSION COMMANDS
- devvalue.openDashboard
- devvalue.startTracking / devvalue.stopTracking
- devvalue.resetBranch
- devvalue.exportData

## STATUS BAR FORMAT
`$(clock) 2h 15m | $(branch) main | $148.50` — click opens dashboard

## WEBVIEW ↔ EXTENSION MESSAGING
Extension sends: { type: 'update', sessions, currentBranch, config }
Webview sends: { type: 'setRate', hourlyRate } | { type: 'resetBranch', branchName } | { type: 'requestUpdate' }

## BUILD
pnpm, esbuild, TypeScript strict. `pnpm run compile` to build. Fn+F5 to test.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChrisZHHG)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ChrisZHHG)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
