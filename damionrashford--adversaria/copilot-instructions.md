## adversaria

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Adversaria is a **Claude Code plugin**, not an application. There is no build step, no bundler, and no dependency manifest. Every file is one of three things: a JSON manifest, a Markdown instruction file that Claude reads at runtime, or a TypeScript file run directly by Bun.

The consequence: **the Markdown *is* the product.** Editing `agents/*.md` or `skills/*/SKILL.md` changes behavior as directly as editing code. Treat prose changes in those files with the same care as a code change, and re-read the frontmatter contract below before touching it.

The plugin is distributed through a public marketplace and runs on other people's machines, inside their projects. Nothing in this repo may hard-code an absolute path, assume a particular operating system or directory layout, or carry information specific to one contributor. Paths resolve through `${CLAUDE_PLUGIN_ROOT}` and `${CLAUDE_SKILL_DIR}` (see *Path contract*).

## Development tooling

[.claude/settings.json](.claude/settings.json) enables `plugin-dev`, Anthropic's official plugin-authoring toolkit, for anyone working in this repo. It is the authority on plugin structure — prefer it over recalling conventions from memory, since the plugin spec changes.

It loads automatically once the official marketplace is known locally. If Claude Code reports the marketplace is missing:

```
/plugin marketplace add anthropics/claude-plugins-official
```

| Component | Use it when |
| :--- | :--- |
| `plugin-dev:plugin-validator` (agent) | Before any commit that touches `.claude-plugin/`, `hooks/hooks.json`, or agent/skill frontmatter. Validates manifests and file layout. |
| `plugin-dev:skill-reviewer` (agent) | After editing any `SKILL.md`, especially the `description` — that field is what makes the skill auto-trigger. |
| `plugin-dev:agent-creator` (agent) | Adding a fourth agent, or reworking an existing agent's frontmatter. |
| `/plugin-dev:create-plugin` (command) | Scaffolding only. This plugin already exists; rarely needed here. |
| `plugin-dev:plugin-structure` (skill) | Questions about directory layout and manifest fields. |
| `plugin-dev:skill-development` (skill) | Writing or restructuring a skill, including `references/` and progressive disclosure. |
| `plugin-dev:agent-development` (skill) | Agent frontmatter fields: `model`, `tools`, `skills`, `memory`, `effort`, `maxTurns`. |
| `plugin-dev:hook-development` (skill) | Anything in `hooks/` — event names, matchers, exit-code contract, `${CLAUDE_PLUGIN_ROOT}`. |
| `plugin-dev:command-development` (skill) | Slash-command surface and `argument-hint`. |
| `plugin-dev:mcp-integration` (skill) | Only if an MCP server is ever added. There is none today. |
| `plugin-dev:plugin-settings` (skill) | `settings.json` keys and plugin enablement scopes. |

`plugin-dev` ships **no tools and no MCP server** — it contributes instructions only, so it cannot break this plugin at runtime.

For Claude Code's own behavior (hook events, settings keys, plugin loading, agent frontmatter), check the official docs at https://code.claude.com/docs rather than asserting harness behavior from memory. These surfaces change between releases, and a stale assumption here ships to every installed user.

## Commands

Run the reasoning engines directly to test them, exactly as the skills do:

```bash
bun scripts/argument-graph.ts add "cost-reduction" claim --obs "cuts spend 40%" --agent advocate
bun scripts/argument-graph.ts add "vendor-lockin" weakness --obs "single-provider dependency" --agent adversary
bun scripts/argument-graph.ts relate "vendor-lockin" undermines "cost-reduction" --agent adversary
bun scripts/argument-graph.ts lock      # seal the record (appeal); reset + edits to prior testimony refuse
bun scripts/argument-graph.ts unlock
bun scripts/argument-graph.ts analyze
bun scripts/argument-graph.ts verdict
bun scripts/argument-graph.ts export
bun scripts/argument-graph.ts transcript
bun scripts/argument-graph.ts show
bun scripts/argument-graph.ts reset
```

```bash
bun scripts/thought-chain.ts init "the subject" --total 7
bun scripts/thought-chain.ts think 1 "<content>"
bun scripts/thought-chain.ts revise 1 "<content>" --reason "<why>"
bun scripts/thought-chain.ts branch "<id>" 1 2 "<content>"
bun scripts/thought-chain.ts status
bun scripts/thought-chain.ts render
bun scripts/thought-chain.ts reset
```

