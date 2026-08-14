## implicit-kalman

> - Prefer modern, maintained APIs over deprecated/removed ones.

# ImplicitKalman

## Dependency policy

- Prefer modern, maintained APIs over deprecated/removed ones.
- When a dependency drops an API, do **not** pin to an old major version as a long-term fix — first try to migrate to a functionally equivalent modern replacement (in the same package or a sibling package already in `requirements.txt`). Pinning is only acceptable as a temporary unblock.
- If migration requires a new dependency, prefer one that is already transitively present (e.g. `scikit-image`, `Pillow`) over adding a new top-level package.
- Keep `requirements.txt` minimal and unpinned unless a specific version is required for correctness; record the reason inline as a comment when pinning.

### Known migrations

- `scipy.misc.imresize` → `skimage.transform.resize(..., order=0, preserve_range=True, anti_aliasing=False)` for nearest-neighbor; drop the legacy uint8 round-trip (`/255 * (max-min) + min`) since `preserve_range=True` keeps the original float range.

---
> Source: [XIXUM/implicit-kalman](https://github.com/XIXUM/implicit-kalman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
