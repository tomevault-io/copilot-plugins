## waldo

> These rules apply to the entire repository.

# Working rules for WALDO

These rules apply to the entire repository.

## Begin with the contracts

Read `docs/VISION.md`, `docs/UX.md`, `docs/ARCHITECTURE.md`, and
`docs/COMPATIBILITY.md` before making architectural or cross-domain changes.
Record durable architectural decisions under `docs/adr/`.

The old WALDO repository is a behavioral and compatibility reference. Do not
copy its package structure or move code from it without a specific written
reason.

## Product vocabulary

- Use **lookaside**, never `store`, for the user-facing command group and the
  backend domain that holds content-addressed objects.
- An **OpenWALDO BOM** is the immutable, resolved handoff from the data side;
  materialization then verifies every object named by it.
- A model **compose** is consumed by `waldo model train <name> <file>`. Model composition
  terminology belongs to the model lifecycle; source acquisition uses an
  **ingestion recipe**.
- Fetchers are external shell scripts that live in another repository. WALDO
  may execute them only when the user explicitly supplies a strict
  `waldo-ingest-recipe`; scripts populate WALDO-owned temporary input space
  and never own conversion, publication, or index mutation.

## Dependency direction

- `index`, `record`, `license`, `lookaside`, and `corpus` must not import
  `model` or `training`.
- `model` and `training` consume OpenWALDO BOMs and provenance contracts;
  they must not independently traverse index trees, resolve manifest
  inheritance, normalize licenses, or fetch unverified shards.
- CLI packages may wire domains together but contain no domain logic.
- Interfaces belong near the consumer that needs them. Avoid general-purpose
  service containers and shared utility packages.
- A fact has one authoritative type and one owner. Do not create parallel
  manifest, shard, license, or OpenWALDO BOM representations for convenience.

## Development discipline

- Work in small vertical slices with an observable command and tests.
- Preserve the existing index format only where `docs/COMPATIBILITY.md` says
  it is a contract. Internal APIs are not compatibility surfaces.
- Write a run record before launching an external trainer and persist every
  terminal state.
- Prefer explicit data flow and ordinary Go over reflection or framework
  machinery.
- Error messages should state what failed, identify the relevant object or
  path, and give the next useful action when one exists.
- Never advertise an unimplemented command as working. Planned command
  scaffolds must say they are unavailable.

Before handing off a change, run:

```bash
gofmt -w .
./testing/all.sh
```

---
> Source: [openwaldo/waldo](https://github.com/openwaldo/waldo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