```bash
bun scripts/transcript.ts serve --port 7391 --no-open
```

Run the tests — this is the one place a change can be verified automatically:

```bash
bun test
```

**No dependency manifest, ever.** Bun runs `.ts` directly and `bun test` works with no `package.json`, no `node_modules`, and no lockfile. Adding any of them breaks skill loading. There are no third-party imports anywhere in this repo, and there must not be.

Load the working tree as a plugin to exercise agents, skills, and hooks end to end:

```bash
claude --plugin-dir /path/to/this/repo
```

Verify what actually loaded, and inspect enablement scopes:

```bash
claude plugin list
```

`bun test` covers the semantics — labelling, cycles, and record parsing. It does **not** cover the skills, agents, or hooks, which are prose and shell-level behavior. So after `bun test` passes, still load the plugin with `--plugin-dir` and drive one agent (`@adversary`, `@advocate`, `@judge`) through a real analysis. A change that only compiles is not verified.

## Architecture

Three layers, each with one job. Understanding the boundary between them is the thing that requires reading multiple files.

**Layer 1 — Agents** ([agents/](agents/)). Three personas with distinct epistemic stances: `advocate` argues for, `adversary` argues against, `judge` rules on what survives. Each agent's frontmatter binds it to exactly one skill via a `skills:` entry. Agents own *stance and delegation*; they do not own workflow.

**Layer 2 — Skills** ([skills/](skills/)). Five skills define the actual workflows: `steelman`, `crucible`, `verdict`, `devils-advocate`, and `appeal`. The first three map 1:1 onto the three agents. `devils-advocate` is the orchestrator that runs all three phases in sequence as a seven-step thought chain. `appeal` is the Roman Rota second panel — it reviews a standing verdict without a fourth agent, since a fresh subagent context already satisfies the different-panel requirement. Skills own *workflow and graph writes*.

**Layer 3 — Engines** ([scripts/](scripts/)). Three dependency-free TypeScript CLIs that hold all state. [argument-graph.ts](scripts/argument-graph.ts) is a typed graph (7 entity types × 7 relation types) that decides outcomes by **grounded argumentation semantics**, not by counting. [thought-chain.ts](scripts/thought-chain.ts) tracks numbered reasoning steps with revision and branching. [transcript.ts](scripts/transcript.ts) serves the live WebSocket UI.

### The verdict is a fixpoint, not a tally

This is the part worth understanding before changing anything in the engine. Every node is labelled `IN` (all attackers are `OUT`), `OUT` (some attacker is `IN`), or `UNDECIDED`, iterated to convergence in `groundedLabelling`. **An attack from a defeated node does not count** — which is the only reason a `counter` can rescue a claim the adversary attacked. Attacks are `undermines`, `mitigates`, and `contradicts` (bidirectional); dependencies are `assumes`, `depends_on`, and `if_fails`. Cycles come out of `findCycles` (Tarjan SCC) as `circular_reasoning`.

Change the attack/dependency sets and you change every verdict the plugin will ever render. [references/argument-semantics.md](references/argument-semantics.md) is the user-facing explanation and must stay in sync with the code.

### Shared state is the coupling

Agents do not talk to each other. They compose **only** through the argument graph, which persists as JSONL between runs: the advocate writes claims and evidence, the adversary writes weaknesses and counters against those same node names, and the judge reads the resulting topology. The verdict is derived from graph structure, not from re-reading the conversation.

Graph location is `$DA_GRAPH_PATH` if set, otherwise `<tmpdir>/adversaria/<cwd-basename>-<cwd-hash>-graph.jsonl` — scoped per project so two concurrent analyses don't collide. The chain is the same shape via `$DA_CHAIN_PATH`. Always set `DA_GRAPH_PATH` when testing so you don't corrupt a live session's state.

Two implications when changing anything:

- **Entity names are the join key.** The adversary can only attack a claim by naming it exactly as the advocate registered it. Renaming conventions in one skill silently breaks composition with the others.
- **Node and relation types are a closed vocabulary** — the `ENTITY_TYPES` and `RELATION_TYPES` const arrays in [scripts/argument-graph.ts](scripts/argument-graph.ts), which derive the union types the compiler enforces. Adding a type means updating the array, the attack/dependency sets if it participates in semantics, every skill that emits it, the hook validator, and the README table together.
- **Provenance is load-bearing.** Every record carries `agent` and a monotonic `seq`. The transcript UI and the judge's citations both read them, so a write that omits `--agent` silently becomes `unknown`.

