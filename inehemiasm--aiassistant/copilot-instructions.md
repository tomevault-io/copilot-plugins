## aiassistant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands
- Build project: `./gradlew assembleDebug`
- Run all tests: `./gradlew test`
- Run Android instrumented tests: `./gradlew connectedAndroidTest`
- Run lint: `./gradlew lint`
- Clean project: `./gradlew clean`

## High-level Architecture
The project is an Android application following Clean Architecture and MVI (Model-View-Intent) patterns, enhanced with an **Agentic Workflow** and a hardened **Model Management System**.

### Project Structure
- `:app`: Main application module.
- `:ui-designsystem`: Shared design system (themes, components).

### Architecture Layers (in `:app`)
- **Domain Layer** (`app/src/main/java/com/neo/aiassistant/domain//`): Use cases, repository interfaces, and shared models like `ChatMessage`, `InferenceRequest`, and `InstalledModel`.
- **Data Layer** (`app/src/main/java/com/neo/aiassistant/data//`):
    - **Agent Layer** (`data/agent/`):
        - `AgentOrchestrator`: Implements the Reason-Act-Observe loop.
        - `ToolRegistry`: Manages available `AgentTool` implementations.
    - **Inference Runtime** (`data/inference/`):
        - `InferenceManager`: Central entry point for model loading and generation.
        - `LlmRuntimeManager`: Manages LiteRT-LM `Engine` and hardware backends (GPU/CPU).
    - **Data Sources** (`data/datasource/`):
        - `CompositeModelCatalogDataSource`: Merges models from Kaggle, Hugging Face, and Firestore.
        - `KaggleModelCatalogDataSource`: Curated TFLite models from Kaggle/TFHub.
        - `HuggingFaceModelCatalogDataSource`: Search + curated HF items with aggressive quality filtering.
    - **Registry** (`data/datasource/local/`): 
        - `RoomInstalledModelRegistry`: The source of truth for model health.
        - `AppDatabase`: Room database (current version: 3).
    - **Repository**: `ChatRepositoryImpl.kt` orchestrates between the Agent layer, Inference runtime, and Data sources. Includes disk-sync and orphan cleanup logic.
- **UI Layer**:
    - `MarketplaceViewModel.kt`: Manages discovery, lifecycle safety rules, and model switching.
- **Background Tasks**: `ModelDownloadWorker.kt` handles atomic downloads with SHA-256 verification and `VERIFYING` status transitions.

### Model Management Rules
- **Lifecycle Safety**: Active models cannot be deleted. Downloads/Switches cannot be interrupted by deletion.
- **Health Checks**: Models must pass minimum size and file-type validation to be considered `INSTALLED`.
- **Integrity**: SHA-256 verification is performed during the `VERIFYING` phase of download.

## Development Guidelines
- **Documentation Standards**: Always generate KDoc for new implementations and update kdoc when changes occur.
- **Database Schema**: If modifying `InstalledModelEntity`, increment `AppDatabase.version` and handle migration (current: fallback destructive).
- **Filtering**: New discovery items should be filtered for quality (size, runtime compatibility) before appearing in the UI.
- **Error Handling**: Use `InstallStatus` (e.g., `CORRUPTED`, `INVALID`) to surface actionable errors in the Marketplace.
- **State Representation**: Always use `sealed classes`, `sealed interfaces`, or `enums` to represent states (e.g., UI states, loading statuses). **Never use hardcoded raw strings** for state management.
- **Commit Attribution**: Do not add 'Co-Authored-By' or any other AI/agent attribution to git commit messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inehemiasm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
