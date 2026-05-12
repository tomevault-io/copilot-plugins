## swiftports

> A monorepo of pure-Swift, cross-platform reimplementations of standard

# Agent instructions — SwiftPorts

A monorepo of pure-Swift, cross-platform reimplementations of standard
CLI tools and SDK clients. They share one `Package.swift`, one git
history, and one test runner.

## Today's targets

| Library        | Binary | What it ports                                     |
|----------------|--------|---------------------------------------------------|
| `ForgeKit`     | —      | Host-agnostic CLI plumbing: ANSI/TTY, Git client (Process + No-op), Secret store (Keychain + InMemory). Used by `GitHub` and the upcoming `GitLab`. |
| `ZipKit`       | —      | Pure-Swift PKZIP archive operations on top of `weichsel/ZIPFoundation`. Shared by Zip / Unzip / GitHub. |
| `ZipCommand`   | `zip`  | Info-ZIP `zip(1)` — create archives. |
| `UnzipCommand` | `unzip`| Info-ZIP `unzip(1)` — extract / list / test / pipe. |
| `GitHub`       | —      | GitHub SDK: API client, Codable models, OAuth device flow, GraphQL. No ArgumentParser dependency. |
| `GhCommand`    | `gh`   | The `gh` subcommand tree — built on top of `GitHub` + `ForgeKit`. SwiftBash extends `GhCommand` to register the whole tree as a Bash builtin. |
| `GitLab`       | —      | GitLab SDK: REST API client (`X-Next-Page` pagination, Bearer auth, `gitlab.com` and self-hosted instances), Codable models, `RepositoryReference` with nested-subgroup support. No ArgumentParser dependency. |
| `GlabCommand`  | `glab` | The `glab` subcommand tree — built on top of `GitLab` + `ForgeKit`. Today: `issue list / view / create / close / reopen / note / subscribe / unsubscribe / delete / board`, `mr list / view / create / update / close / reopen / merge / approve / unapprove / note / subscribe / unsubscribe / checkout / diff / delete`, `ci list / view / trace / status / retry / cancel / run / lint`, `repo {view,list,create,clone,fork,archive,unarchive,edit,delete}`, `release {list,view,create,delete,download}`, `tag {list,create,delete}`, `variable {list,set,unset}`, `label {list,create,delete}`, `api`, `auth {status,login,logout,token}`. Uses libgit2 via `SwiftGit.GitClient` (no shell-out). |
| `SwiftGit`     | —      | In-process `GitClient` impl backed by libgit2 1.9.x (vendored from `ibrahimcetin/libgit2` SwiftPM package). Drop-in replacement for `ForgeKit`'s `ProcessGitClient` — no system `git` binary required. HTTPS auth via `CredentialProvider` callback. Named `SwiftGit` (not `Git`) so its build artifacts don't case-fold-collide with the lowercase `git` exec on macOS. |
| `GitCommand`   | `git`  | The `git` subcommand tree — `init {--bare,-b <branch>} / clone / fetch / pull {--rebase} / checkout {-b/-B/--/<ref> --} / switch {-c/-C} / restore {--staged, --source} / push / add / reset {--soft,--mixed,--hard,-- <paths>} / status {-s,--porcelain,-b, ahead/behind} / commit / merge {--ff,--no-ff,--ff-only} / rebase {<upstream>,--continue,--skip,--abort,--onto} / cherry-pick {<commit>,--continue,--skip,--abort} / diff / log {--oneline,--format,--stat,-p,-<n>,<a>..<b>,-- <paths>} / show / blame / apply {--cached, --index} / reflog / describe {--tags,--dirty,--abbrev <n>} / ls-tree {-r, --name-only} / cat-file {-t,-s,-e,-p} / rev-parse {--short,--abbrev-ref,--git-dir,--show-toplevel,--is-inside-work-tree} / ls-files / mv / rm {--cached} / clean {-f, -n} / config {--get,--set,--list,--unset,--global,--system,--local} / stash {push,list,apply,pop,drop,clear,show,branch} / tag {-a -m, -d, -l, -n, -f} / remote {-v, add, get-url, set-url, remove, rename} / branch {-d, -D, -m, -M, --show-current} / version`. Output and exit-code semantics mirror real git for every supported case. SwiftBash can register `GitCommand` as the `git` builtin to shadow system git. See [Docs/SwiftGit.md](Docs/SwiftGit.md) for the full module surface. |

## Build, test, run

```bash
swift build                              # builds everything
swift test                               # all targets, all tests (156 today)
swift run gh ...                         # GitHub CLI
swift run glab ...                       # GitLab CLI
swift run git ...                        # libgit2-backed git CLI
swift run zip ...                        # zip(1)
swift run unzip ...                      # unzip(1)
```

`swift build -c release` produces optimized binaries under
`.build/release/`.

## CI

GitHub Actions workflow at `.github/workflows/swift.yml` runs on push
to `main` and PRs. Matrix:

