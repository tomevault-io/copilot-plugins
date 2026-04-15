## psych-gen-app

> Guide the placement of new Dart files according to the feature-driven architecture.


- When creating a new file containing a Flutter Widget:
- If the widget represents a full screen or page (e.g., `FaceGenerationPage`), it must be placed in `lib/features/<feature_name>/presentation/pages/`.
- If the widget is a reusable UI component that is specific to one page, create a subdirectory for that page's widgets (e.g., `lib/features/face_generation/presentation/widgets/home/` for widgets used only by `FaceGenerationPage`).
- If a widget is shared across multiple features, place it in a `lib/core/widgets/` directory.
- Avoid creating new files directly under a top-level `lib/widgets/` directory. All UI components should belong to a specific feature or be explicitly shared in `core`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Szirle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Szirle)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
