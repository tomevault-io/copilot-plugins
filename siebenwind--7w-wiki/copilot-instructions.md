## 7w-wiki

> > **Canonical Entry Point for Autonomous Agents**

# 🤖 Siebenwind Wiki: Agent Protocols

> **Canonical Entry Point for Autonomous Agents**
> *Compatible with: OpenAI Codex, MCP-capable IDEs/CLIs, legacy Antigravity alias*

## 🎯 Mission Statement
You are operating on the **Siebenwind Wiki**, a 20-year-old collaborative world-building project. Your goal is to preserve its history while modernizing its infrastructure.
**Repository Root:** `.`

## 📜 The Golden Rules (Non-Negotiable)

1.  **Runtime Authority**: The **ONLY** executable interface is `./7w_wiki.py`. Do not create custom scripts or use `sed`/`awk` for complex logic.
2.  **Epistemic Integrity**:
    -   Never hallucinate lore. If information is missing, use tags like `[UNGEKLÄRT]`.
    -   **Epistemic precedence is `Homepage > Quellen > Wiki Pages`.**
    -   `docs/Siebenwind_Wiki/` is the technical edit tree and publishing-facing page tree, not the highest epistemic authority.
    -   Die kanonische Vollregel fuer Drift, Pages und Praezedenz steht in [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md).
3.  **Link Hygiene**:
    -   **NO** absolute paths (`file://`).
    -   Use `[[WikiLinks]]` for knowledge base articles.
    -   Use repo-relative paths (`System/...`) for documentation.
4.  **Coordination Central**:
    -   Every new system file MUST be registered in `System/COORDINATION_HUB.md`.
    -   Use `System/Synapse_Board/` for conflict resolution.
5.  **Agent Interop**:
    -   Respect the folder structure: `.agent/` is for internal logic.
    -   Treat `.agent/catalog/catalog.v1.json` as the canonical discovery catalog.
    -   Treat `lore_manifest.json` as a generated compatibility surface, never as the source of truth.
    -   Treat `.agents/skills/` plus `.codex/config.toml` as the Codex adapter surface, not as the source of truth.
    -   Use `.agent/config/tools.json` for machine-readable tool discovery (OpenAI-compatible schema).
    -   Use `./7w_wiki.py --help-json` for dynamic CLI introspection.
6.  **Mission Report Protocol**:
    -   Every session or task MUST end with a status report via `mail done` (if working on a specific message) or `mail post` (for general updates).
    -   Reports must be concise but include: What was done, what was verified, and what is next.
7.  **Inquisitive Protocol**:
    -   If you find an anomaly unrelated to your current task, **ASK**. Do not ignore it.
    -   Use `mail post` to query the `Coordinator` or `Technician`.
8.  **Machine-Readable First**:
    -   When available, agents MUST use CLI commands with `--json` (e.g., `advisor --json`, `audit --json`) for reliable parsing.
    -   Output parsing of human-readable text is discouraged if a JSON flag exists.
9.  **No Bridge-Placeholders by Default**:
    -   Fix links to canonical targets first.
    -   Do not ship generic bridge/stub pages as final repairs.
    -   Temporary bridge exceptions require lifecycle metadata (`bridge_mode`, `bridge_target`, `bridge_ticket`, `bridge_review_until`).

<!-- BEGIN GENERATED DRIFT CONTRACT REFERENCE -->
> Generated reference block. The surrounding narrative text remains manually maintained.
> Canonical contract: [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md)
>
> - Epistemic precedence: `Homepage > Quellen > Wiki Pages`.
> - `docs/Siebenwind_Wiki/` is the technical edit/publish tree, not the highest truth source.
> - Technical drift is validated via `./7w_wiki.py sanitize`, `./7w_wiki.py audit`, and `./7w_wiki.py pages validate --json [--strict-links]`.
> - Deterministic contract/CI checks use `./7w_wiki.py pages validate --contract --json`.
> - `--strict` hardens the MkDocs build; `--strict-links` is the hard unresolved-link gate.
> - Generated command registries are synced by `./7w_wiki.py tech --sync-docs` / `--sync-interop`; narrative rules live in the canonical contract.
<!-- END GENERATED DRIFT CONTRACT REFERENCE -->

## 🛠️ Command Registry (Executable Capabilities)

Use `./7w_wiki.py <command>` for all operations.


