## glm-5-3-flash-nvfp4-2x-4x-dgx-sparks-ringside

> - Every served byte is pinned by sha256. Change a profile only through `sources/import_plan.py` from a lab plan (a

# Project instructions

- Every served byte is pinned by sha256. Change a profile only through `sources/import_plan.py` from a lab plan (a
  measured plan, or one derived from a measured plan with the checks it passed) and its files (`docs/updating.md`); do
  not edit patches, `src/` files or the manifests by hand.
- Keep upstream bases, patches, the recipe's own files and binary artefacts separately attributable. Preserve upstream
  authorship, licences and notices; never relicense a third-party file.
- A CPU check or a successful build is not GPU or serving qualification. Keep measured results separate from
  estimates, and keep single-run caveats and the non-deterministic MoE accumulation of both profiles visible.
- Name comparison recipes by GitHub owner and repository with the measured commit; call them comparison recipes.
- Site data (addresses, host names, paths, interfaces) stays in an ignored `site.env`; never commit it.
- Don't access nodes, launch containers, create remotes, push or publish unless the person you are working for asks
  for it. Contributions follow `CONTRIBUTING.md`.

---
> Source: [othexmr/GLM-5.3-Flash-NVFP4-2x-4x-DGX-Sparks-RiNGSiDE](https://github.com/othexmr/GLM-5.3-Flash-NVFP4-2x-4x-DGX-Sparks-RiNGSiDE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