- **macOS** — `macos-15` runner, `swift build && swift test`
- **iOS Simulator** — same runner, `xcodebuild` against the auto-generated
  `SwiftPorts-Package` umbrella scheme
- **Linux** — `swift:6.0-jammy` Docker image
- **Windows** — `windows-latest` + `SwiftyLab/setup-swift`
  (`continue-on-error` until libgit2 builds cleanly there)
- **Android** — `skiptools/swift-android-action`
  (`continue-on-error`, same caveat)

The Windows + Android jobs are stretch goals — `ibrahimcetin/libgit2`
1.9.x doesn't advertise official support and the C build path needs
platform-specific defines we haven't tuned yet.

## Layout — umbrella convention

A port is either:

- **Pure library** — flat folder `Sources/<Name>/`, one library target.
- **Library + binaries** — umbrella folder `Sources/<Umbrella>/` containing:
  - `Lib/` — the SDK library target, named `<Umbrella>` (e.g. `ZipKit`, `GitHub`).
  - `<X>Command/` — one library target per binary, holding the
    AsyncParsableCommand types (top-level + every subcommand). Library
    target so SwiftBash and other consumers can import it across
    packages and extend the Command struct.
  - `<x>/` — one executable target per binary, a four-line `Entry.swift`
    `@main` wrapper that delegates to `<X>Command.main()`.

The three layers form a one-way dependency chain:
`<x>` exec → `<X>Command` lib → `<Umbrella>` SDK lib → `ForgeKit` (when
applicable). The SDK lib has zero `ArgumentParser` dependency.

```
Sources/
  ForgeKit/                          flat library — no umbrella
    IO/, Git/, Secrets/

  ZipKit/                            Info-ZIP umbrella — 1 SDK + 2 binaries
    Lib/                                target "ZipKit"
    ZipCommand/                         target "ZipCommand"
    UnzipCommand/                       target "UnzipCommand"
    zip/                                target "zip"   (exec)
    unzip/                              target "unzip" (exec)

  GitHub/                            gh umbrella — 1 SDK + 1 binary
    Lib/                                target "GitHub"      (no ArgumentParser dep)
    GhCommand/                          target "GhCommand"   (Subcommands/, glue)
    gh/                                 target "gh"          (exec)

  GitLab/                            glab umbrella — same shape as GitHub
    Lib/                                target "GitLab"      (no ArgumentParser dep)
    GlabCommand/                        target "GlabCommand" (Subcommands/Issue/, glue)
    glab/                               target "glab"        (exec)

  SwiftGit/                          libgit2-backed GitClient + git CLI
    Lib/                                target "SwiftGit"    (Libgit2GitClient)
    GitCommand/                         target "GitCommand"  (Subcommands/, Subcommands/Remote/)
    git/                                target "git"         (exec)

Tests/
  ForgeKitTests/      — IO, Git, Secret store primitives (folded into GitHubTests today)
  ZipKitTests/        — Archive round-trips, GlobMatcher
  ZipTests/           — ZipCommand argv (depends on "ZipCommand")
  UnzipTests/         — UnzipCommand argv (depends on "UnzipCommand")
  GitHubTests/
    Fixtures/         — captured GitHub API JSON
    *Tests/           — SDK decode, networking mocks, OAuth, Configuration,
                       command parsing — depends on "GitHub", "GhCommand",
                       "ForgeKit"
  GitLabTests/
    Fixtures/         — captured GitLab API JSON
    *Tests/           — SDK decode, RepositoryReference parsing, Configuration,
                       IssueArgument parsing — depends on "GitLab",
                       "GlabCommand", "ForgeKit"
  SwiftGitTests/      — Libgit2GitClient + Credentials bridge round-trips
                       (depends on "SwiftGit", "ForgeKit")
  GitCommandTests/    — git argv parsing (depends on "GitCommand")
```

## Naming conventions

| Kind | Folder | Target name | Product / Binary |
|------|--------|-------------|------------------|
| SDK library | `<Umbrella>/Lib/` | `<Umbrella>` (PascalCase, matches umbrella) | `.library(name: <Umbrella>)` |
| Command library | `<Umbrella>/<X>Command/` | `<X>Command` (PascalCase) | `.library(name: <X>Command)` |
| Executable | `<Umbrella>/<x>/` | `<x>` (lowercase, matches binary) | `.executable(name: <x>)` |
| Standalone library | `<Name>/` (flat) | `<Name>` | `.library(name: <Name>)` |

- **One declaration per file.** `Type+Concern.swift` extensions for
  splitting big types.
- File basenames must be **unique within a target**. SwiftPM's build
  output uses the basename for `.o` files; duplicates collide.