<!-- BEGIN GENERATED COMMAND REGISTRY -->
| Command | Purpose | Context |
| :--- | :--- | :--- |
| `search <query> [remaining...]` | Semantic RAG search (Oracle) across wiki and source corpus. | `.agent/skills/oracle/search.py` |
| `start [--run]` | Show or run the onboarding workflow. | `.agent/workflows/start.md` |
| `test` | Run interoperability and clean-state test suites. | `.agent/scripts/test_runner.py` |
| `takeover [--run]` | Show or run the takeover workflow. | `.agent/workflows/takeover.md` |
| `handover [--run]` | Show or run the handover workflow. | `.agent/workflows/handover.md` |
| `historian [query]` | Deep lore analysis workflow or direct topic run. | `.agent/workflows/historian.md` |
| `repair` | Interactive or automatic repair of audit findings, including Pages / Roamlinks fixes. | `.agent/scripts/repair.py` |
| `audit` | Run consistency audit for duplicates, broken links, and orphaned content. | `.agent/scripts/register_check.py` |
| `index` | Manage the semantic search index. | `.agent/skills/oracle/build_index.py` |
| `index-pages` | Generate category index pages for the wiki. | `.agent/scripts/generate_wiki_indices.py` |
| `pages <status|build|validate>` | Build or validate GitHub Pages documentation and site-integrity health. | `.agent/scripts/pages_tool.py` |
| `advisor` | Show system status and recommended next actions. | `System/Advisor` |
| `inquisition` | Run batch ingestion of legacy sources. | `.agent/scripts/inquisition.py` |
| `sanitize [target]` | Normalize structure, H1 usage, and frontmatter. | `.agent/scripts/wiki_sanitizer.py` |
| `lint [target]` | Run the combined lint pipeline. | `.agent/scripts/lint_tool.py` |
| `score <file>` | Calculate Lore Quality Score for one markdown file. | `.agent/scripts/lore_score_manager.py` |
| `ingest <file>` | Run the ingest pipeline for one file. | `.agent/scripts/ingest_pipeline.py` |
| `translate [args...]` | Translate Falandric texts or manage dictionaries. | `.agent/scripts/translator.py` |
| `watch` | Start the live watcher for index updates. | `.agent/scripts/watcher.py` |
| `package` | Build archive-first install bundles for supported platforms. | `.agent/scripts/package_tool.py` |
| `check [path]` | Run style and grammar checks. | `.agent/skills/lektor/style_checker.py` |
| `archive <sync|rotate|unpack>` | Manage archive symlinks, rotation, and unpack operations. | `docs/Archiv` |
| `mail <post|inbox|read|claim|done>` | Interact with the dispatch system using structured subcommands. | `System/Synapse_Board/SY_DISPATCH.md` |
| `scout` | Promoted discovery entrypoint for external source scanning. | `.agent/scripts/forum_scanner.py` |
| `tech` | Show the technician workflow or run interop maintenance helpers. | `.agent/workflows/tech_master.md` |
| `version` | Show or bump the wiki standard version. | `.agent/scripts/version_manager.py` |
| `antigravity` | Deprecated alias for start workflow overview. | `.agent/workflows/antigravity.md` |
| `leitpunkt <view|status|check|scaffold>` | Manage the human maintainer standpoint workflow. | `.agent/scripts/leitpunkt_tool.py` |
| `stats` | Generate reader-facing stats and machine snapshots. | `.agent/scripts/generate_wiki_stats.py` |
| `mcp` | Start the MCP server for structured agent access. | `System/MCP/server.py` |
<!-- END GENERATED COMMAND REGISTRY -->

## 📂 Documentation Map

-   **Governance**: [SY_INTEROP.md](System/Synapse_Board/SY_INTEROP.md) (Interop Standards)
-   **Drift-/Pages-Vertrag**: [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md)
-   **Coordination**: [COORDINATION_HUB.md](System/COORDINATION_HUB.md) (Registry)
-   **Operations Overview**: [AGENT_OPERATIONS_HANDBOOK.md](System/AGENT_OPERATIONS_HANDBOOK.md) (Agents, Skills, Workflows, Dispatch)
-   **Testing Protocol**: [SY_TESTING.md](System/Synapse_Board/SY_TESTING.md) (Suites, Defect-Flow, Agent Mentality)
-   **Workflow-CLI Bridge**: [SY_WORKFLOW_CLI_MATRIX.md](System/Synapse_Board/SY_WORKFLOW_CLI_MATRIX.md)
-   **Workflows**: `.agent/workflows/*.md` (Standard Operating Procedures)
-   **Personas**: `.agent/instructions/*.md` (Role definitions)

## 🧭 Adapter Surfaces

Canonical layer model:

- **Canonical core**: `.agent/` + `./7w_wiki.py`
- **Open runtime surface**: MCP via `./7w_wiki.py mcp` and `mcp_config.json`
- **Compatibility surface**: `lore_manifest.json`
- **Codex adapter surface**: `.agents/skills/` + `.codex/config.toml`
- **Future discovery surface**: `docs/.well-known/agent.json`

