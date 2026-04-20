## dagster-sqlmesh

> Architecture Patterns for dg-sqlmesh module


# Architecture Patterns for dg-sqlmesh

## Individual Asset Pattern

### Overview

Each SQLMesh model becomes a separate Dagster asset for granular control and better UI experience.

### Implementation

```python
# In [src/dg_sqlmesh/factory.py](mdc:src/dg_sqlmesh/factory.py)
def create_model_asset(current_model_name, current_asset_spec, current_model_checks):
    @asset(
        key=current_asset_spec.key,
        description=f"SQLMesh model: {current_model_name}",
        group_name=current_asset_spec.group_name,
        metadata=current_asset_spec.metadata,
        deps=current_asset_spec.deps,
        check_specs=current_model_checks,
        tags={
            **(current_asset_spec.tags or {}),
            "dagster/max_retries": "0",
            "dagster/retry_on_asset_or_op_failure": "false"
        },
    )
    def model_asset(context: AssetExecutionContext, sqlmesh: SQLMeshResource, sqlmesh_results: SQLMeshResultsResource):
        # Shared execution logic
        pass
```

### Benefits

- **Granular Control**: Each model can succeed/fail independently
- **Better UI**: Individual assets visible in Dagster UI
- **Selective Materialization**: Can materialize specific models
- **Proper Dependencies**: Dagster's native dependency resolution

## Shared SQLMesh Execution Pattern

### Overview

Single SQLMesh execution per Dagster run, shared between all selected assets via `SQLMeshResultsResource`.

### Implementation

```python
class SQLMeshResultsResource(ConfigurableResource):
    """Resource pour partager les résultats SQLMesh entre les assets d'un même run."""

    def store_results(self, run_id: str, results: Dict[str, Any]) -> None:
        """Stocke les résultats SQLMesh pour un run donné."""
        self._results[run_id] = results

    def get_results(self, run_id: str) -> Optional[Dict[str, Any]]:
        """Récupère les résultats SQLMesh pour un run donné."""
        return self._results.get(run_id)

    def has_results(self, run_id: str) -> bool:
        """Vérifie si des résultats existent pour un run donné."""
        return run_id in self._results
```

### Execution Logic

```python
# Check if SQLMesh already executed for this run
if not sqlmesh_results.has_results(context.run_id):
    # First asset in run - execute SQLMesh for all selected assets
    selected_asset_keys = context.selected_asset_keys
    models_to_materialize = get_models_to_materialize(selected_asset_keys, ...)
    plan = sqlmesh.materialize_assets_threaded(models_to_materialize)

    # Store results for other assets
    results = {
        "failed_check_results": sqlmesh._process_failed_models_events(),
        "skipped_models_events": sqlmesh._console.get_skipped_models_events(),
        "evaluation_events": sqlmesh._console.get_evaluation_events(),
    }
    sqlmesh_results.store_results(context.run_id, results)
else:
    # Use existing results from this run
    results = sqlmesh_results.get_results(context.run_id)
```

### Benefits

- **Respects SQLMesh Dependencies**: Natural upstream/downstream skipping
- **Single Execution**: One SQLMesh run per Dagster run
- **Consistent State**: All assets see same SQLMesh results
- **Better Performance**: Single SQLMesh context and execution

## Event-Driven Status Pattern

### Overview

Asset and check status determined from SQLMesh events captured by custom console.

### Implementation

```python
# In [src/dg_sqlmesh/sqlmesh_event_console.py](mdc:src/dg_sqlmesh/sqlmesh_event_console.py)
class SQLMeshEventCaptureConsole(IntrospectingConsole):
    def _handle_log_failed_models(self, event: LogFailedModels) -> None:
        """Capture failed model events for asset check creation."""
        self.failed_models_events.append(event)

    def _handle_log_skipped_models(self, event: LogSkippedModels) -> None:
        """Capture skipped model events for dependency logic."""
        self.skipped_models_events.append(event)

    def _handle_update_snapshot_evaluation(self, event: UpdateSnapshotEvaluationProgress) -> None:
        """Capture evaluation progress for audit results."""
        self.evaluation_events.append(event)
```

### Status Determination

```python
# Check if model was skipped due to upstream failure
if model_was_skipped:
    raise Exception(f"Model {model_name} was skipped due to upstream failures")

# Check if model materialized but audits failed
elif model_has_audit_failures:
    # Asset materializes successfully but with failed checks
    return MaterializeResult(
        asset_key=asset_key,
        check_results=[AssetCheckResult(passed=False, ...)]
    )
else:
    # Full success - materialization and audits passed
    return MaterializeResult(
        asset_key=asset_key,
        check_results=[AssetCheckResult(passed=True, ...)]
    )
```

## Tag Convention Pattern

### Overview

Use `dagster:property_name:value` convention to pass SQLMesh model properties to Dagster assets.

### Implementation

```python
# In [src/dg_sqlmesh/translator.py](mdc:src/dg_sqlmesh/translator.py)
def _get_dagster_property_from_tags(self, model, property_name: str) -> Optional[str]:
    """
    Parse SQLMesh tags to extract Dagster properties.
    Convention: "dagster:property_name:value"
    """
    tags = getattr(model, "tags", set())

    for tag in tags:
        if tag.startswith("dagster:"):
            parts = tag.split(":")
            if len(parts) >= 3 and parts[1] == property_name:
                return parts[2]

    return None
```

### Usage

```sql
-- SQLMesh model with Dagster properties
MODEL (
    name my_model,
    tags [
        "dagster:group_name:custom_group",
        "dagster:owner:data_team",
        "dagster:description:Custom model description",
        "production",  -- Regular SQLMesh tag
        "critical"     -- Regular SQLMesh tag
    ]
)
```

## No Retry Pattern

### Overview

Force no retries via Dagster tags to prevent infinite loops on persistent SQLMesh audit failures.

### Implementation

```python
# In asset definition
@asset(
    # ... other parameters
    tags={
        **(current_asset_spec.tags or {}),
        "dagster/max_retries": "0",
        "dagster/retry_on_asset_or_op_failure": "false"
    },
)
def model_asset():
    pass

# In job definition
sqlmesh_job = define_asset_job(
    name="sqlmesh_job",
    selection=sqlmesh_assets,
    tags={
        "dagster/max_retries": "0",
        "dagster/retry_on_asset_or_op_failure": "false"
    }
)
```

### Benefits

- **Prevents Infinite Loops**: No retries on persistent SQLMesh audit failures
- **Module-Level Control**: Works regardless of user configuration
- **Resource Efficient**: No wasted CPU/memory on failed retries
- **Clear UI**: Single failed run instead of multiple retries

## Version Mapping Pattern

### Overview

Map SQLMesh `data_hash` to Dagster `code_version` and SQLMesh snapshot version to Dagster `data_version`.

### Implementation

```python
# Code version from SQLMesh data_hash
code_version = str(getattr(model, "data_hash", "")) if hasattr(model, "data_hash") and getattr(model, "data_hash") else None

# Data version from SQLMesh snapshot version
snapshot = assetkey_to_snapshot.get(asset_key)
if snapshot:
    snapshot_version = getattr(snapshot, "version", None)
    data_version = DataVersion(str(snapshot_version)) if snapshot_version else None
```

### Benefits

- **Proper Sync Status**: Dagster assets show correct sync status
- **Avoid Unnecessary Re-materializations**: Only re-materialize when code or data actually changes
- **SQLMesh-Native**: Uses SQLMesh's existing versioning system

---

description:
globs:
alwaysApply: false

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fosk06) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