- Lowercase exec folders sit alongside PascalCase lib folders without
  case-folding collisions on macOS's case-insensitive filesystem
  (because the names don't share a prefix).

## Conventions inherited across all ports

- **Models are Codable structs**, one per file. Decoder is configured
  centrally via `JSONDecoder.gitHub()` style factories
  (snake_case → camelCase, ISO 8601 dates, base64 data).
- **ArgumentParser** for every CLI. `AsyncParsableCommand` for
  anything that does I/O; sync `ParsableCommand` for the very few
  pure-string subcommands. Lives in the `<X>Command` library, never
  in the SDK lib or the exec target.
- **Tests with Swift Testing** (`@Test`, `#expect`, `#require`) — not
  XCTest.
- **HTTP via `swift-http-types`** + `URLSession` from
  `HTTPTypesFoundation`. Mocked in tests with a `URLProtocol` subclass
  registered on an `URLSessionConfiguration`.
- **No `Process` shellouts** anywhere except in clearly-marked
  Mac/Linux-only paths. The `GitClient` protocol exists specifically
  so iOS / sandboxed embedders can inject `NoGitClient`.

## SwiftBash consumption

SwiftBash registers a port's whole top-level command as a Bash builtin
via a one-line conformance against the `<X>Command` library target:

```swift
import UnzipCommand
import BashInterpreter
import ArgumentParser

extension UnzipCommand: ParsableBashCommand {
    public mutating func execute() async throws -> ExitStatus {
        do { try await self.run(); return .success }
        catch let code as ExitCode { return ExitStatus(rawValue: Int(code.rawValue)) }
    }
}
```

The same pattern works for the deep multi-subcommand CLIs:

```swift
import GhCommand
extension GhCommand: ParsableBashCommand { … }
```

Registering `GhCommand` makes the entire subcommand tree
(`gh issue list`, `gh pr view`, `gh auth status`, …) addressable as
one Bash builtin — argv parsing dispatches into the AsyncParsableCommand
graph automatically.

The conformance lives in SwiftBash; the implementation lives here. No
cycle. SwiftBash never depends on an executable target.

## GitLab / `glab` specifics

Detailed status, command inventory, and GitLab-specific conventions:
see [Docs/GitLab.md](Docs/GitLab.md). Today's surface: full `glab
issue` (list / view / create / update / close / reopen / note /
subscribe / unsubscribe / delete / board {list / view / create /
delete}), full `glab mr` (list / view
/ create / update / close / reopen / merge / approve / unapprove / note
/ subscribe / unsubscribe / checkout / diff / delete), `glab ci` (list
/ view / trace / status / retry / cancel / run / lint), `glab repo`
(view / list / create / clone / fork / archive / unarchive / edit / delete),
`glab release` (list / view / create / delete / download), `glab tag` (list /
create / delete), `glab variable` (list / set / unset), `glab label`
(list / create / delete), `glab api`
(generic authenticated REST request, with -F/-f field flags), and
`glab auth` (status / login / logout / token, PAT-based).

## GitHub / `gh` specifics

Detailed status, command inventory, and GitHub-specific conventions:
see [Docs/GitHub.md](Docs/GitHub.md).

Quick highlights:
- `auth login [--web] [--clipboard]` runs the OAuth device flow
  (covered by `Docs/OAuthAppSetup.md` for the publish-time client-ID
  swap).
- Token resolution: `GH_TOKEN > GITHUB_TOKEN > Keychain >
  ~/.config/gh/hosts.yml.oauth_token`.
- Repo inference from cwd via `git remote get-url origin`
  (`ProcessGitClient`, in `ForgeKit`).
- Adopts `swift-configuration`, `swift-http-types`, `swift-crypto`,
  `Yams`, Apple's `Security` framework.

## Adding a new port

1. Decide pure-library or library + binary.
2. **Pure library**: add `Sources/<Name>/` directly with a `.target`
   entry in `Package.swift` and a matching `.library` product.
3. **Library + binary**:
   - Create `Sources/<Umbrella>/Lib/`, `<X>Command/`, and `<x>/`.
   - Add three targets in `Package.swift`: `.target(name: <Umbrella>,
     path: "Sources/<Umbrella>/Lib")`, `.target(name: <X>Command, path:
     "Sources/<Umbrella>/<X>Command", dependencies: [<Umbrella>,
     ArgumentParser])`, `.executableTarget(name: <x>, path:
     "Sources/<Umbrella>/<x>", dependencies: [<X>Command])`.
   - Add three products: `.library(name: <Umbrella>)`, `.library(name:
     <X>Command)`, `.executable(name: <x>)`.
4. Add tests under `Tests/<X>Tests/`.
5. Update this AGENTS.md's table.
6. If SwiftBash should adopt it, the conformance + registration
   happens there against the `<X>Command` library.

## Skipped / out of scope

For the GitHub port specifically: `attestation` (Sigstore stack),
`codespace ssh` (dev-tunnels + PTY), `extension install` (Go-binary
plugin model), web-OAuth flow (browser + localhost listener; device
flow is sufficient).

---
> Source: [Cocoanetics/SwiftPorts](https://github.com/Cocoanetics/SwiftPorts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
