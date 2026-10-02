## ledge

> The notebook for developers and DevOps, on macOS, Linux and Windows: it runs

# Ledge

The notebook for developers and DevOps, on macOS, Linux and Windows: it runs
code and commands straight from your Markdown. Built on Electrobun: Bun main
process (`src/bun/`, owns the filesystem and PTYs) + a React app in the system
webview (`src/mainview/`), talking over the typed RPC in
`src/shared/rpc-schema.ts`. On Windows the server runs in WSL (remote.md §11).

## Standards — read the one that governs your change

The standards live in `docs/contributor/`; the end-user manual (compiled into
the app as its built-in docs, via `src/bun/docsContent.ts`) lives in
`docs/user/`.

- **[architecture.md](docs/contributor/architecture.md)** — process/trust
  boundaries, filesystem invariants (rename-not-unlink, path guards), state
  ownership (store vs `useState` vs `configureX` hooks), settings policy (what
  earns a knob in settings.json), the recipe for adding an RPC method,
  dependency policy. Read before adding modules, RPC methods, state, settings,
  or dependencies.
- **[interactions.md](docs/contributor/interactions.md)** — every user-facing
  action is a command in `src/mainview/commands/`; hotkey allocation, row
  verbs, destructive-action policy, Escape layering, tooltips. Read before
  adding any user-facing action, key, menu, or button.
- **[locking.md](docs/contributor/locking.md)** — note locking (per-note
  encryption): the vault and envelope, the readNote/writeNote seam rules,
  the agents-never-read-locked-bodies invariant (MCP/CLI refusals, prompt
  fences), sealed images, and the lock commands' interaction grammar. Read
  before touching anything that reads note or asset bytes, the vault RPCs,
  or the lock UI.
- **[testing.md](docs/contributor/testing.md)** — what must be tested and how:
  colocated `bun test`, pure-core/DOM-wrapper split (no happy-dom — do not
  add one), invariant tests, the headless-WebKit harness (`test:e2e`) for UI
  behavior, live WKWebView probe recipe for the native seams (always against
  a scratch `LEDGE_NOTES_ROOT`). Read before writing tests or calling work
  done.
- **[remote.md](docs/contributor/remote.md)** — the `ledge-server` package and remote
  notes: the client is the least-trusted end, the framed protocol over ssh
  stdin/stdout, forced-command keys and host-key pinning, what state belongs
  to a server and what to a client, sessions that outlive connections, the
  one-round-trip budget, Linux and the Docker image. Read before touching the
  wire, connections, the daemon, or anything that assumes the server is in
  this process.
- **[ios.md](docs/contributor/ios.md)** — the iOS client (its §14 phases are
  all done: `ios/` is a Swift app that reaches a server over ssh with a Secure
  Enclave key, and all of §8's v1 works on a real iPhone): the shell around the same React view,
  why the protocol stays in JavaScript and which half of the transport is
  therefore portable, the twenty-string bridge, NIOSSH and what it does not do
  for you, the enclave and the pairing line, the server list a phone keeps for
  itself, what iOS suspension does to a connection, and how a device build
  differs from a Simulator one (§12).
  Its touch column is implemented and now lives in interactions.md §1a. Read
  before touching `src/shared/transport.ts`, `src/mainview/boot.tsx` or `ios/`.
- **[android.md](docs/contributor/android.md)** — the Android client: a Kotlin
  shell in `android/` around the same page as the iPhone, over sshj with a
  Keystore key. Where Android differs from ios.md (the bridge port, Bouncy
  Castle, the background network cut, Back, long presses, pictures), the
  `--store` bundle and its upload key, the Play Console's closed test, and the
  emulator probe traps. Read before touching `android/` or the page's Android
  paths in `nativeBridge.ts`.
- **[writing.md](docs/contributor/writing.md)** — prose style, for the docs and
  for the comments in the source: headings name the feature keyword-first, lead
  with the answer, one idea per sentence, mechanism before rationale, no
  aphorisms or design self-commentary, no em dashes, facts in tables, Diátaxis
  mode separation, plus the `docs/user/` mechanics (one line per paragraph, H1s
  are wikilink targets) and the comment mechanics (§11: name what the comment
  explains, five lines is the ceiling, cite the contributor page rather than
  restating it). Read before writing or editing any page in `docs/`, and before
  writing a comment.

These are normative: if code and doc disagree, one of them is wrong — fix
deliberately, not silently.

- **[releasing.md](docs/contributor/releasing.md)** — the release runbook, not a
  standard: what a release consists of, the two version numbers, the signing and
  notarization credentials, and what to verify on a signed build before
  publishing. Read before cutting a release or touching `electrobun.config.ts`'s
  build/signing keys.

## Commands

```
bun test             # unit + filesystem tests (scratch root via preload)
bun run test:e2e     # UI behavior in headless WebKit (Playwright harness)
bunx tsc --noEmit    # typecheck
bunx vite build      # build the view
bun run dev          # launch (bunx electrobun dev; bare `electrobun` is not on PATH)
bun run release      # the signed, notarized DMG (releasing.md)
bun run cli <verb>   # the `ledge` CLI from the checkout (src/bun/cli.ts; interactions.md §9)
bun run ios          # the iOS client in the Simulator (`-- --phone` for a device, `-- --store` for an App Store .ipa; ios.md §12)
bun run android      # the Android client in the emulator (`-- --server ledge@10.0.2.2 --port 2222` points a debug build at a server, `-- --test` runs its unit tests, `-- --store` the signed Play bundle; android.md §8)
bun run probe:ssh    # the ssh hop against a real sshd in Docker
bun run build:npm    # assemble the publishable `ledge-server` package (Mac + Docker; remote.md §11)
bun run probe:npm    # install that package on a toolchain-free container and drive it
bun run build:server # a server release: the packed npm tarball, and server.sh that installs it with a pinned Bun (remote.md §11)
bun run build:wsl    # the linux-x64 server a Windows build carries into WSL, in dist-wsl/ (releasing.md §10)
bun run probe:install # run that server.sh on Debian, Ubuntu and Alpine, dial it over sshd, update it
bun run probe:backup # ledge backup against an S3 server in Docker: setup, now, status, restore, the daemon's idle-exit backup
```

Done means: tsc clean, build clean, tests green (e2e too when UI behavior
changed) — and live-verified in the real webview when the change touches the
native seams (testing.md §6).

---
> Source: [ledgesh/ledge](https://github.com/ledgesh/ledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