### Path contract

`${CLAUDE_PLUGIN_ROOT}` resolves to the plugin root at runtime. Every path in every skill, agent, and hook goes through it — `${CLAUDE_PLUGIN_ROOT}/scripts/`, `${CLAUDE_PLUGIN_ROOT}/references/`, `${CLAUDE_PLUGIN_ROOT}/hooks/scripts/`.

Never write a relative path or an absolute path in a skill, agent, or hook. `${CLAUDE_SKILL_DIR}` (the invoking skill's own directory) is deliberately unused here: the traditions are shared across skills, so they live at the plugin root instead of being copied per skill.

### Hooks

[hooks/hooks.json](hooks/hooks.json) wires five lifecycle points:

| Event | Matcher | Script | Effect |
| :--- | :--- | :--- | :--- |
| `SessionStart` | `startup\|resume` | `session-restore.ts` | Injects a summary of prior saved graph/chain state as context |
| `PreToolUse` | `Bash` | `graph-validate.ts` | Blocks malformed `argument-graph.ts` calls with **exit 2** |
| `PostToolUse` | `Bash` | `auto-analyze.ts` | Async graph analysis after graph writes |
| `SubagentStop` | `advocate\|adversary` | `agent-summary.ts` | Captures findings when an arguing agent finishes |
| `Stop` | — | `session-save.ts` | Async persist of graph and chain to the data dir |

`graph-validate.ts` **enforces kebab-case entity names** and rejects invalid types by exiting 2. If a skill's example commands use `"Cost Reduction"` instead of `"cost-reduction"`, the hook will block the agent mid-run. Keep skill examples consistent with what the validator accepts.

Hook scripts are TypeScript. `hooks.json` invokes them as `bun "${CLAUDE_PLUGIN_ROOT}/..."` with an explicit interpreter rather than relying on the executable bit, which does not survive every distribution path. They keep the `#!/usr/bin/env bun` shebang so they stay directly runnable too. They read the JSON event from stdin, import the engine's own `graphPath()` rather than guessing at a location, and must stay executable (`chmod +x`). They persist to `${CLAUDE_PLUGIN_DATA:-/tmp/adversaria-data}`.

### References and progressive disclosure

The 10 reasoning traditions live as Markdown in [references/](references/) at the plugin root — **one copy each, shared by all five skills** — alongside [argument-semantics.md](references/argument-semantics.md), the user-facing explanation of how labelling decides a verdict. Skills read them on demand rather than inlining them, which keeps each `SKILL.md` short. Each skill loads only the traditions it uses; `devils-advocate` loads all 10 across its phases.

They were previously duplicated into each skill's own `references/`, which meant editing a tradition in two places. Don't reintroduce that: add a tradition once, in `references/`, and point at it with `${CLAUDE_PLUGIN_ROOT}/references/<name>.md`.

Every reference read resolves — verify after any move:

```bash
for f in $(grep -rho 'references/[a-z-]*\.md' skills/ | sort -u); do test -f "$f" || echo "MISSING $f"; done
```

## Releasing

Changes reach installed users through the marketplace, so a commit here is a release. Before committing:

- Keep [.claude-plugin/plugin.json](.claude-plugin/plugin.json) and [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) in agreement. `version` appears in **both** and must match — bump them together, and check with:

  ```bash
  python3 -c "import json;a=json.load(open('.claude-plugin/plugin.json'))['version'];b=json.load(open('.claude-plugin/marketplace.json'))['plugins'][0]['version'];print('MATCH' if a==b else f'DRIFT {a} vs {b}')"
  ```
- The README's structure diagram and its skills, agents, and traditions tables are the user-facing documentation. Update them in the same commit as the change they describe, or they go stale for everyone who installs.
- Agent and skill `description` fields are the auto-trigger surface: they decide whether Claude delegates to this plugin at all. A vague description causes silent non-triggering; an over-broad one hijacks unrelated requests. Run `plugin-dev:skill-reviewer` on any description change.

---
> Source: [damionrashford/Adversaria](https://github.com/damionrashford/Adversaria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
