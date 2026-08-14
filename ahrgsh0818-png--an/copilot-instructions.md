## an

> - Read `CONTEXT.md` before naming or changing product concepts. Keep it a glossary, not a feature specification.

# Repository instructions

## Sources of truth

- Read `CONTEXT.md` before naming or changing product concepts. Keep it a glossary, not a feature specification.
- Read `docs/adr/0007-ship-language-requests-before-writing-assistance.md` before changing v1 scope.
- Read `docs/adr/0008-use-native-shells-and-a-shared-rust-engine.md` before changing process or platform seams.
- Read `docs/adr/0001-use-the-local-codex-session.md` before touching authentication or model access.
- Read `docs/specs/0003-codex-review-workflow.md` before changing records, sync, review scheduling, Markdown projection, MCP tools, plugin packaging, or review authentication.
- Read `docs/specs/0004-knowledge-import-and-indexing.md` before changing Knowledge Folders, source import, Markdown preservation, indexing modes, retrieval slices, or knowledge upload tools.
- Read the relevant platform ADR before changing macOS compatibility, remote testing, or future Writing Assistance.
- Read `docs/specs/0002-native-ui-interaction.md` before changing the icon, Result Window, system-area entry, Work Mode controls, appearance, or interaction. Implement only its Settled decisions; grill the user and update that document before implementing an unspecified UI choice.

## Implementation contract

- A Selection and a Language Request remain separate user actions.
- Platform shells own global triggering, selection/context capture, permissions, and native presentation.
- The shared Rust engine owns request validation, Codex App Server communication, durable local event recording, the sync outbox, and Markdown projection.
- Communicate between a shell and the engine with versioned JSONL over local standard streams. Do not add a local network listener.
- Let Codex own ChatGPT authentication. Inspect account state through App Server; never read, copy, log, or store Codex credentials.
- Keep local translation authentication separate from An Review authentication. Device Sync and remote MCP use distinct OAuth clients, credentials, audiences, and scopes; neither may reuse Codex or ChatGPT credentials.
- Keep `language-core` free of platform APIs, process management, filesystem access, and network access.
- Keep `review-domain` free of platform APIs, network, filesystem, databases, OAuth, and MCP. It is the single Rust authority for Review Event validation, FSRS-6 transitions, due ordering, and progress projections.
- Preserve one Daily Log and at most one Review Log per event-local calendar date. They are system-managed, rebuildable Markdown projections; do not create one Markdown file per request and do not treat Markdown as the scheduling database.
- A Language Request must remain usable when An Review Cloud is unavailable. Commit its local event and durable outbox entry before reporting it as recorded; sync only in the background.
- After Cloud Authority Cutover, only the cloud service may calculate an official next review time or accept a canonical Review Grade. The legacy local helper becomes migration and diagnostics tooling only.
- A model may submit a final Review Grade, never Stability, Difficulty, Retrievability, interval, or due time.
- Preserve every imported Markdown Source Version byte-for-byte. Treat indexing as rebuildable derived data and require an explicit Knowledge Folder for every import.
- Keep one An MCP and one product capability contract for Codex and ChatGPT. Model current host limitations in the release capability matrix, never as server-side host discrimination: Version 0.1.0 Codex can compile, transfer, write, and run Voice Review; ChatGPT currently reads and practises without write-back.
- Treat Codex conversation as the Version 0.1.0 knowledge-management surface. It must inspect and report the resolved Knowledge Folder/Source state before performing ordinary create, import, rename, move, archive, restore, version, or delete operations through MCP; do not invent a hidden local-file management path.

## Scope discipline

- Build the explicit Language Request path first.
- Treat Harper integration, continuous text observation, cross-application underlines, and Writing Suggestions as post-v1 work.
- Do not add An-native review, account, sync, plugin, or mobile UI without first updating the relevant UI specification and receiving user confirmation. Hosted browser authentication is not an An-native surface.
- Prefer system UI and accessibility frameworks over web views or simulated native controls.
- Add a dependency only when the standard library or current workspace cannot express the required behavior clearly.

## Verification

Run the narrowest relevant checks while iterating, then finish Rust changes with:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

Finish macOS shell changes with the Swift package checks on macOS CI or a Mac:

```text
swift build --package-path apps/macos
swift test --package-path apps/macos
```

Do not describe macOS interaction behavior as verified from a Windows build or cloud compile. Accessibility, focus, animation, and cross-application behavior require the M1 test Mac.

## Agent skills

### Issue tracker

Issues and specs are tracked in this repository's GitHub Issues. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default five-role Matt Pocock triage vocabulary. See `docs/agents/triage-labels.md`.

### Domain docs

This is a single-context repository with one root glossary and system-wide ADR directory. See `docs/agents/domain.md`.

---
> Source: [ahrgsh0818-png/An](https://github.com/ahrgsh0818-png/An) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
