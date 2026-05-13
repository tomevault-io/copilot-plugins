## self

> This folder contains the standalone Self Context Standard v0.5 proposal site and starter examples.

# Self Context Standard - Agent Instructions

This folder contains the standalone Self Context Standard v0.5 proposal site and starter examples.

## Critical Rule: Self-Updating Documentation

Keep this file current. Whenever an agent introduces a new module, changes an existing module's purpose, adds or removes dependencies, or changes the development workflow, update this file as part of the same work.

## Project Direction

Self is a portable `self/` folder standard for agent runtime context, with progressive disclosure as the first principle. The proposal site must stay usable from disk and avoid framework or bundler assumptions.

The primary deployment target is a downloaded `self/` folder used directly inside Claude or Codex Skills-style environments. Optimize the markdown instructions for those harnesses first. Local scripts and the static proposal site are helpful maintainer tooling, but the portable markdown files must remain sufficient on their own when scripts are unavailable.

## Runtime Worlds

Self has three runtime worlds that must stay distinct:

* **Consumer uploaded Skill world**: The user downloads or uploads `self/` as a Skill. `SKILL.md` is the activation surface, `SELF.md` is the routing entrypoint, and markdown files are the portable runtime truth. Do not assume scripts, `manifest.json`, or repo tooling will run automatically.
* **Shell-enabled advanced Skill world**: The Skill is running in an environment that exposes the `self/` files plus Bash and Node.js. In this world, natural-language maintenance requests such as "make Self notice this next time," "make this easier for Claude or Codex to find," "check Self is up to date," "scan Self after this change," "keep the skill in sync," "update the package map," or "check that this still validates" may route to `node self/scripts/sync-self.js` and `node self/scripts/validate-self.js`. These helpers mirror markdown trigger truth into `manifest.json`; they do not make `manifest.json` the source of truth.
* **Repo maintainer/developer world**: Work inside this proposal repo uses `scripts/sync-self-spec.js` and `scripts/validate-self.js` to keep canonical examples, bridge copies, the homepage starter bundle, and the setup wizard base package aligned.

## Current Modules

* **`index.html`**: Static proposal website shell with anchor sections for the standard, manifest, explorer, disclosure flow, validation, common questions, install flow, developer integration notes, and references, plus a Mintlify-style page contextual menu (`Copy page` split button), a GitHub repo + star badge, and a hero `self/` promo block that can toggle between grid and tree file displays.
* **`styles.css`**: Design tokens, layout, responsive behavior, cards, install/developer sections, the compact common-question carousel with left-side tabs and active/hover/waiting states, tool-call simulation styling, code presentation, contextual-menu/dropdown styling, GitHub badge styling, hero folder view-toggle styling, and hover/focus heading-anchor affordances.
* **`app.js`**: Classic non-module browser script. Embeds starter files, exposes `window.SelfStarter` for static subpages, renders the schema explorer, renders install bridge snippets with an OpenAI/Anthropic toggle, validates examples, powers copy/download actions, simulates progressive-disclosure tool calls for common questions, renders the tabbed question carousel and keeps its active/waiting states in sync, wires the contextual menu actions with working page-copy/view/share behavior, renders inline current-color contextual-menu logos, includes hosted-only MCP/Cursor/VS Code install entries in that menu, fetches live GitHub stars for the header repo tracker, adds heading anchor-link copy behavior on larger screens, and drives the hero `self/` grid/tree toggle behavior.
* **`setup/`**: Static consumer setup wizard subpage. `setup/index.html` loads the shared stylesheet, `app.js`, and `setup/setup.js`; `setup/setup.js` uses `window.SelfStarter` to customize a Skill-ready `self/` ZIP locally in the browser with deterministic markdown generation, ordered final previews, explicit unlock-to-edit guards for generated/router/template files, a first-use prompt for post-upload onboarding, live SKILL.md/manifest validation, template-only `security.md`, and starter-only `chrono/`.
* **`assets/context-menu/`**: Downloaded SVG logo fallbacks retained for local reference; the live contextual menu currently uses inline SVGs in `app.js` so the logos inherit the menu icon color.
* **`LICENSE`**: MIT license for the proposal site and starter materials.
* **`.gitignore`**: Keeps local macOS filesystem metadata out of the repository.
* **`llms.txt`**: Agent-readable project summary with links to the proposal, starter example, manifest, validator, and license.
* **`examples/self/`**: Canonical starter Self folder. Includes `SELF.md` as the Self-native entrypoint, `SKILL.md` as the Agent Skills-compatible activation wrapper, Skills-style YAML frontmatter in Markdown files, slot-level `trigger_domains` and `private_trigger_domains` metadata for recursive trigger maintenance, optional portable `self/scripts/sync-self.js` and `self/scripts/validate-self.js` helpers for shell-enabled environments, dedicated Claude/Codex desktop quit helpers for macOS and Windows under `self/scripts/`, `skills/first-use.md` for post-upload onboarding and project instruction wiring, `manifest.json` as a machine-readable mirror for validators and apps, and `metadata.json` as a minimal Self system diagnostic stamp. The markdown files must remain usable as a standalone uploaded skill bundle even when helper scripts are not available.
* **`scripts/validate-self.js`**: Node-based Self validator for project-specific rules that the upstream Agent Skills validator does not check, including markdown-first trigger coherence, SKILL.md frontmatter and size limits, private context disclosure defaults, and embedded starter-file sync.
* **`scripts/sync-self-spec.js`**: Node-based sync script that reads the markdown trigger contract from `SELF.md` and slot frontmatter, regenerates the mirrored Claude/Codex skill wrappers, refreshes the synced `SELF.md` purpose summary, mirrors trigger data into `manifest.json`, and rebuilds the embedded `starterFiles` object in `app.js`.
* **`examples/codex/`**: Codex bridge templates, including `AGENTS.md` and optional `.agents/skills/self/SKILL.md`.
* **`examples/claude/`**: Claude Code bridge templates, including `CLAUDE.md`, optional `.claude/commands/self.md`, and optional `.claude/skills/self/SKILL.md`.
* **`README.md`**: Short local usage notes.

