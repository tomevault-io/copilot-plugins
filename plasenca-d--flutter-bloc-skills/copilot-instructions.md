## flutter-bloc-skills

> This repository's approved collection contains exactly five skills:

# Agent instructions

This repository's approved collection contains exactly five skills:

- `skills/flutter-bloc-architecture/SKILL.md`
- `skills/flutter-bloc-feature/SKILL.md`
- `skills/flutter-bloc-cubit/SKILL.md`
- `skills/flutter-bloc-bloc/SKILL.md`
- `skills/flutter-bloc-performance/SKILL.md`

## Maintenance conventions

- Keep every skill directory lowercase and hyphenated; its directory name must match the frontmatter `name`.
- Keep `name`, one-line quoted `description`, `license`, `metadata.author`, and `metadata.version` in frontmatter.
- Preserve the section order: Activation Contract, Hard Rules, Decision Gates, Execution Steps, Output Contract, References.
- Write imperative runtime instructions and keep the main skill body concise. Move conceptual detail and upstream evidence into local `references/` files.
- Keep copyable code templates under each skill's `assets/` directory and explain their workflow in a local `references/` file. Templates are manual inputs, not proof that Mason or another generator ran.
- Every path named by a skill must be local to that skill. External upstream URLs belong inside the local reference file, not as the skill's only reference.
- Keep the source boundary clear: architecture guidance does not scaffold features; feature guidance does not pretend to run unavailable generators; Cubit guidance does not replace architecture boundaries or feature scaffolding; Bloc guidance focuses on event-driven runtime behavior and does not replace architecture, feature, or Cubit guidance; performance guidance focuses on Bloc-specific rebuild, equality, concurrency, stream, scalability, and diagnostic decisions and does not replace those four skills or generic Flutter optimization.
- Update `README.md` whenever installation, listing, or usage commands change. Keep commands based on the `npx skills` CLI.
- The performance skill's profiling and DevTools guidance is explicitly local engineering guidance; its references and Dart assets are manual recipes, not upstream benchmarks, generated output, or proof that a build/test ran.

---
> Source: [plasenca-d/flutter-bloc-skills](https://github.com/plasenca-d/flutter-bloc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
