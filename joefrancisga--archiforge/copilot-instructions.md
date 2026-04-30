## archiforge

> Where to start when extending ArchLucid (runs, agents, authority, connectors, alerts).


# ArchLucid navigation (quick map)

## Read first

- **Contributor decision tree (what folder to open first)**: `docs/START_HERE.md` → section **What to open first (contributor decision tree)** (Mermaid flowchart + table by change type).
- **System flows (Mermaid)**: `docs/SYSTEM_MAP.md`
- **Component detail**: `docs/ARCHITECTURE_COMPONENTS.md`
- **API contracts**: `docs/API_CONTRACTS.md`

## Entry points

| Task | Start here |
|------|------------|
| Add a **finding engine** | `ArchLucid.Decisioning/Interfaces/IFindingEngine.cs` → register in `ServiceCollectionExtensions.Decisioning.cs` → template `templates/archlucid-finding-engine/` |
| Add a **context connector** | `ArchLucid.ContextIngestion/Interfaces/IContextConnector.cs` → `ContextConnectorPipeline.CreateOrderedContextConnectorPipeline` → `RegisterContextIngestionAndKnowledgeGraph` (Api startup) |
| Add an **agent handler** | `ArchLucid.AgentRuntime/IAgentHandler.cs` → `RegisterAgentExecution` in `ServiceCollectionExtensions.AgentsGovernanceRetrieval.cs` |
| Add an **alert delivery channel** | `ArchLucid.Decisioning/Alerts/Delivery/IAlertDeliveryChannel.cs` → register with alert dispatcher |
| Change **authority pipeline** | `ArchLucid.Persistence/Orchestration/AuthorityRunOrchestrator.cs` and `Orchestration/Pipeline/*` stages |
| **SQL + RLS + replica** | `ArchLucid.Persistence/Connections/SqlServerOptions.cs` (section `SqlServer`) |
| **Configuration validation** | `ArchLucid.Host.Core/Startup/Validation/ArchLucidConfigurationRules.cs` |

## Data vs persistence

- **`ArchLucid.Persistence.Data.*`**: Dapper repositories against SQL, connection factories, migrations scripts consumption.
- **`ArchLucid.Persistence`**: Domain-oriented persistence ports (`IRunRepository`, UoW, orchestration, caching decorators, archival).

## Product rename (in progress)

- **Rename checklist**: `docs/ARCHLUCID_RENAME_CHECKLIST.md` — ArchiForge → ArchLucid, done incrementally per `.cursor/rules/ArchLucid-Rename.mdc`.

## Host roles

- **`Hosting:Role`**: `Combined` (default dev), `Api` (HTTP only), `Worker` (background services). See `ArchLucid.Host.Core/Hosting/ArchLucidHostingRole.cs`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joefrancisGA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