## Development Workflow

Open `index.html` or `setup/index.html` directly in a browser. Do not require a dev server.

Keep the site vanilla:

```sh
# No server required
open /Users/ali/Code/self/index.html
open /Users/ali/Code/self/setup/index.html
```

Constraints:

* No Next.js, Vite, router, server, ES modules, or React-style app structure. Keep the site as plain HTML, CSS, and classic browser JavaScript.
* Runtime `fetch` is allowed for optional live client-side features, such as GitHub metadata or contextual-menu integrations, as long as the page still loads from disk and core content does not depend on a server.
* No package dependencies unless the user explicitly changes the static-site constraint.
* If files under `examples/` change, keep the embedded `starterFiles` object in `app.js` aligned so the offline ZIP download, browser validator, and `/setup` wizard base package remain accurate.
* Keep `examples/self/model.md` limited to model identity facts: creator, provider, model name, version or release date, official reference links, known capabilities, and known limits. Put the application, CLI, desktop app, IDE, or web service surface in `harness.md`.
* Keep `examples/self/harness.md` focused on how the user accesses the model and what that interface can expose or do: harness name, provider/operator, interface type, access details, surfaces, capabilities, APIs/endpoints, tool or script routes, permission boundaries, app control behavior, and known harness limits.
* Keep `examples/self/orch.md` focused on workflow coordination across Self, the harness, available models, subagents, tools, scripts, and private context. It may intentionally repeat important harness routes, and it is the right place to log disclosed model or harness strengths, weaknesses, quirks, or recurring habits that affect workflow behavior.
* Keep `examples/self/user.md` private and consent-gated. Store user-stated biography, professional context, preferences, beliefs, habits, routines, decision preferences, workflows, and communication defaults there, and keep inferred or tentative preferences clearly separated from user-stated facts.
* Keep `examples/self/metadata.json` minimal. It is a Self system stamp for troubleshooting, debugging, and diagnostics; it may include Self version, created date, installed date, generator, and maintainer label, but not routing rules, model details, app integration data, changelogs, or private user facts.
* If the Self activation surface changes, edit the markdown trigger contract first: `examples/self/SELF.md` plus the relevant slot file frontmatter. Then run `node scripts/sync-self-spec.js` so `SKILL.md`, `manifest.json`, bridge skill copies, and `app.js` stay in sync.
* If the setup wizard changes generated Self behavior, preserve the same self-updating contract in generated `SELF.md`, generated `SKILL.md`, and generated `manifest.json`: markdown frontmatter is the primary trigger truth and manifest trigger hints mirror it.
* Keep first-use onboarding focused on surveying `model.md`, `harness.md`, `orch.md`, and `user.md`; asking concise grouped questions for unknown fields; writing only after permission; and appending `chrono/` continuity notes only with explicit user approval.
* Keep the setup wizard final preview ordered for consumer comprehension: direct wizard-customized files first, generated routing/Skill files next, and starter templates/helpers last. Generated/router/template files may remain editable, but only behind an explicit unlock guard that explains the risk.
* Keep the plain-language Self maintenance route current in both `examples/self/SELF.md` and generated `SKILL.md`. Users may not know terms like manifest or trigger metadata, so route phrases about making Self notice, find, scan, sync, or validate updated content to the same optional sync/validation helpers when the environment supports them.
* Treat edits inside `examples/self/` as recursive Self work: after changing a Self file, review whether `SKILL.md`, `SELF.md`, or bridge guidance should also change to preserve future discovery in Claude and Codex.
* Keep close-program routing aligned across the portable markdown files and the dedicated quit helpers in `examples/self/scripts/`. The primary portable route is `SELF.md -> harness.md -> specific quit helper`, while `scripts/README.md` acts only as a backup registry. If the quit helper behavior or supported platforms change, update those files together.
* Optimize for markdown-first behavior in uploaded Skills environments. Helper scripts may assist maintainers here, but the portable instructions must not rely on scripts being available or executed.
* Do not treat `manifest.json`, `app.js`, or local scripts as automatically loaded by Claude or Codex during normal skill use. For uploaded skills, `SKILL.md` is the trigger surface and `SELF.md` is the first guaranteed deeper routing file after activation.
* Script assumptions must stay conservative. Claude Code docs explicitly allow skill bundles to include scripts Claude can execute, and Claude Code desktop/web sessions can run Bash when that environment exposes it. OpenAI docs explicitly allow skill scripts when the shell/computer environment is present, but the consumer Codex skills docs do not clearly promise script execution for every skill surface. Treat scripts as optional accelerators, not as the only path for correct behavior.
* Skills portability limits to preserve:
  `SKILL.md` frontmatter must satisfy the Agent Skills spec: `name` 1-64 chars, lowercase letters/numbers/hyphens only, matching the directory name; `description` 1-1024 chars; optional `compatibility` max 500 chars.
  Claude Code uses `description` and optional `when_to_use` for auto-loading; that combined listing text is truncated at 1,536 characters per skill, and long skill descriptions across many installed skills compete within a dynamic global listing budget.
  Agent Skills recommends keeping the main `SKILL.md` under 5,000 tokens and under 500 lines. For Claude Code compaction, only the first 5,000 tokens of each invoked skill are re-attached, with a 25,000-token combined budget across recent skills.
