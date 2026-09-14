## devsweep

> macOS menu bar app (Swift 6 toolchain, SwiftUI, macOS 14+) that finds dev processes stuck in limbo and dev caches/builds worth cleaning, suggests them proactively and cleans them in one click. Open source, MIT, contributions mostly in YAML.

# DevSweep

macOS menu bar app (Swift 6 toolchain, SwiftUI, macOS 14+) that finds dev processes stuck in limbo and dev caches/builds worth cleaning, suggests them proactively and cleans them in one click. Open source, MIT, contributions mostly in YAML.

## Commands

```bash
swift build                                           # debug build
swift test                                            # all tests, incl. stack/locale/privacy validation
swift test --filter StackDefinitionTests              # just validate stacks/*.yaml
DEVSWEEP_LIVE=1 swift test --filter LiveReportTests   # read-only report of what the app detects on this Mac
make app                                              # build/DevSweep.app (ad-hoc signed)
make install                                          # build, copy to /Applications, launch
swift scripts/redact-screenshot.swift in.png out.png --menubar 27,50 --blur x,y,w,h --cut y0-y1
```

Never run anything that actually cleans or kills on the contributor's machine to "test" a change. Use the live report, which only reads.

## Where things live

| Path | What |
|---|---|
| `stacks/*.yaml` | **What to detect and clean**, one file per stack. Schema: `CONTRIBUTING.md`. Start here for any "support tool X" request. |
| `locales/{en,pt-BR,es}.yaml` | UI strings. `en` is the reference. |
| `Sources/DevSweep/Definitions/` | YAML schema (`Schema.swift`), `Matcher`, loader with validation (`DefinitionLoader`), unknown-key lint (`SchemaLint`), globs/filters (`PathResolver`). |
| `Sources/DevSweep/Storage/` | `StorageCatalog` (definitions → targets), `StorageProviders` (Swift escape hatch), `ProjectScanner`, `Cleaner` + `SafeDelete`. |
| `Sources/DevSweep/Processes/` | `ps`/`lsof` parsing, grouping by rule, limbo evaluation, `ProcessKiller`. |
| `Sources/DevSweep/Advisor/` | `Advisor` (what to suggest), `Notifier`. |
| `Sources/DevSweep/App/` | `AppModel` (state, timers, actions), `Updater` (Sparkle), entry point. |
| `Sources/DevSweep/UI/` | Popover views. |
| `Tests/DevSweepTests/` | `TestSupport` loads the real `stacks/` and `locales/`. |
| `.github/workflows/` | CI + coverage, CodeQL, Scorecard, release-please + app/appcast publishing. |

## Rules

- **YAML first.** New detection or cleanup belongs in `stacks/`. Add a provider in `StorageProviders.swift` only when globs, filters and `project` can't express it, and add a test for it.
- **Every UI string goes through `tr("key")`** and the key must exist in all three locale files with the same `{placeholders}`. `LocalizationTests` scans `Sources/` for `tr("…")` calls. Don't build keys dynamically.
- **Stack texts** need `en`. Add `pt-BR` and `es` when you can.
- **No personal data anywhere in the repo:** no absolute user paths, usernames, emails, company or private project names. Stack paths must start with `~/` or `${VAR:-~/…}`. `PrivacyTests` enforces this; don't weaken it, fix the content. Test fixtures use neutral paths like `/opt/…` or `/p/…`.
- **Coverage stays at 90% or above** for everything except SwiftUI views, `AppModel`, `Updater` and `Notifier`, which need a UI host or system services. That exclusion list lives in two places that must stay in sync: `ignore:` in `codecov.yml` and `-ignore-filename-regex` in `.github/workflows/ci.yml`. Prefer making code testable (an option for a root path, a pure parsing function) over adding to the list.
- **Binaries must not leak build paths either.** `scripts/build-app.sh` strips debug symbols (`strip -S -x`) and fails if the bundle still contains any home folder path. Don't remove that step.
- **Safety invariants.** Keep them intact:
  - Every deletion goes through `SafeDelete.remove` (home-only, protected roots).
  - `ProcessKiller.terminate` re-checks the PID's command before signaling and only targets the current user's processes.
  - `data_loss` items never run in bulk ("Clean recommended", notifications) and always need a second click.
- **Risk labels are a promise to users.** `safe` = regenerated without downloads, `redownload` = comes back with a download, `data_loss` = user data.
- **Rule order matters:** first match wins, by stack `order`, then file order, then `fallback: true` rules. When adding a broad matcher, add a grouping test in `ProcessTests.swift` proving it doesn't steal processes from a more specific rule.
- The code builds in Swift 5 language mode (`swiftLanguageModes: [.v5]`). Keep blocking work (`ps`, `du`, deletion) off the main actor, as `AppModel` does with `Task.detached`.
- Repository docs, code comments and commit messages are in English.

## Commits and releases

Use Conventional Commits. They drive release-please's changelog and version bumps:
`feat(stacks): detect Rust target folders`, `fix(processes): …`, `feat(i18n): add French`, `docs: …`, `ci: …`.
Merging the release PR that release-please opens publishes the GitHub release, and the workflow attaches the universal app zip and the Sparkle `appcast.xml`. See `.claude/skills/release`.

## Skills in this repo

- `add-stack`: support a new tool, language or framework, or fix a false positive or negative.
- `add-language`: add or update a translation.
- `investigate-detection`: explain why something is or isn't detected or suggested.
- `redact-screenshots`: prepare screenshots for the README or a PR without leaking personal data.
- `release`: cut a release or fix the release pipeline. One-time account and token setup lives in `docs/MAINTAINING.md`.

---
> Source: [Lukitaduarte/DevSweep](https://github.com/Lukitaduarte/DevSweep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
