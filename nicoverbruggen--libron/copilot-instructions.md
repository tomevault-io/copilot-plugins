## libron

> - Use the `fntbld-oci` container for build-related tasks instead of native host tooling. This container has access to Python with `fonttools`, and also FontForge.

# Agent Instructions

- Use the `fntbld-oci` container for build-related tasks instead of native host tooling. This container has access to Python with `fonttools`, and also FontForge.
- Always validate that the build script works correctly when applying changes.
- Preferred command from the repository root:
  `podman run --rm -v "$PWD":/work -w /work ghcr.io/nicoverbruggen/fntbld-oci:latest python3 build.py`
- When creating a new release, bump `VERSION` in the repository's root first.

---
> Source: [nicoverbruggen/libron](https://github.com/nicoverbruggen/libron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
