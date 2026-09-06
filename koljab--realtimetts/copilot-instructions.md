## realtimetts

> - A release must use `tools/release_guard.py publish`; direct `twine upload` is not an accepted release path.

# RealtimeTTS release contract

- A release must use `tools/release_guard.py publish`; direct `twine upload` is not an accepted release path.
- Build wheel and sdist once from a clean commit. Install that exact wheel in the target runtime, verify the service uses that runtime Python, run the focused runtime smoke, and create a detached Ed25519-signed attestation no more than 30 minutes before publishing those unchanged artifacts.
- The guard must inspect every linked worktree. If any worktree is dirty, stop and reconcile or explicitly preserve its changes; never discard them to make a release pass.
- Do not copy edited Python files directly into `site-packages`. Emergency fixes still go through a locally built wheel and an attestation before deployment is considered complete.
- Run the relevant test suite once before building. The parity guard is intentionally hash-based and fast; do not repeat long suites merely to satisfy it.
- Publication must require the wheel and sdist, a valid `linux-services` signature, matching remote master/tag refs, and an exact PyPI/TestPyPI JSON hash roundtrip. A local publish from Linux evidence must use `--allow-remote-attestation`; never use that option for a local runtime.

---
> Source: [KoljaB/RealtimeTTS](https://github.com/KoljaB/RealtimeTTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
