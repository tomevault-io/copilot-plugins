## cc-skills

> Claude Code skills marketplace: **37 plugins** with skills for ADR-driven development workflows.

# CLAUDE.md

Claude Code skills marketplace: **37 plugins** with skills for ADR-driven development workflows.

**Architecture**: Link Farm + Hub-and-Spoke with Progressive Disclosure

## Documentation Hierarchy

```
CLAUDE.md (this file)                          ◄── Hub: Navigation + Essentials
    │
    ├── plugins/CLAUDE.md                      ◄── Spoke: Plugin development (all plugins listed)
    │       └── {plugin}/CLAUDE.md             ◄── Deep: Per-plugin SSoT
    │                                                (project/stack/conventions/architecture live here,
    │                                                 NOT duplicated in root)
    │           └── skills/{skill}/CLAUDE.md   ◄── Deepest: Per-skill SSoT (emerging — opt-in per skill)
    │                                                (file table, invariants, recent-change log,
    │                                                 edit conventions; sibling to SKILL.md)
    │
    └── docs/CLAUDE.md                         ◄── Spoke: Documentation standards
            ├── HOOKS.md                       ◄── Hook development patterns
            ├── RELEASE.md                     ◄── Release workflow
            ├── PLUGIN-LIFECYCLE.md            ◄── Plugin internals
            └── LESSONS.md                     ◄── Lessons learned (dated entries)
```

**Progressive disclosure rule**: each layer must add information the next-shallower layer didn't already cover. Don't restate plugin invariants in the root; don't restate skill invariants in the plugin. When the user asks Claude something specific, Claude follows links downward — so the deepest layer's freshness matters most.

## Navigation

### Spokes & Docs

| Topic             | Document                                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Installation      | [README.md](./README.md)                                                                                                     |
| Plugin Dev        | [plugins/CLAUDE.md](./plugins/CLAUDE.md)                                                                                     |
| Documentation     | [docs/CLAUDE.md](./docs/CLAUDE.md)                                                                                           |
| Hooks Dev         | [docs/HOOKS.md](./docs/HOOKS.md)                                                                                             |
| Lessons Learned   | [docs/LESSONS.md](./docs/LESSONS.md)                                                                                         |
| Cargo TTY Fix     | [docs/cargo-tty-suspension-prevention.md](./docs/cargo-tty-suspension-prevention.md)                                         |
| Claude Code Proxy | [devops-tools/skills/claude-code-proxy-patterns/SKILL.md](./plugins/devops-tools/skills/claude-code-proxy-patterns/SKILL.md) |
| Release           | [docs/RELEASE.md](./docs/RELEASE.md)                                                                                         |
| Plugin Lifecycle  | [docs/PLUGIN-LIFECYCLE.md](./docs/PLUGIN-LIFECYCLE.md)                                                                       |
| Troubleshooting   | [docs/troubleshooting/](./docs/troubleshooting/)                                                                             |
| ADRs              | [docs/adr/](./docs/adr/)                                                                                                     |
| Resume Context    | [docs/RESUME.md](./docs/RESUME.md)                                                                                           |

### Plugin CLAUDE.md Files (37/37)

All 37 plugins have their own CLAUDE.md with Hub+Sibling navigation links. Access via `plugins/{name}/CLAUDE.md` or browse the full table in [plugins/CLAUDE.md](./plugins/CLAUDE.md).

**Emerging deeper layer**: skill-level CLAUDE.mds (one per skill, sibling to `SKILL.md`) are appearing where a skill is large enough that maintainers need a separate compass from the user-invocable instructions. First adopter: [`plugins/macro-keyboard/skills/{configure-macro-keyboard,emit-fn-key-on-macos,diagnose-hid-keycodes}/CLAUDE.md`](./plugins/macro-keyboard/CLAUDE.md). Add one to your skill if SKILL.md is starting to mix "what to do when invoked" with "what to know before editing".

**Active project (SSoT):** [plugins/claude-tts-companion/CLAUDE.md](./plugins/claude-tts-companion/CLAUDE.md) — project/stack/conventions/architecture for the Swift macOS companion binary. Critical invariants (e.g., _do not replace afplay with AVAudioPlayer_) live there, not here.

Key plugin docs: [itp](./plugins/itp/CLAUDE.md) | [itp-hooks](./plugins/itp-hooks/CLAUDE.md) | [gh-tools](./plugins/gh-tools/CLAUDE.md) | [devops-tools](./plugins/devops-tools/CLAUDE.md) | [gmail-commander](./plugins/gmail-commander/CLAUDE.md) | [tts-tg-sync](./plugins/tts-tg-sync/CLAUDE.md) | [calcom-commander](./plugins/calcom-commander/CLAUDE.md) | [claude-tts-companion](./plugins/claude-tts-companion/CLAUDE.md)

## Essential Commands

