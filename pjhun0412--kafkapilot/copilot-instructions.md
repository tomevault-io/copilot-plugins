## kafkapilot

> This file gives AI coding agents project-specific guidance for working in KafkaPilot.

# AGENTS.md

This file gives AI coding agents project-specific guidance for working in KafkaPilot.

## Project summary

KafkaPilot is an Electron, React, TypeScript, and KafkaJS desktop client for Apache Kafka. It is designed as a local Kafka workbench for browsing clusters, consuming and producing messages, replaying records, managing consumer group offsets, inspecting Avro payloads, and visualizing coordinate streams in a standalone Map Viewer.

## Codebase memory

This repository uses `codebase-memory-mcp`.

Always prefer the knowledge graph for code discovery before falling back to text search:

1. `search_graph` — find functions, classes, routes, variables, and feature areas.
2. `trace_path` — inspect callers/callees and impact paths.
3. `get_code_snippet` — read exact function/class source after finding its qualified name.
4. `query_graph` — run Cypher for broad structural questions.
5. `get_architecture` — get high-level project structure and hotspots.

Use grep/ripgrep only for:

- string literals, copy, error messages, or i18n keys;
- non-code files such as Markdown, config, scripts, and build metadata;
- cases where graph results are insufficient.

The current graph project name is `kafka-tool`.

## Architecture

Main source layout:

```text
src/
├─ main/       Electron Main Process, KafkaJS integration, IPC handlers, storage, updater, native windows
├─ preload/    Safe bridge APIs exposed to renderer windows
├─ renderer/   React UI, Zustand stores, hooks, workspace components, Map Viewer renderer
└─ shared/     Shared TypeScript types
```

Important runtime boundaries:

- Renderer code must call Main Process capabilities through the preload API.
- Kafka Admin, Consumer, Producer, file IO, dialogs, updater, and native windows belong in `src/main`.
- React components should stay UI-focused; orchestration belongs in hooks/controllers; reusable pure logic belongs in focused utility modules.
- Shared request/response shapes belong in `src/shared/types.ts` when used across Main/Preload/Renderer.

## Key feature areas

- Consume: `src/main/ipc/consume*.ts`, `src/renderer/components/workspace/consume/`, `src/renderer/consume*.ts`
- Replay: `src/renderer/replayJobs.ts`, replay logic in consume/message inspector components, produce IPC in Main
- Produce: `src/renderer/components/workspace/produce/ProducePanel.tsx`, `src/renderer/produceTemplate.ts`
- Consumer Groups: `src/main/ipc/consumerGroup*.ts`, `src/renderer/components/workspace/groups/`
- Map Viewer: `src/main/liveMapWindow.ts`, `src/preload/liveMapPreload.ts`, `src/renderer/map-viewer.ts`, `src/renderer/mapPreview.ts`, `src/renderer/mapViewerVehicles.ts`
- Avro: `src/main/avroDecoder.ts`, `src/renderer/manualAvroSchema.ts`, `src/renderer/components/modals/ManualAvroSchemaDialog.tsx`
- Settings/storage: `src/main/storage.ts`, `src/main/settingsTransfer.ts`, preferences stores and hooks in renderer
- i18n: `src/renderer/i18n.ts`

## Development workflow

Install dependencies:

```bash
npm ci
```

Run development app:

```bash
npm run dev
```

Validate before handoff:

```bash
npm run build
```

Optional type-only check:

```bash
npm run typecheck
```

Package/release scripts:

```bash
npm run package:win
npm run package:mac
npm run release:win
npm run release:mac
```

macOS packaging must run on macOS. See `docs/release.md` and `docs/macos-install.md`.

## Coding guidelines

- Keep TypeScript types explicit at process boundaries and IPC payloads.
- Avoid `Record<string, any>` for controller wiring unless there is no practical typed alternative.
- Keep controller hooks compositional. `useWorkspaceAppController.ts` should remain an assembly point, not a dumping ground for business logic.
- Do not introduce direct circular references between controller modules.
- Prefer small focused modules for parsing, formatting, mapping, and template rendering.
- Preserve split-pane independence. Topic, consume, produce, group, viewer, and selection state may differ per pane.
- Preserve per-topic and per-server preferences when adding viewer, produce, map, or consume settings.
- Treat exported settings as sensitive because server secrets, Schema Registry credentials, and bearer tokens may be included.
- For large consumes and live streams, avoid retaining unbounded raw payloads in renderer memory.
- When writing files or streams in Main Process, handle backpressure and cancellation paths.
- Keep user-facing strings in the i18n layer instead of hard-coding UI copy.

## Refactoring priorities

The graph currently identifies several large/high-complexity areas. Prefer reducing these before adding more behavior inside them:

1. `src/renderer/components/workspace/consume/MessageInspector.tsx`
2. `src/renderer/components/workspace/consume/ConsumePanel.tsx`
3. `src/renderer/components/workspace/produce/ProducePanel.tsx`
4. workspace refresh/navigation hooks
5. replay job and map viewer flows when adding more async behavior

For `MessageInspector.tsx`, split new work into focused pieces where possible:

- replay dialog/actions;
- viewer mode rendering;
- field picker;
- map actions;
- copy/export actions.

## Documentation

When behavior changes, update the closest user/developer document:

- `README.md` and `README.ko.md` for user-visible product capabilities;
- `CHANGELOG.md` for release notes;
- `PROJECT_STRUCTURE.md` for source layout or architectural changes;
- `docs/consume-filters.md` for filter syntax or payload display/export behavior;
- `docs/avro.md` for Schema Registry/manual schema behavior;
- `docs/enhancement-roadmap.md` for roadmap status and prioritization;
- `docs/release.md` for release process changes.

Keep English and Korean README files aligned on version, feature list, document links, and security warnings.

## Safety notes

- Do not commit generated build outputs from `dist/`, `out/`, or `node_modules/`.
- The codebase memory artifact is written to `.codebase-memory/graph.db.zst`; commit it only when the team wants to share a fresh graph snapshot.
- Never remove user profiles or settings as part of refactoring.
- Do not weaken the legacy Kafka Tool settings migration path unless intentionally changing migration behavior.
- Be careful with Consumer Group Offset Reset and produce/replay flows; these can affect real Kafka clusters.

---
> Source: [pjhun0412/KafkaPilot](https://github.com/pjhun0412/KafkaPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
