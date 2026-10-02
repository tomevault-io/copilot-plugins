## ai-journal-companion

> These instructions apply to the entire repository. They preserve the mandatory OOP 3 assessment requirements and must be followed by every contributor and coding agent.

# AI Journal Companion Development Rules

These instructions apply to the entire repository. They preserve the mandatory OOP 3 assessment requirements and must be followed by every contributor and coding agent.

## Source of truth

- Use `docs/CITEMS-APP-001_v1.0_AI_Journal_Companion_Planning.docx` as the primary planning source.
- Keep `docs/ARCHITECTURE.md`, `docs/REQUIREMENTS.md`, `README.md`, user help, and implementation details aligned.
- Treat assessment-mandated technologies, algorithms, data structures, and interactions as non-negotiable unless the assessor formally changes the brief.

## Mandatory technology and features

- Build the Android application in Kotlin with Jetpack Compose.
- Build the backend with Python FastAPI.
- Use a locally running Ollama service with the `gemma:2b` model for emotion analysis and advice generation.
- Communicate between Android and FastAPI using REST and JSON.
- Implement Bubble Sort, Insertion Sort, and Selection Sort as custom algorithms.
- Implement Binary Tree search, HashMap search, and Doubly Linked List search as the required search strategies and data structures.
- Draw the emotion-frequency pie chart with Jetpack Compose Canvas.
- Implement drag-and-drop deletion through a visible delete or trash target.
- Package a local HTML help file and display it in the application.

## Design rules

- Follow OOP principles: encapsulate state, give classes focused responsibilities, program to clear interfaces where useful, and prefer composition over tightly coupled code.
- Keep UI, domain models, networking, backend integration, sorting, searching, data structures, and visualisation logic modular.
- Keep journal entries in memory for the assessment unless a documented scope change approves persistence.
- Keep the Android UI responsive during network work and expose clear loading, success, and error states.
- Validate data at system boundaries and keep REST request and response models explicit.

## Algorithm and data structure rules

- Do not replace required custom sorting algorithms with built-in sorting methods such as `sorted`, `sort`, `sortedBy`, or equivalent library operations.
- Do not replace the required Binary Tree or Doubly Linked List implementations with library collections simply because they are easier.
- A Kotlin HashMap may back the required HashMap search strategy, but the lookup/indexing behaviour must be implemented explicitly and remain demonstrable as its own strategy.
- Keep each algorithm and search strategy independently selectable from the user interface and independently testable.
- Standard-library operations may be used in tests as an oracle for comparison, but not as the assessed implementation.

## Incremental development

- Build the assessment in small, reviewable increments following the iterations in the planning document.
- Establish and verify the FastAPI boundary before adding Ollama, then connect Android, then add journal management, algorithms, data structures, the chart, drag-and-drop, and help.
- Keep commits small and meaningful. Use feature branches for substantial work when practical.
- Do not add out-of-scope enhancements at the expense of assessment requirements.

## Testing and evidence

- Do not claim tests, builds, integrations, or manual checks passed unless they were actually run in the current environment.
- Record the command or procedure used for meaningful verification and report failures honestly.
- Test custom algorithms with empty, single-item, duplicate, already ordered, and reverse-ordered data where applicable.
- Test the REST API independently before relying on Android integration.
- Preserve evidence useful to the assessment, including test results, screenshots, defect notes, and meaningful Git history.

## Documentation

- Update documentation whenever behaviour, configuration, architecture, requirements, or setup steps change.
- Document local-network configuration and the requirement to run FastAPI and Ollama locally.
- Keep the local HTML help consistent with actual UI controls and limitations.
- Clearly state that journal history is in-memory for the current assessment scope.

---
> Source: [NeXuS-879/AI-Journal-Companion](https://github.com/NeXuS-879/AI-Journal-Companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
