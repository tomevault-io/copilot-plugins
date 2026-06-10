## eve-online-overview-forge

> - Prefer correctness, small diffs, and verified behavior.

# Repository Instructions

- Prefer correctness, small diffs, and verified behavior.
- Keep the core app offline/local. Do not make ESI required for Phase 1 behavior.
- Preserve unknown EVE overview YAML data whenever practical. Do not silently discard unknown keys.
- Preserve source top-level YAML key order during roundtrip when the document came from YAML.
- Preserve modeled pair-list order during YAML roundtrip when present.
- Tolerant YAML import should record structured `importWarnings` when it skips malformed content.
- SDE/group validation must remain optional and offline by default.
- `--group-ids` validation must use only local files; do not add network SDE downloads without explicit approval.
- `download-sde` may fetch only explicit or official CCP static-data URLs and must not make validation depend on network access.
- Canonical JSON import should be strict Pydantic validation. Tolerant import behavior belongs to YAML only.
- Overview snapshots should write canonical JSON plus a checksum manifest and should not mutate source YAML.
- Snapshot restore must verify checksums and write only to an explicit output path, whether restoring JSON or YAML.
- Workspace project files should reference overview JSON, snapshot roots, and profile roots; they should not embed opaque EVE profile files.
- Workspace project SDE paths should reference local files only.
- Expected CLI failures should return structured JSON errors, not Python tracebacks.
- GUI import/export should default to the current user's `Documents\EVE\Overview`, remember user changes, and avoid overwriting existing files by generating unique names.
- Current overview mode defaults to 20 tabs. Legacy compatibility mode remains capped at 5 tabs.
- Treat `_BracketFilterShowAll` and `DefaultPreset_<digits>` as EVE built-in/default preset references.
- Keep generator behavior small and explicit; do not turn it into a broad DSL without approval.
- Treat `core_user_*.dat` and `core_char_*.dat` as opaque files. Do not parse or edit them at field level.
- Character names may be resolved from `core_char_<id>.dat` filenames only when explicitly requested; normal profile scans must remain offline.
- Resolved character names may be cached locally; the cache must not be required for offline profile scanning.
- GUI character name resolution defaults to on, while CLI/core scans remain explicit/offline by default.
- GUI SDE/group-name display should use bundled or local metadata where practical and fall back safely to numeric IDs.
- Profile reports must remain read-only.
- Ignore `.dat.bak` files and `settings_*.bak_*` directories in normal profile root scans unless the user explicitly targets a backup folder.
- Clone planning must remain dry-run by default. Exact identifier mapping is the default; source-to-all fan-out requires an explicit option.
- Clone plan output should keep summary metadata and overwrite/risk details current whenever planner behavior changes.
- Backups created from clone plans should copy only unique planned target files and include the clone operation ID.
- Clone execution must require a reviewed plan and matching verified backup manifest; never delete target files.
- Clone execution must write an audit manifest with plan/backup references and before/after checksums.
- Rollback must verify backup checksums before restoring files and must write rollback metadata.
- Do not automate, inject into, or modify the running EVE client.
- Do not implement packet sniffing or out-of-client overview share-link handling.
- Legacy XML overview import is intentionally deferred/closed unless real XML samples and a concrete need are provided.
- Any future write operation to EVE folders must support dry-run, backup-before-write, preview, and rollback metadata.
- Match existing style and keep changes focused.
- Run targeted verification after behavior changes.

---
> Source: [gregoryhorn/EVE-ONLINE-OVERVIEW-FORGE](https://github.com/gregoryhorn/EVE-ONLINE-OVERVIEW-FORGE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
