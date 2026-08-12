## fnq3

> FnQuake3 is a modernized Quake III Arena engine branch. Every change should protect these project constraints:

# AGENTS.md

## Mission

FnQuake3 is a modernized Quake III Arena engine branch. Every change should protect these project constraints:

1. Full compatibility with retail Quake III Arena and its demos.
2. Speed and efficiency in hot paths and tooling.
3. Modern platform support without regressing existing targets.
4. Cross-platform viability, even when a feature starts on one platform first.

## Repository Rules

- Treat demo, protocol, asset-loading, and VM behavior as compatibility-sensitive by default.
- Prefer incremental engine changes over broad rewrites unless a rewrite is the only coherent fix.
- Keep release packaging deterministic. `.install/` is the staged distribution area, not a scratchpad.
- Use `.tmp/` for temporary outputs, investigation notes, and disposable staging work.
- Do not ship documentation that mixes end-user guidance with maintainer detail. Keep the user surface in `README.md` and deeper material in linked technical docs.
- When versioning changes are required, update the canonical metadata in `version/fnq3_version.h` first.
- Treat Meson `subprojects/*.wrap` files as the ownership boundary for bundled third-party dependencies. Do not restore deleted in-tree vendor directories such as `code/libcurl/`, `code/libjpeg/`, `code/libogg/`, `code/libvorbis/`, `code/libsdl/`, or `code/openal/`.
- Legacy Makefile builds may use system development packages for dependencies, but they must not grow new private copies of third-party source trees. If a dependency needs a bundled fallback, wire it through Meson `subprojects/`.

## Local References

- `README.md`: end-user overview.
- `BUILD.md`: platform-specific build instructions.
- `docs/fnquake3/TECHNICAL.md`: maintainer-facing project, release, and repo notes.
- `version/fnq3_version.h`: single source of truth for project version metadata.
- `scripts/version.py`: version/channel helper for humans and CI.
- `scripts/generate_docs.py`: refreshes `README.md` and `.install/README.html` from templates.
- `scripts/release.py`: stages artifacts through `.install/` and produces release archives plus manifests.
- `subprojects/*.wrap`: Meson fallback definitions for third-party dependencies such as SDL3, OpenAL Soft, libcurl, libjpeg-turbo, Ogg, and Vorbis.
- `.github/workflows/release.yml`: main-branch build validation and manual release publishing.

## Directory Map

- `.install/`: tracked distribution docs plus generated package outputs during release staging.
- `.tmp/`: ignored scratch workspace for temporary files and intermediate staging.
- `code/`: engine, renderer, platform, and VM sources.
- `docs/`: maintainer docs, legacy upstream docs, and template sources.
- `subprojects/`: Meson wrap definitions and fetched fallback source directories for third-party dependencies. Wrap metadata is source-controlled; downloaded/extracted dependency trees are build inputs, not engine source.
- `version/`: canonical version metadata consumed by code, docs, and CI scripts.
- `scripts/`: small repo-local automation for docs, versioning, and packaging.

## Release Workflow

1. Update `version/fnq3_version.h` for the next tagged release.
2. Run `python scripts/generate_docs.py` to refresh user-facing docs.
3. Build platform artifacts.
4. Run `python scripts/release.py --channel manual` or `python scripts/release.py --channel release --ref-name <tag>` against the downloaded artifact directory.
5. Publish the archives produced under `.install/packages/` with the generated manifest and checksums.

## Guardrails

- If a change touches runtime identity strings, keep compatibility-sensitive behavior unchanged unless the user explicitly wants a compatibility break.
- If you have to choose between a cleaner abstraction and a safer compatibility-preserving patch, default to the compatibility-preserving patch and document the tradeoff.
- When release packaging changes, ensure `.install/README.html` remains valid and the package still includes `LICENSE` and `docs/fnquake3/TECHNICAL.md`.
- When build or CI changes need third-party libraries, prefer Meson dependency declarations with `dependency(..., fallback: ...)` and the existing `subprojects/*.wrap` files. Make and legacy project files should either consume system packages or delegate to Meson; they should not point at removed `code/lib*` dependency trees.

---
> Source: [themuffinator/FnQ3](https://github.com/themuffinator/FnQ3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
