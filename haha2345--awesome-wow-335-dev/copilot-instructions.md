## awesome-wow-335-dev

> This repository is written for both humans and coding/research Agents. Follow these rules before proposing or implementing WoW 3.3.5a work from its contents.

# Agent guide for this repository

This repository is written for both humans and coding/research Agents. Follow these rules before proposing or implementing WoW 3.3.5a work from its contents.

## Reading order

1. Read the relevant language README.
2. Read `docs/catalog.yml`.
3. If provenance matters, read `docs/migrations.yml`.
4. Before invoking a bundled script, read `scripts/catalog.yml` and its topic guide.
5. Read `docs/<language>/01-development-map.md`.
6. Read the topic guide and any linked case study.
7. Check the page metadata and maturity language before reusing a conclusion.

## Target and ownership

- Default client target is WoW `3.3.5a.12340` on Windows x86.
- Default server target is AzerothCore WotLK.
- Treat addresses, signatures, offsets, object layouts, DBC rows, IDs, and binary assumptions as build-specific.
- Choose the lowest stable owning layer: AddOn, static client data, runtime client extension, server data/script, standalone module, then Core.
- Keep client presentation separate from server-authoritative ownership, rewards, persistence, and actions.

## Identity and dependency rules

- Never use “model ID” as an ambiguous catch-all.
- Keep ItemID, ItemDisplayInfo ID, SpellID, Mount ID, CreatureDisplayID, FileDataID, and paths distinct.
- An asset port is a dependency and identity chain, not a single M2 file.
- A DisplayID does not prove that an item, spell, mount, or server template exists.
- Do not copy modern DB2 rows directly into 3.3.5 DBC files.

## Evidence language

- Do not promote `REFERENCE`, `PLAN_ONLY`, `VALIDATED_LOCAL`, or `CASE_SPECIFIC` material to `REAL_CLIENT_ACCEPTED`.
- Keep source/static, build, server runtime, client runtime, and visual acceptance separate.
- Keep local commit, remote push, public Release, and deployment separate.
- Record unresolved checks instead of filling gaps with assumptions.
- A successful build is not visual acceptance for models, cameras, UI, markers, mounts, or animation.

## Safe public outputs

- Do not add Blizzard binaries or assets, including EXE, DLL, MPQ, DBC, DB2, M2, SKIN, BLP, WMO, ADT, WDB, audio, or video.
- Do not add database dumps, credentials, tokens, private logs, crash dumps, or machine-local absolute paths.
- Use placeholders such as `<CLIENT_ROOT>`, `<CORE_ROOT>`, and `<CASE_ROOT>`.
- Preserve third-party attribution and license links.
- Do not document anti-cheat evasion, credential capture, combat automation, or account abuse.
- Bundled original scripts are MIT-licensed; external libraries and game data are not.

## Documentation changes

- Chinese pages live under `docs/zh-CN`; English mirrors live under `docs/en`.
- A factual change should update both mirrors in the same pull request, or be explicitly labeled as a translation follow-up.
- Keep filenames and section order mirrored where practical.
- Add or update the matching entry in `docs/catalog.yml`.
- Prefer bounded, reproducible examples over machine-specific command transcripts.
- Public scripts must use explicit parameters instead of machine-local defaults and must document their write scope.
- Use the templates under `templates/` for new case studies and operation records.

## Agent handoff

When handing work to another Agent, include:

- target client/server and exact build where known;
- owning layer and why;
- inputs, IDs, and dependency chain;
- public/private asset boundary;
- current evidence state;
- rollback path;
- explicit acceptance scenarios;
- files allowed to change and files that must remain untouched.

---
> Source: [haha2345/awesome-wow-335-dev](https://github.com/haha2345/awesome-wow-335-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
