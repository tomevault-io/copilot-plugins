## quality-bar

> Quality expectations — when to spec, testing, anti-patterns, AI workflow


# zapstore-cli — Quality Bar

## When to Create a Feature Spec

Create a spec if the work:

- Changes install, update, or remove behavior
- Modifies verification logic
- Adds a new subcommand
- Changes the filesystem layout or symlink strategy
- Modifies platform/arch detection or asset filtering
- Could affect correctness of installed binaries

**Skip the spec** if:

- Output formatting changes (colors, copy)
- Flag alias additions
- Dependency update with no API changes
- Bug fix with obvious cause and fix

## Testing

- Verification logic must be unit-tested with known-good and known-bad hashes.
- Filesystem operations must be tested with temp directories.
- Platform detection must be tested for all supported OS/arch combinations.
- No real relay queries in tests — mock the Nostr client.

## Implementation Expectations

- `store/` handles all filesystem operations — commands must not do direct file I/O.
- `nostr/` handles all relay queries — commands must not construct filters directly.
- Output formatting lives in `ui/` — commands call `ui.Action(...)`, not `fmt.Printf`.
- Prefer extending existing commands over adding new packages.

## Anti-Patterns

- Installing a binary without verifying its hash
- Comparing version names instead of version codes
- Writing partial downloads to the final install path
- Mixing status output (stderr) with data output (stdout)
- Hardcoding relay URLs (use env var with default)

## Working With AI

- Spec-first for install/verify/update flow changes.
- Work packets in `spec/work/` for non-trivial tasks.
- Never modify `spec/guidelines/` without explicit permission.

## Knowledge Entries

After a work packet merges, promote non-obvious decisions to `spec/knowledge/DEC-XXX-*.md`. See `spec/knowledge/_TEMPLATE.md` for format and criteria.

---
> Source: [zapstore/zapstore-cli](https://github.com/zapstore/zapstore-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