| Task             | Command                                                   |
| ---------------- | --------------------------------------------------------- |
| Validate plugins | `bun scripts/validate-plugins.mjs`                        |
| Release (full)   | `mise run release:full`                                   |
| Release (dry)    | `mise run release:dry`                                    |
| Execute workflow | `/itp:go feature-name -b`                                 |
| Setup env        | `/itp:setup`                                              |
| Add plugin       | `/plugin-dev:create plugin-name`                          |
| Autonomous loop  | `/autoloop:start` / `/autoloop:stop` / `/autoloop:status` |

## Plugin Discovery

**SSoT**: `.claude-plugin/marketplace.json`

```bash
# Validate before commit
bun scripts/validate-plugins.mjs
```

Missing marketplace.json entry = "Plugin not found". See [plugins/CLAUDE.md](./plugins/CLAUDE.md).

## Directory Structure

```
cc-skills/
├── .claude-plugin/marketplace.json  ← Plugin registry (SSoT, 37 plugins)
├── plugins/                         ← 37 marketplace plugins (each has CLAUDE.md)
│   ├── claude-tts-companion/        ← Swift macOS binary (active project)
│   ├── itp/                         ← Core 4-phase workflow
│   ├── itp-hooks/                   ← Workflow enforcement + code correctness (incl. autoloop stall guard)
│   ├── autoloop/                    ← Self-paced loop mode (replaces deprecated ru; renamed from autonomous-loop)
│   ├── clarify-prompts/             ← Stop-hook AskUserQuestion nudge (autoloop-aware)
│   ├── mise/                        ← User-global mise workflow commands
│   ├── gemini-deep-research/        ← Gemini Deep Research browser automation
│   ├── gmail-commander/             ← Gmail bot + CLI (1Password OAuth)
│   ├── macro-keyboard/              ← Karabiner remap for cheap 3-key pads (skill-level CLAUDE.mds)
│   └── ...                          ← 27 more plugins (full table: plugins/CLAUDE.md)
├── docs/
│   ├── adr/                         ← Architecture Decision Records
│   ├── design/                      ← Implementation specs (1:1 with ADRs)
│   ├── HOOKS.md                     ← Hook development patterns
│   ├── RELEASE.md                   ← Release workflow
│   ├── PLUGIN-LIFECYCLE.md          ← Plugin internals
│   └── LESSONS.md                   ← Lessons learned
├── .autoloop/                       ← autoloop campaign storage (gitignored)
│   └── <campaign-slug>--<short-hash>/
│       ├── CONTRACT.md              ← Live LOOP_CONTRACT
│       ├── PROVENANCE.md            ← Owner+history index
│       └── state/                   ← heartbeat.json + revision-log
└── .mise/tasks/                     ← Release automation (5 phases + postflight)
```

## Key Files

| File                                   | Purpose                 |
| -------------------------------------- | ----------------------- |
| `.claude-plugin/marketplace.json`      | Plugin registry (SSoT)  |
| `.releaserc.yml`                       | semantic-release config |
| `scripts/validate-plugins.mjs`         | Plugin validation       |
| `scripts/sync-hooks-to-settings.sh`    | Hook synchronization    |
| `scripts/sync-commands-to-settings.sh` | Command synchronization |

## Link Conventions

| Context        | Format    | Example                          |
| -------------- | --------- | -------------------------------- |
| Skill-internal | Relative  | `[Guide](./references/guide.md)` |
| Repo docs      | Repo-root | `[ADR](/docs/adr/file.md)`       |
| External       | Full URL  | `[Docs](https://example.com)`    |

## Development Toolchain

**Bun-First Policy** (2025-01-12): JavaScript global packages installed via `bun add -g`.

```bash
bun add -g prettier          # Install
bun update -g                # Upgrade all
bun pm ls -g                 # List
```

**Auto-upgrade**: `com.terryli.mise_autoupgrade` runs every 2 hours.

## Lessons Learned

See [docs/LESSONS.md](./docs/LESSONS.md).

<!-- GSD:project-start source:plugins/claude-tts-companion/CLAUDE.md -->

## Active Project

**[claude-tts-companion](./plugins/claude-tts-companion/CLAUDE.md)** — the Swift macOS companion binary (Telegram bot + Kokoro TTS + subtitle overlay) has its own CLAUDE.md as the SSoT. Project description, constraints, stack, conventions, architecture, and critical invariants live there. **Do not duplicate them here** — pre-2026-04-07 the root had a full copy and it drifted (wrong model path, wrong audio playback description).

Quick hand-off: read `plugins/claude-tts-companion/CLAUDE.md` when the user mentions TTS, karaoke subtitles, Telegram bot, session notifications, `tts_kokoro.sh`, or anything under `plugins/claude-tts-companion/`.

<!-- GSD:project-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:

- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.

<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.

<!-- GSD:profile-end -->

---
> Source: [terrylica/cc-skills](https://github.com/terrylica/cc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
