## mobhunt

> > **App Acquisition Integrity**: NEVER download mobile binaries (IPAs, APKs, XAPKs) from third-party websites or services (APKPure, APKMirror, Aptoide, etc.). ALL app binaries MUST be acquired exclusively from official sources:

# MobHunt: Claude Code Configuration

> **CRITICAL RULES**
>
> **App Acquisition Integrity**: NEVER download mobile binaries (IPAs, APKs, XAPKs) from third-party websites or services (APKPure, APKMirror, Aptoide, etc.). ALL app binaries MUST be acquired exclusively from official sources:
> - **iOS**: App Store via `ipatool` (requires authenticated Apple ID)
> - **Android**: Google Play or Aurora Store on an Android emulator, then pull via ADB (`tools/acquisition/android_acquire.sh`)
> Third-party mirrors cannot be trusted; they may contain tampered, outdated, or repackaged binaries. If official store download fails, ask the user to provide the binary manually. NEVER suggest or attempt 3P download as a fallback.
>
> **No 3P Credential Tools**: NEVER hand user credentials (passwords, tokens, API keys) to third-party CLI tools or libraries beyond minimal crypto primitives. Build our own tools instead. This includes tools like `apkeep`, even if open-source.
>
> **Python Environment**: ALWAYS activate the virtual environment at `.venv` before installing Python dependencies. Create it with `python3 -m venv .venv` if it doesn't exist.

## Project Overview

MobHunt is an agentic mobile security research system for automated vulnerability hunting through bug bounty programs. It uses an orchestrator agent that spawns focused sub-agents, each assigned a narrow objective (one vulnerability class or attack surface area per agent).

See `DESIGN.md` for full architecture documentation, and `docs/ios-decryption.md` for the iOS decryption fallback chain.

## Workflow

```
/hunt <program>  →  scope → acquire → recon → hunt → chain → validate → report
                                                 ↑                        |
                                                 └──── agentic loop ──────┘
```

## Rules (Always Active)

The following rules are loaded on every session and must always be followed:

- `rules/hunting.md`: Mobile hunting discipline (scope verification, 5-min rule, concrete evidence)
- `rules/reporting.md`: Report quality standards (impact-first titles, mobile context, exact repro steps)
- `rules/ethics.md`: Responsible disclosure boundaries (in-scope only, no user data, static first, no publication without approval)
- `rules/mobile-specific.md`: Platform-specific rules (Apple Silicon, decryption verification, FAT binaries)

## Agents

Agents are spawned by the orchestrator with specific, narrow objectives:

| Agent | File | Model | Purpose |
|-------|------|-------|---------|
| Orchestrator | `.claude/agents/orchestrator.md` | Opus | Pipeline coordination, agentic loop |
| Scope Agent | `.claude/agents/scope-agent.md` | Haiku | Identify mobile assets from bug bounty programs |
| Acquisition Agent | `.claude/agents/acquisition-agent.md` | Sonnet | Download, extract, decrypt apps |
| iOS Hunter | `.claude/agents/ios-hunter.md` | Opus | iOS vulnerability hunting (x5 parallel) |
| Android Hunter | `.claude/agents/android-hunter.md` | Opus | Android vulnerability hunting (x5 parallel) |
| Android Mariana Hunter | `.claude/agents/android-mariana-hunter.md` | Opus (1M) | Mariana Trench taint analysis (x1, runs alongside Android hunters) |
| Android Semgrep Hunter | `.claude/agents/android-semgrep-hunter.md` | Sonnet (1M) | Semgrep pattern triage on jadx Java (x1) |
| iOS Semgrep Hunter | `.claude/agents/ios-semgrep-hunter.md` | Sonnet (1M) | Semgrep scan of IPA resources + embedded sources (x1) |
| Validator | `.claude/agents/validator.md` | Sonnet | Mobile 7-Question Gate + 4 post-gates |
| Report Writer | `.claude/agents/report-writer.md` | Opus | Generate H1/Bugcrowd reports |
| Chain Builder | `.claude/agents/chain-builder.md` | Sonnet | Combine findings into exploit chains |

Model assignments are configurable in `config/providers.json`. Three cost profiles are available: `quality`, `balanced` (default), and `economy`. Set `cost_profile` in providers.json to switch.

## Commands

