## redact

> `redact` is a Rust CLI that lets users manage filesystem trees by editing text buffers in their preferred editor.

# AGENTS.md

## Project Mission
`redact` is a Rust CLI that lets users manage filesystem trees by editing text buffers in their preferred editor.

Primary UX model:
1. Open directory as editable text.
2. Edit lines to express filesystem changes.
3. Save buffer to commit changes while staying in the same editor session.
4. Buffer is refreshed in place after each successful commit.

## Product Scope (v1)
In scope:
1. Rename/move by editing existing lines.
2. Move/delete-from-source by prefixing existing lines with `d ` (staged to clipboard).
3. Copy by prefixing existing lines with `c `.
4. Paste from clipboard by adding new lines with `p` or `p <name>`.
5. Create new files/dirs by adding plain new lines.
6. Directory creation using trailing `/`.
7. Global clipboard/stash at `/tmp/redact/mv`.
8. Parent navigation hint line `../`.
9. Editor-native navigation only.

Out of scope:
1. Editor-specific plugins/keymaps in core.
2. Multi-clipboard history.
3. Live apply-on-each-save daemon mode.

## Core Semantics (Must Preserve)
1. Prefix `d ` on existing entry lines moves them to clipboard and removes from source on commit.
2. Prefix `c ` on existing entry lines means copy to stash.
3. Prefix `p` on a new line pastes the next clipboard item with its original name.
4. Prefix `p <name>` on a new line pastes the next clipboard item and renames it to `<name>`.
5. Plain new lines (no prefix) create new files/dirs.
6. New copy/cut batch replaces previous stash batch.
7. Removing an existing line without `d ` is a validation error.
8. Rename is valid when the edited entry keeps its original `# id` and stays on its original line position.
9. Paste collisions on equal names are resolved by adding `_cpy` before the extension (or at end when no extension).
10. Navigation into a folder is triggered by saving with an empty line immediately below an existing directory entry line.
11. Navigation to parent is triggered by saving with an empty line immediately below `../`.
12. If a new line starts with `cd <path>` and `<path>` resolves to an existing directory, all other edits are ignored for that save and redact navigates to that directory.
13. Rendered listing keeps directories above regular files and aligns columns as: name | icon | size | modified | id.

## Buffer Contract
Expected format:
1. Comment lines start with `#`.
2. `../` exists for navigation and is ignored by apply.
3. Existing entries include metadata token `# id:<stable_id>`.
4. `c ` and `d ` prefixes apply only to known existing entries.
5. Trailing `/` indicates directory for new entries.
6. `p` and `p <name>` apply only to new lines.
7. Existing lines must not be removed silently; deletion must use `d `.
8. Each redact process owns one stable buffer file at `/tmp/redact/buffer-<instance-id>.redact` for its entire lifetime.
9. Different redact instances must never share the same buffer file.
10. Apply summaries/warnings/errors are written to `/tmp/redact/buffer-<instance-id>.log` and must not be injected into the editable buffer UI.
11. `cd <path>` is a navigation command (new line, no `# id`) and has precedence over normal apply operations when valid.

Never break backward compatibility of this format without explicit migration logic.

## Architecture Guidelines
Use these modules:
1. `cli.rs` for argument parsing and command dispatch.
2. `editor.rs` for editor process launch and environment handling.
3. `snapshot.rs` for reading directory state and stable ids.
4. `buffer.rs` for rendering/parsing textual buffers.
5. `diff.rs` for converting edits into planned operations.
6. `clipboard.rs` for stash manifest/payload lifecycle.
7. `apply.rs` for filesystem mutation ordering and safety.

## Safety and Correctness Rules
1. Source-removal behavior must be explicit (`d ` prefix).
2. Use deterministic operation ordering.
3. Emit actionable diagnostics with path + reason.
4. Preserve atomic intent where possible (especially rename ordering).
5. Handle cross-device copy/move fallback for clipboard payload operations.
6. Do not silently ignore malformed lines or omitted existing entries.

## Testing Requirements
Before merge, cover:
1. Parser correctness and error paths.
2. Rename/copy/cut/paste/create scenarios.
3. Stash replacement behavior.
4. Ambiguous name matching.
5. Hidden file behavior toggles.
6. Integration tests over real temp directories.

## CLI UX Rules
1. Respect editor precedence: `--editor` > `REDACT_EDITOR` > `EDITOR` > `vi`.
2. Launch editor with cwd set to target directory.
3. Support `redact logs <buffer-id>` and `redact logs --follow <buffer-id>` for instance log inspection.
4. Exit non-zero for fatal parse/apply failures.
5. Do not print apply summaries directly into interactive editor sessions; write them to per-instance log files.

## Change Policy
When extending features:
1. Keep default behavior stable.
2. Prefer additive flags/subcommands over semantic breaks.
3. Document new behavior in README and examples.
4. Add tests for both happy path and failure path.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JNSFilipe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JNSFilipe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
