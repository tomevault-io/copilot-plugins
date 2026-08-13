## boffin-cpp-routing

> Activate ParselFire Core C++ pack index


# C++ Pack Activation

For C++ source files:

- Load `packs/cpp-architecture/pack.urf.md`.
- Use its `## ROUTING` and `## LEAVES` records for leaf selection.
- Use its `## STAGE-REFS` records to gather the family K ids (and their mirrored X ids) at each stage of the walk.

For ambiguous `.h` or `.c` files that reach this rule via the always-on router:

- Apply cpp-architecture only if the file contains C++ constructs (class, template, namespace, references, std::, noexcept, overloads).
- If the file is plain C, stay on universal only.

---
> Source: [MicSm/boffin](https://github.com/MicSm/boffin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
