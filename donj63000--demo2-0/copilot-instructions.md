## demo2-0

> Guidance for maintainers and automation agents working on DemoLoterie. Keep this playbook close, follow the checklists, and add context when you discover new constraints.

# Agent Playbook

Guidance for maintainers and automation agents working on DemoLoterie. Keep this playbook close, follow the checklists, and add context when you discover new constraints.

---

## TL;DR Workflow
1. **Prep** – read open issues/TODOs, skim `AGENTS.md` + `README.md`, update dependencies if needed.
2. **Build once** – `mvn clean package` to ensure the shaded jar still assembles.
3. **Code + test in lockstep** – every time you add production code, create or extend its matching unit test before moving on.
4. **Tight dev loop** – run `mvn javafx:run` for UI changes, or `mvn test -Dtest=ClassNameTest` for focused suites; add logs where they clarify runtime behavior.
5. **Validate everything** – execute the full suite (`mvn test`) after every change, then note any manual UI/data spot checks.
6. **Deliver** – summarize intent, list changes, capture evidence in commit/PR body, reference affected data files.

---

## Mission Brief
- Extend the JavaFX lottery simulator launched via `org.example.Launcher`.
- Keep new code in the `org.example` namespace unless there is a justified module boundary.
- Ship portable artifacts through the shaded jar (`target/demoloterie.jar`) or, when requested, the Windows installer profile.
- Treat UI fidelity and deterministic draws as non-negotiable requirements.

---

## Exigences QA incontournables
- **Chaque ajout de code** doit s’accompagner de son test unitaire dédié avant de pousser plus loin.
- **Chaque modification**, même minime, impose d’exécuter **toute la suite** (`mvn test`) afin de garantir l’absence de régressions.
- **Chaque flux sensible** doit exposer des logs explicites (niveau INFO/DEBUG/WARN selon le cas) pour faciliter le diagnostic terrain.
- Documentez dans les PR quelles classes/tests/logs ont été ajoutés ou enrichis afin que la traçabilité reste totale.

---

## Architecture Snapshot
- **Launcher/Main** (`src/main/java/org/example/Main.java`): bootstraps JavaFX, applies theming, and wires up the wheel, ledger, and controls.
- **Roue & Gains**: RNG-backed wheel, must stay deterministic under seeded tests.
- **DonationsLedger & Participant**: CSV-backed models that mirror `loterie-dons.csv` et al. Never mutate user data during tests.
- **ScaledContentPane**: owns the global UI scale. Always construct it with the design resolution (`DESIGN_WIDTH/HEIGHT`) and choose the upscale policy explicitly.
- **Theme/resources**: CSS, FXML, and images belong under `src/main/resources/` so they are packaged with the jar.
- **Runtime data**: `loterie-dons.csv`, `loterie-historique.txt`, `loterie-save.txt` live at the repo root; prefer fixtures under `src/test/resources/` when writing tests.

When adding new functionality, colocate related classes in `org.example` (or a small subpackage such as `org.example.ledger`) to keep imports tidy.

---

## UI Layout & Scaling Rules
- The root scene is wrapped by `ScaledContentPane`. Set `scaledViewport.setAllowUpscale(true)` for "shrink + grow" behavior (recommended) or `false` for letterboxing on large displays.
- Never apply additional `setScaleX/Y` transformations to `rootPane`; all global scaling should remain inside `ScaledContentPane`.
- Keep the logical design size at 1920×1080. If you must change it, update constants, CSS expectations, tests, and documentation in the same change.
- When touching JavaFX layouts, run `mvn javafx:run` on both small (≤1366×768) and large (≥2560px wide) windows to confirm centering/clipping remain correct.
- Add concise comments for non-obvious layout constraints (drag thresholds, clip rectangles, animation timing).

---

## Command Reference
- **Build shaded jar**: `mvn clean package`
- **Run UI quickly**: `mvn javafx:run`
- **Execute packaged app**: `java -jar target/demoloterie.jar`
- **Unit tests**: `mvn test` or `mvn test -Dtest=ClassNameTest`
- **Windows installer**: `mvn clean package -Pwindows-installer` (JDK 21+, Windows)

Cache these commands in scripts if you automate them, but keep the canonical invocation in this doc.

---

## Coding Standards & Tooling
- Java 21, 4-space indentation, lines ≲120 chars. No tabs.
- Classes = UpperCamelCase, members = lowerCamelCase, constants = SCREAMING_SNAKE_CASE.
- Use JavaFX properties where state is observable; avoid static singletons for UI controllers.
- Prefer `rg`, `mvn`, and provided scripts inside the repo; do not introduce new build tools without discussion.
- Keep comments short and meaningful—explain *why*, not *what*.
- Instrument functional code with explicit logs (e.g., via `Logger`) so operational issues are visible without attaching a debugger; include context such as participant IDs, draw results, and ledger deltas.

---

## Testing & QA Expectations
- Tests live under `src/test/java/org/example/` and must follow the `*Test` suffix (e.g., `DonationsLedgerTest`).
- Every new feature or class must ship with at least one focused unit test that exercises the added behavior; do not merge production code without its companion test.
- Seed RNG-dependent code (wheel, gains) so tests are deterministic. Inject seeds or expose seam points when necessary.
- Use fixtures instead of mutating the runtime CSV/TXT files. If a bug depends on current data, clone the relevant rows into a test resource.
- Run the full suite (`mvn test`) after any change—no matter how small—to ensure regressions are caught early. Document any skipped suites or known flaky tests in the PR description.
- For UI-affecting changes, list manual verification steps (screen size, action performed, expected outcome).

---

## Data & Configuration Discipline
- Preserve the column order and encoding of CSV/TXT files. Document any irreversible migration and supply conversion scripts when needed.
- New resource files (CSS/FXML/images) belong in `src/main/resources/` and should be loaded via the classpath (`getResource`).
- Avoid hard-coded absolute paths. Use `Paths.get` or resource streams.
- When touching ledger persistence, mention the expected impact on `loterie-dons.csv` and ensure back-compat with existing entries.

---

## Commit & PR Workflow
- Commit messages: imperative voice, ≤60 characters (`Refine ledger persistence`). Add a short body with context, data impacts, and test evidence.
- Reference issues (`Fix #12`) when relevant. If UI or data change, attach before/after notes or screenshots.
- PR checklist:
  - [ ] Intent + scope clearly stated.
  - [ ] Key changes enumerated (features, refactors, data updates).
  - [ ] Test evidence pasted (`mvn test`, `mvn javafx:run`, manual checks).
  - [ ] Data or config migrations explained, with rollback notes if applicable.

---

## When in Doubt
- Prefer incremental, reversible changes. Leave `TODO(username): context` markers rather than half-finished refactors.
- Ask for clarification before moving files out of the documented structure or adding new external dependencies.
- Keep local experiments (scratch code, large datasets) out of commits; clean your workspace before pushing.
- Document surprises (API quirks, platform bugs) in this file so the next agent starts informed—feel free to append new sections following the style above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Donj63000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Donj63000)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