| Command | File | Description |
|---------|------|-------------|
| `/hunt <program>` | `.claude/commands/hunt.md` | Full pipeline: scope → acquire → recon → hunt → chain → validate → report |
| `/scope <program>` | `.claude/commands/scope.md` | Identify mobile assets in a bug bounty program |
| `/acquire <app>` | `.claude/commands/acquire.md` | Download, extract, and decrypt a mobile app |
| `/recon <app>` | `.claude/commands/recon.md` | Static reconnaissance and attack surface mapping |
| `/hunt-ios <app>` | `.claude/commands/hunt-ios.md` | iOS-specific vulnerability hunting |
| `/hunt-android <app>` | `.claude/commands/hunt-android.md` | Android-specific vulnerability hunting |
| `/validate` | `.claude/commands/validate.md` | Apply mobile validation gates to findings |
| `/triage` | `.claude/commands/triage.md` | Rapid 2-minute go/no-go assessment |
| `/chain` | `.claude/commands/chain.md` | Build exploit chains from multiple findings |
| `/report` | `.claude/commands/report.md` | Generate submission-ready report |

## Skills

Skills are contextually loaded by agents and commands:

| Skill | File | Loaded By |
|-------|------|-----------|
| Mobile Hunting | `.claude/skills/mobile-hunting/SKILL.md` | `/hunt`, orchestrator |
| iOS Analysis | `.claude/skills/ios-analysis/SKILL.md` | `/hunt-ios`, ios-hunter |
| Android Analysis | `.claude/skills/android-analysis/SKILL.md` | `/hunt-android`, android-hunter |
| Mobile Vuln Classes | `.claude/skills/mobile-vuln-classes/SKILL.md` | hunters, validator |
| Binary Reversing | `.claude/skills/binary-reversing/SKILL.md` | hunters |
| Mobile Recon | `.claude/skills/mobile-recon/SKILL.md` | `/recon`, acquisition-agent |
| Report Writing | `.claude/skills/report-writing/SKILL.md` | `/report`, report-writer |
| Triage Validation | `.claude/skills/triage-validation/SKILL.md` | `/validate`, `/triage`, validator |
| SDK Analysis | `.claude/skills/sdk-analysis/SKILL.md` | hunters |

## Configuration

1. Copy `config/config.example.json` → `config/config.json` and fill in credentials
2. Copy `config/providers.example.json` → `config/providers.json` and configure LLM providers
3. Install external dependencies (see `DESIGN.md` → External Dependencies)

Both `config.json` and `providers.json` are gitignored. Credentials belong in those files or in the environment; never pass a password, token, or API key as a command-line argument, because argv is readable by any other process on the machine and lands in shell history.

## Key Directories

### Runtime output (gitignored, never published)

These four directories do not exist in a fresh checkout. The tool creates them as it runs, and they fill with target-specific material: acquired binaries, decompiled code, extracted secrets, findings, and draft reports. All of it is confidential to the program being tested.

| Directory | Purpose |
|-----------|---------|
| `workspace/` | Acquired binaries, extraction and decompilation output, per-target hunt state |
| `recon/` | Per-target reconnaissance output (scope, attack surface maps, SDK inventory) |
| `findings/` | Raw, validated, and killed vulnerability findings |
| `reports/` | Draft submission reports |

Never commit, push, or publish anything under these four, and never quote their contents into a public document. Paths are configurable under `directories` in `config/config.json`; if you repoint them, keep the new locations out of version control too.

### Repository content (committed)

| Directory | Purpose |
|-----------|---------|
| `.claude/agents/` | Sub-agent definitions |
| `.claude/commands/` | Slash command definitions |
| `.claude/skills/` | Contextually loaded skills |
| `rules/` | Always-active rules, plus Semgrep rules (`rules/semgrep/`) and Mariana Trench config (`rules/mt/`) |
| `tools/` | All automation scripts and tools |
| `wordlists/` | Pattern files for secret detection, URL schemes, etc. |
| `docs/` | Supplementary design documentation |
| `config/` | Example configuration files |

## External Tools

**Required**: `ipatool`, `jadx`, `apktool`, Ghidra
**Optional**: Binary Ninja (set `analysis.binary_analyzer` to `"binja"` in config), Semgrep, Mariana Trench
**Pre-installed** (macOS/Xcode): `otool`, `codesign`, `plutil`, `nm`, `lipo`, `strings`

---
> Source: [ivRodriguezCA/MobHunt](https://github.com/ivRodriguezCA/MobHunt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
