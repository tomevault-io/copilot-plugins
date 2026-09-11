## llm-dungeon

> This repository contains the replacement application authorized in the design conversation. On 2026-09-07 the user authorized replacing the tracked contents of `TJurijs/llm-dungeon` with this project, preserving Git history. The separate old local checkout and its uncommitted work remain a read-only reference; do not inherit its maintenance constraints as this game's design.

# LLM Dungeon Reloaded

This repository contains the replacement application authorized in the design conversation. On 2026-09-07 the user authorized replacing the tracked contents of `TJurijs/llm-dungeon` with this project, preserving Git history. The separate old local checkout and its uncommitted work remain a read-only reference; do not inherit its maintenance constraints as this game's design.

## Agreed principles

- Open-ended play guided by setting, scenario, character, and established history. Invent missing material during play; preserve established facts and meaningful consequences.
- Genuine d100 only for meaningful uncertainty; possibility precedes numeric odds. Warn about foreseeable meaningful risk before commitment. Severe consequences remain possible.
- Simple living character facts, persistent items and NPC relationships, timed commitments, ammunition counts, otherwise light logistics.
- English/Russian UI and play. Gemini `gemini-3.8-flash` is the exact default. Custom model IDs and major-provider connections have explicit compatibility checks, not gameplay-quality certification.
- In-chat appeals can correct errors and reasonably undo affected actions, preserving evidence. Technical recovery is not an appeal and must not reroll or duplicate a result.
- Current truth, recoverable history, and compact working context have distinct responsibilities. Compaction must not erase original evidence or change world state.
- Improve generic behavior and root causes. Keep incidents as test cases rather than adding fictional noun-specific rules or endless prompt reminders.

## Engineering

Node >=22.13 ESM with built-in SQLite, HTTP, fetch, and test runner. Vanilla browser UI reuses the original visual foundation. Core engine owns gameplay; HTTP and playtests use the same boundary. SQLite records snapshots, requests, and history. ProviderService owns a durable global spending ledger. Detailed rationale is in docs/design.

No secrets in source, browser payloads, prompts, reports, or logs. Runtime may load `.env` privately after the user's implementation/test authorization. Never print its contents. External model calls across compatibility, setup, gameplay, compaction, appeals, and tests share the durable authorized ceiling; preserve reservations for unaccounted requests and never auto-top-up. On 2026-09-06 the user explicitly authorized an additional USD 20, then approved proceeding with non-batch cost optimization and the agreed readiness tests. Record this extension once with an idempotent authorization; preserve the original USD 20 history. This new round may add at most USD 20 liability beyond its recorded start, including at most USD 2 for the initial cost benchmark. Use runtime spending ceilings as well as the durable ledger. No paid 24-hour batch jobs. Readiness requires fresh EN/RU evidence; budget exhaustion is not a pass.

The user subsequently authorized EUR 30 more and said to proceed sparingly. `tools/authorize-playable-budget.mjs` records this once as authorization `playable-eur30-2026-09-06`, using a conservative USD 34 allowance (ECB 2026-09-04: USD 1.1622 per EUR). Combined incremental allowance from the original readiness baseline is USD 54, with overall runtime ceiling USD 71.996466; preserve every earlier request and unknown reserve. Individual phases can retain smaller ceilings. The additional allowance is a cap, not a target to spend.

Updated acceptance: seek dependable, enjoyable play with rare repairable mistakes and working appeals. Harmless wording and an isolated recoverable slip do not invalidate a whole game or require an expensive restart. Fix recurring or fundamental failures. Assess frequency, severity and repairability, plus pacing and cost. No 1,000-turn test or arbitrary minimum turn/roll count is required. Exercise actual memory compaction and fresh EN/RU games, but target useful coverage and diminishing returns rather than perfection. Routine automatic actions have no dice badge; meaningful calculated checks show the base, labeled modifiers, raw target, clamped chance and result, including guaranteed endpoints without a die.

Budget-efficient development: default to Codex reviewing saved transcripts, state changes and existing provider results, writing reports, and choosing test-player actions in the current task. These activities need no additional game-provider calls. Use offline fixtures and deterministic checks first. Reserve standalone model diagnostics for a concrete unresolved hypothesis about the target model's behavior, with a small explicit call/cost cap and reusable evidence. Paid API player automation is optional and must not be the default for manual supervised playtests. Keep production consistency checks enabled when measuring the playable product; distinguish their cost from development-only analysis. All actual gameplay calls and necessary live probes still use the existing durable ceiling and accounting.

Run `npm test` and `npm run check` after changes, and appropriate browser and live tests. Report failed/inconclusive runs honestly; no test count or model judge establishes universal consistency. Do not claim a synthetic long history proves a real 1,000-turn campaign.

## Explicit replacements for old maintenance assumptions

The user explicitly requested a testing viewer for fictional DM secrets and hidden template notes on 2026-09-06. The no-secret-browser-payload rule above continues to protect credentials and ordinary gameplay responses. The deliberate exception is the allowlisted, read-only spoiler endpoint, called only after the user confirms the UI warning; close/navigation clears displayed private content and reopening requires confirmation. Never expose raw provider results, pending alternative outcomes, credentials or diagnostic logs through that viewer.

This app allows ongoing invention, bounded history lookup, NPC activity over game time, appeals with correction/undo, and SQLite persistence. It does not preserve the earlier app's authored-capability-only, fixed-two-generation, non-agentic/no-offscreen-activity, Markdown-authority, or forward-only-save restrictions. The GitHub source replacement does not authorize deleting or migrating old local saves or uncommitted work. A fresh clone is not authorization to spend on provider calls or replay historical budget extensions.

---
> Source: [TJurijs/llm-dungeon](https://github.com/TJurijs/llm-dungeon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