* The starter zip includes the complete starter schema, including `examples/self/user.md`, `examples/self/chrono/`, and `examples/self/security.md`, so new users receive the slots they need. Context disclosure is a separate app/runtime flow and must keep private slot contents, especially `examples/self/user.md` and `examples/self/chrono/`, out of the model context unless the user asks for them.

## Verification

Recommended checks:

```sh
node --check app.js
node --check setup/setup.js
node --check scripts/sync-self-spec.js
node --check scripts/validate-self.js
python3 -m json.tool examples/self/manifest.json >/tmp/self-manifest-check.json
uvx --from skills-ref agentskills validate examples/self
node scripts/sync-self-spec.js
node scripts/validate-self.js
node -e "const fs=require('fs');const t=fs.readFileSync('examples/self/SKILL.md','utf8');const m=t.match(/^---\\n([\\s\\S]*?)\\n---\\n/);const f=m?m[1]:'';const v=k=>((f.match(new RegExp('^'+k+':\\\\s*(.*)$','m'))||[])[1]||'').trim();console.log({name:v('name').length,description:v('description').length,when_to_use:v('when_to_use').length,lines:t.split('\\n').length,approxTokens:t.trim().split(/\\s+/).filter(Boolean).length})"
rg -n "type=\"module\"|type='module'" index.html app.js styles.css setup examples scripts
LC_ALL=C rg -n "[^\\x00-\\x7F]" AGENTS.md README.md index.html app.js styles.css setup examples
```

The `rg` checks should return no matches.

---
> Source: [thealialexander/self](https://github.com/thealialexander/self) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
