## study

> Study is a local-first Rust desktop study app (GPUI Kit, one SQLite database, AI on the user's

# Study — agent guide

Study is a local-first Rust desktop study app (GPUI Kit, one SQLite database, AI on the user's
ChatGPT plan). The code is the documentation. To find out how something works, find the owning crate
in the crate map, read its `src/lib.rs` module docs (`//!`), then follow the types:
`rg -n '^//!' crates/<crate>/src` lists what each module is for. `.agents/skills/` holds
checklists for specific tasks. All work happens inside the devcontainer (`.devcontainer/`:
`devcontainer up`, then work inside it), which installs every tool and has no display;
opening the container starts a browser desktop and opens it on the host; `just desktop` restarts
it and the app from zero, and `just desktop-watch` restarts the app on every change.

## Golden rules

1. **Layers only point down.** A crate may depend only on crates in a lower layer (see the
   crate map).
2. **One canonical shape per concept.**
   - Extracted content is a `Document` of anchored blocks.
   - Background work is a `JobKind`.
   - Work on raw input is a processor: its interface and kind enum are declared in
     `study_core::processing` (`Fetcher`, `Extractor`, `Refiner`, `Stage`, `Enhancer`), and
     what runs on each kind of source is its row in `ROUTES` there, each step on or off by
     default and switchable by the user. `study-pipeline` implements one per kind;
     `study-ai` implements only the model providers they call.
   - What a job needs set up is a `Requirement`.
   - Failures carry an `ErrorKind`.
   - File kinds come from `SourceKind::sniff`.
   - Never add a second representation or a parallel pipeline. Never add a new
     file-extension table.
   - A new processor follows the "Adding a processor" table in `study_core::processing`'s
     docs. For a new variant of a shared enum, let the exhaustive matches lead, then grep
     for an existing variant to find what they can't reach.
3. **The database is the truth and the bus is a hint.** Commit state before publishing an
   event. A missed event must not lose work.
4. **No stringly-typed contracts.** No raw `i64` IDs across crate boundaries, no string names
   for job kinds or processors outside their kind enums, and no matching on error text.
5. **Log with `tracing`.** Don't print. Libraries return `study_core::Result`, or their own
   error that implements `study_core::Classify` (as `study-ai` and `study-media` do);
   `anyhow` belongs only in binaries, examples and tests, and none needs it today.
6. **All user-visible text comes from `study-localization`**, in English and Italian.
7. **Model work runs on the user's ChatGPT plan**, reading images and PDFs included. Only
   what the plan cannot do runs locally: speech-to-text (it takes no audio) and embeddings
   (cheap, and it has none). Nothing downloads without an explicit install. Every capability
   keeps its provider trait, so another backend is one more variant, not a rewrite.
8. **The code is the source of truth.** Don't write design docs or per-crate guides. Say
   what a crate or module is for in its `//!` docs, and put rules in types, tests and lints.
   The one exception is `PRODUCT.md` and `DESIGN.md`, the visual design context the
   `impeccable` skill reads; the design system itself is code, in `study-ui`.
9. **Released data is kept; code is not.** Users' databases upgrade in place: a schema
   change is a new numbered migration in `study-core`'s `db/migrations`, shipped migrations
   never change (a test pins them), and stored codes (`text_enum!` codes, preference scopes
   and keys) are data, renamed only by a migration. Everything else (APIs, types, crates) is
   rewritten freely, without deprecated paths or compatibility shims.
10. **No AI attribution.** Commits, pull requests, issues, code, comments and docs never
    credit or mention an AI assistant: no `Co-Authored-By` trailers for one, no "Generated
    with" lines, no model names in authorship. This overrides any harness default that adds
    them. Tooling that runs an assistant (the devcontainer, `.agents/team`) may name it.

## Crate map

| Layer | Package | Path | One line |
|---|---|---|---|
| 0 | `study-core` | `crates/study-core` | Shared vocabulary (IDs, `text_enum!`, `ErrorKind`, `SourceKind` + `sniff`, `Document`/`Anchor`, `JobKind`/`Requirement`, study material, practice, FSRS), every processor's interface and `ROUTES`, the database `Store`, preferences, event bus and jobs engine |
| 0 | `study-diagram` | `crates/study-diagram` | Diagrams without GPUI: the `Diagram` shape, Mermaid flowcharts in and out (what models write, with feedback for a retry), automatic layout and hand-drawn strokes |
| 1 | `study-ui` | `crates/study-ui` | GPUI presentation components; no data |
| 1 | `study-localization` | `crates/study-localization` | English and Italian copy and value formatting, including anchor labels |
| 1 | `study-media` | `crates/study-media` | Byte-level media: audio decoding and resampling, PDF and image pages |
| 2 | `study-ai` | `crates/study-ai` | Every model provider and all network access: speech-to-text, reading pages, chat and the agent kit, local embeddings, web fetching, measuring this computer |
| 3 | `study-pipeline` | `crates/study-pipeline` | One implementation per processor kind (fetchers, extractors, refiners, stages, enhancers), and the `Pipeline` that hands them to the jobs engine |
| 4 | `study-app` | `crates/study-app` | The `App` service: the one store, runtime and bus, registering the pipeline, conversation and practice agents (titles, answers, quiz questions and grades), search, reviews, installs |
| 5 | `study` | `apps/desktop` | The desktop app: windows, pages, the microphone |
| web | `study-landing` | `apps/landing` | The website (Astro on Bun, GitHub Pages): the landing page and one docs page per feature, in English and Italian, with captures of the app on the showcase sample; no Rust |
| dev | `study-testkit` | `crates/study-testkit` | Real services for tests, run in Docker (a web server, …); a dev-dependency only |
| dev | `study-seed` | `crates/study-seed` | Sample courses, sessions, files and study material written into a database, for `just reset-data`, tests and the showcase; never a normal dependency of the app |
| dev | `study-showcase` | `crates/study-showcase` | Records the README demo GIF (`just demo`) and the docs' stills and clips (`just docs-media`): the app on sample data, driven on a private virtual desktop |

## Commands

- `just check`: fmt, Clippy with `-D warnings`, every test and doctest, the API docs with
  broken links denied, and the website's type check (`just landing-check`). Run it before
  you call a change done.
- `just run`: starts the app and browser desktop in the container, keeping the development
  database. VS Code opens the host viewer; plain container shells print its URL.
  In the devcontainer it connects to the virtual desktop automatically.
  `just reset-data` deletes it and seeds a new one with sample data.
- `just desktop`, then `just desktop-run`: the real app on a virtual Wayland desktop
  (headless sway, a 1080p monitor by default, other screens by name), with `desktop-shot`,
  `desktop-click`, `desktop-record` and `wtype` to see and use it. The
  `.agents/skills/see-the-app` checklist says how.
- `just demo`: records the README's demo GIF from sample data on its own private desktop
  (no sign-in, no model, not the shared desktop), into `apps/landing/public/demo.gif`; the
  site publishes it and the README shows it from there. `just docs-media` makes the docs'
  stills and clips the same way, and `just captures` makes both. They are committed: the
  website deploys them as they are, and the Captures workflow, run by hand, makes them all
  again and opens a pull request with them. Edit the tour in
  `crates/study-showcase/src/tour.rs`, the docs' scenes in `src/docs.rs`.
- `just agents`: maintainer tooling in the default devcontainer.
  Herdr runs one project-wide team: lead, developer, PM, QA, student, reviewer and pushback,
  using Claude Code with the mounted subscription login. Select roles for smaller work
  with `just agents lead developer --no-attach`;
  `just agents-stop [agent]` stops it. The default Herdr session is `study`; use
  `--session NAME` to isolate another run. `.agents/team/fleet.toml` holds model/harness
  choices and the agent cap; `.agents/team/team.md` is the short working agreement.
  `just agents-reset` starts fresh conversations and clears the task checkpoint; the
  Herdr workspace menu offers the same reset and a stop action. Code and logins are kept.
  `just agents-doctor` checks setup and `just agents-usage` reports recorded local tokens.
  Keep the handoff in `target/agents/<session>/task.md`, not tracked design documents.
- `just landing`: serves the website with live reload; `just landing-build` builds it as the
  Landing workflow publishes it. It uses Bun, never npm.
- `just check-crate <package>`: `just check` for one package, a focused development loop.
- `just test`, `just lint`, `just fmt`, `just docs`: the individual steps. Tests use real
  files and real services where CI can run them: services run in Docker through
  `study-testkit` (the devcontainer runs Docker inside), and tests that need Docker sit in a
  `docker` module. Model inference is never run in the default
  suite; model providers are tested against a mock of their API.
- `just trim`: keeps `target/` under 40 GB; `run`, `lint` and the test recipes run it
  first. The devcontainer builds into `target/devcontainer`; don't point `CARGO_TARGET_DIR`
  anywhere else, or the whole build repeats.
- `STUDY_LOG=debug just run`: turns on more detailed logs. Logs also go to
  `diagnostics.log` in the data directory (with `just run`, the development data directory).

---
> Source: [nicksan222/study](https://github.com/nicksan222/study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