Practical meaning:
- Use `./7w_wiki.py start` as the canonical onboarding path. `antigravity` survives only as a deprecated compatibility alias.
- Use `.agents/skills/` for Codex-native discoverability, but treat those files as generated adapters derived from the canonical catalog.
- `/scout` remains the broad external discovery entrypoint; `/forum_search` is the dedicated operational path for board-first source discovery.

Maintainer note:
- Regenerate the canonical discovery and adapter surfaces with `./7w_wiki.py tech --sync-surfaces` or the full `./7w_wiki.py tech --sync-interop`.
- Use `./7w_wiki.py tech --repo-hygiene [--apply] [--json]` for conservative hot/cold/runtime/build cleanup and retention.
- `./7w_wiki.py tech --sync-bridges` remains only as a deprecated alias for old runbooks.

## 🚀 How to Work Here (Standard Loop)

1.  **Onboard**: Run `./7w_wiki.py start`, `./7w_wiki.py advisor`, and `./7w_wiki.py mail inbox --status OPEN` first. Read the latest `Logs/Archive/SESSION_MEMORY_*.md` before starting new work.
2.  **Plan**: Check `MASTER_TASK_LIST.md` and `task.md` (if available).
3.  **Execute**: Use `7w_wiki.py` tools. Do NOT edit `7w_wiki.py` unless assigned to "DevOps". Send status heartbeats via `mail post` on long tasks and route contradictions as specialist questions (question-first).
    Classify lore drift before editing: homepage drift, Quellen drift, or wiki-page drift. Reconcile against higher-precedence sources first; only then treat wiki-page edits as resolved.
    Bundle archives under `dist/` are release/build artifacts, not repo truth. Create them locally or via GitHub Releases, but do not commit them.
4.  **Verify**: Run `./7w_wiki.py audit`, `./7w_wiki.py test --suite clean-client-state`, `./7w_wiki.py test --suite interop-command-registry`, `./7w_wiki.py test --suite catalog-contract`, `./7w_wiki.py test --suite adapter-surfaces-contract`, `./7w_wiki.py test --suite delegation-policy-contract`, `./7w_wiki.py test --suite repo-hygiene-contract`, `./7w_wiki.py test --suite manifest-contract`, `./7w_wiki.py test --suite source-tree-contract`, `./7w_wiki.py test --suite legacy-doc-contract`, `./7w_wiki.py test --suite asset-surface-contract`, `./7w_wiki.py test --suite root-tree-retirement-contract`, `./7w_wiki.py test --suite styling-surface-contract`, `./7w_wiki.py test --suite workflow-matrix-contract`, `./7w_wiki.py test --suite tool-manifest-contract`, `./7w_wiki.py test --suite pages-contract-mode-contract`, `./7w_wiki.py test --suite bridge-placeholder-guard`, `./7w_wiki.py test --suite reader-stats-contract`, `./7w_wiki.py test --suite content-contract`, `./7w_wiki.py test --suite split-brain-guard`, and `./7w_wiki.py test --suite render-hygiene` before committing.
   If the published site or docs navigation changed, also run `./7w_wiki.py test --suite pages-full-smoke` plus `./7w_wiki.py pages validate --json` and align the result with [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md).
5.  **Log**: Update `CHANGELOG.md` or `Logs/` as appropriate. End each session with `Logs/Archive/SESSION_MEMORY_YYYY-MM-DD_<THEMA>.md` and reference it via `./7w_wiki.py mail post`.

## 🔎 Oracle Source Policy

For any non-trivial research, run the Oracle with explicit source scope:
- `--source wiki` for derived wiki pages in `docs/Siebenwind_Wiki/`
- `--source quellen` for raw source corpus
- `--source all` for combined cross-checking

Rule: when factual conflicts exist, Homepage and Quellen govern. Wiki pages are maintained derivatives and must be reconciled upward, not treated as the tie-breaker.

---

## 🔌 MCP Server (Model Context Protocol)

The MCP server is the **canonical live interface** for external agents and IDEs. It wraps the CLI as structured tools and also exposes the canonical catalog as resources.

- **Start**: `./7w_wiki.py mcp` (stdio) or `./7w_wiki.py mcp --transport streamable-http --port 7777` (network)
- **Auto-Discovery**: `mcp_config.json` at repo root for MCP-capable clients
- **Docs**: [System/MCP/README.md](System/MCP/README.md)
- **Generated typed tools** auto-generated from `--help-json` — zero maintenance
- **Catalog resources**: `wiki://catalog`, `wiki://workflows`, `wiki://skills`, `wiki://agents`
- **`wiki_mail_quip`**: You ARE encouraged to use this tool for in-character interagency commentary, humor, and personality. See `[QUIP]` tag in [SY_DISPATCH.md](System/Synapse_Board/SY_DISPATCH.md).

> **Dependency**: `pip install 'mcp[cli]'`

---
*Generated: 2026-02-19 | Standard: v1.2 (MCP-Enabled)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Siebenwind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
