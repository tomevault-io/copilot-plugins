## logicpearl

> LogicPearl separates deterministic policy logic from explanation metadata.

# Agent Guidance

LogicPearl separates deterministic policy logic from explanation metadata.

## Feature Dictionaries

Use a feature dictionary before discovery when raw feature IDs need reviewer-facing text:

```bash
logicpearl build traces.csv \
  --feature-dictionary feature_dictionary.json \
  --output-dir /tmp/pearl
```

The dictionary is embedded into the artifact as `input_schema.features[].semantics`. Generated rule labels, messages, counterfactual hints, `inspect`, and `diff` may use it. Runtime evaluation must ignore it.

Minimal dictionary shape:

```json
{
  "feature_dictionary_version": "1.0",
  "features": {
    "feature_id": {
      "label": "Readable feature label",
      "source_id": "optional-source-id",
      "source_anchor": "optional-source-anchor",
      "states": {
        "missing_or_failed": {
          "when": {
            "op": "<=",
            "value": 0.0
          },
          "label": "Readable state label",
          "message": "This rule fires when the readable condition is true.",
          "counterfactual_hint": "Describe the smallest useful change."
        }
      }
    }
  }
}
```

For simple CSV work, `label` alone is enough. Use `states` only when a specific predicate needs precise reviewer-facing text.

Do not fix unreadable output by patching `rules[].label` after discovery or by rewriting labels in a UI. Generate a dictionary from the same source that generated the traces, then pass it to `build` or `discover`.

Do not add healthcare, payer, or other domain-specific parsing to the core crates. The core should not parse prefixes like `requirement__`, suffixes like `__satisfied`, or IDs like `req-003`. Domain integrations own those meanings and should express them through feature dictionary fields.

Proof checklist before claiming feature dictionaries work:

- Build with `logicpearl build ... --feature-dictionary feature_dictionary.json`.
- Inspect the emitted `pearl.ir.json` and confirm `input_schema.features[].semantics` exists for dictionary-backed features.
- Confirm generated `rules[].label`, `rules[].message`, and `rules[].counterfactual_hint` came from LogicPearl rule generation, not a post-build patch or frontend rewrite.
- Run `logicpearl inspect <artifact> --json` and confirm readable feature metadata appears alongside raw `deny_when`.
- Run the artifact with and without dictionary metadata when relevant and confirm runtime bitmasks are identical for identical raw rules.

When reviewing diffs, keep these separate:

- `source_schema_changed`: features or source anchors changed
- `learned_rule_changed`: raw rule expressions changed
- `rule_explanation_changed`: labels, messages, hints, or dictionary text changed while raw logic stayed the same

Diff expectation:

- `logicpearl diff old_artifact new_artifact --json` should expose readable rule metadata when dictionaries are present.
- Raw expression changes should be reported separately from explanation-only changes.
- Adding or improving dictionary text should not be described as a learned policy change unless the raw `deny_when` changed.

Downstream demos and frontends may display artifact metadata, but they must not synthesize, infer, or rewrite rule meaning. The artifact should already contain the human-facing label, message, and counterfactual hint.

Raw `deny_when` expressions are the source of deterministic truth.

## Artifact Manifests

`artifact.json` is a public bundle contract, not only a CLI pointer. New gate,
action, and composed pipeline bundles should emit
`schema_version: "logicpearl.artifact_manifest.v1"`.

The v1 manifest should identify the artifact with `artifact_id`,
`artifact_kind` (`gate`, `action`, or `pipeline`), `engine_version`,
`ir_version`, `created_at`, and the deterministic `artifact_hash`. Use
`files.ir` as the canonical pointer to `pearl.ir.json` or a pipeline JSON file.
Optional sidecars belong under `files.build_report`, `files.feature_dictionary`,
`files.native`, `files.wasm`, and `files.wasm_metadata`.

Use `logicpearl artifact inspect <artifact> --json`,
`logicpearl artifact digest <artifact> --json`, and
`logicpearl artifact verify <artifact> --json` when checking bundle behavior.
`artifact verify` should validate the manifest version, referenced files,
file hashes, artifact hash, IR kind, and input schema hash. Keep old manifest
aliases readable for compatibility, but prefer the v1 field names in new code,
tests, and docs.

## Build Provenance

Build reports should carry `provenance.schema_version:
"logicpearl.build_provenance.v1"`. Provenance belongs in core build output, not
downstream demos, because it is the audit trail for where an artifact came from.

Record stable hashes and bounded metadata: `engine_version`, optional
`engine_commit`, redacted CLI command args, redacted build options plus
`build_options_hash`, trace input hashes and row counts, feature dictionary hash,
plugin run provenance, optional source manifest hash, limited environment facts,
and generated artifact file hashes.

Do not store raw environment variables, full PATH values, hostnames, usernames,
home directories, raw plugin stdout/stderr, raw source documents, or raw
free-form provenance values. Hash plugin boundaries and redact plugin option,
source-reference, and build-option values by default; only allowlist operational
fields with low sensitivity such as `label_column`, `action_column`, and
`dialect`.

`build_report` and `artifact.json` should not be self-hashed inside
`build_report.provenance.generated_files`; use the artifact manifest file hashes
and `logicpearl artifact verify` for bundle integrity.

## Source Manifests

`logicpearl build` accepts `--source-manifest sources.json` as a generic
provenance input. The source manifest is not a feature dictionary and must not
change discovery, runtime evaluation, labels, bitmasks, or rule text.

Expected shape:

```json
{
  "schema_version": "logicpearl.source_manifest.v1",
  "sources": [
    {
      "source_id": "policy_manual_2026_04",
      "kind": "manual_policy",
      "title": "April policy manual",
      "uri": "s3://example/policy_manual_2026_04.pdf",
      "retrieved_at": "2026-04-12T14:30:00Z",
      "content_hash": "sha256:0000000000000000000000000000000000000000000000000000000000000000",
      "data_classification": "customer_confidential"
    }
  ]
}
```

Core should validate the v1 shape, preserve `sources[]`, hash the manifest, and
attach it to `build_report.provenance.source_manifest`. Do not fetch `uri`,
parse PDFs, read remote source content, interpret `source_id`, or add
domain-specific, customer-specific, or benchmark-specific behavior to core.
Domain integrations own source semantics and should express links through
feature dictionary `source_id` / `source_anchor` fields and source manifests.

## Multi-Action Policies

LogicPearl can learn a policy that chooses one action from reviewed examples, not just a yes/no gate. Use this when a trace dataset has an action column such as `next_action`.

```bash
logicpearl build traces.csv \
  --action-column next_action \
  --default-action do_nothing \
  --output-dir /tmp/actions
```

The build writes the same familiar bundle shape: `artifact.json`, `pearl.ir.json`, and an action report. The `pearl.ir.json` file contains the learned action policy, including the available actions, the default action, and the rules that point to each action.

Use `--compile` when the action policy should be deployed like any other pearl:

```bash
logicpearl build traces.csv \
  --action-column next_action \
  --default-action do_nothing \
  --output-dir /tmp/actions \
  --compile
```

Compiled action builds emit a native runner. When the local Rust `wasm32-unknown-unknown` target is installed, they also emit `pearl.wasm` and `pearl.wasm.meta.json` for browser/runtime use.

Action traces can also come from a `trace_source` plugin. The plugin should return flat scalar records, or rows shaped as `features` plus the action column:

```bash
logicpearl build \
  --trace-plugin-manifest plugins/revenue_pack_trace_source/manifest.json \
  --trace-plugin-input va_tricare_notification_readiness \
  --action-column next_action \
  --feature-dictionary feature_dictionary.json \
  --output-dir artifacts/revenue_packs/va_tricare_notification_readiness/actions
```

`logicpearl inspect /tmp/actions` should read like a reviewable policy:

```text
Action rules:
  1. water
     Soil Moisture at or below 18% and Water used in the last 7 days at or below 0.2
```

At runtime, LogicPearl evaluates all matching rules into a bitmask, then selects the action from those matched rules. If no rule matches, it returns the configured default action.

```bash
logicpearl run /tmp/actions today.json --json
```

Action learning is priority-aware. By default, non-default actions are learned in support-based order, with rarer actions first and first-seen order as the tie breaker. Use `--action-priority block,redact` when the domain has an explicit high-to-low priority order. Earlier actions learn first, then lower-priority actions learn against residual rows not already captured by earlier routes. Use `--action-max-rules N` to cap the total emitted non-default action rules; when omitted, LogicPearl scales per-action budgets from trace support.

The JSON result includes the artifact/policy id, `decision_kind: "action"`, selected `action`, matched-rule `bitmask`, `defaulted`, `ambiguity`, selected and matched rules, and the feature dictionary semantics for the matched rule predicates.

Gate runtime JSON follows the same pattern for the fields that apply to gates: artifact/policy id, `decision_kind: "gate"`, `allow`, `bitmask`, matched rules, and source-grounded feature explanations.

`logicpearl diff` understands action-policy bundles. Its JSON summary separates action set changes, default action changes, rule predicate changes, rule priority changes, source/schema changes, and explanation-only changes.

## Runtime JSON Schemas

Runtime JSON is a public engine contract. Gate, action, pipeline, rule explanation, feature explanation, runtime result, and artifact error schemas live under `schema/` as versioned `logicpearl.*.v1` contracts.

Every v1 runtime result emitted by `logicpearl run --json` or `logicpearl pipeline run --json` should include:

- `schema_version`
- `engine_version`
- `artifact_id`
- `artifact_hash`
- `decision_kind`

Gate results use `schema_version: "logicpearl.gate_result.v1"`. Action results use `schema_version: "logicpearl.action_result.v1"`. Pipeline results use `schema_version: "logicpearl.pipeline_result.v1"`.

Do not hand-patch downstream demos or frontends to infer result meaning. The engine result should already contain the stable IDs, bitmask, selected or matched rules, and explanation metadata. Browser code that needs the exact wire shape should use `evaluateJson()` from `@logicpearl/browser`; app code may still use the ergonomic BigInt-returning `evaluate()` API.

Compatibility policy for v1:

- Additive fields are allowed.
- Removing fields, renaming fields, changing field meaning, or changing scalar encoding requires a v2 schema.
- Plugin stage `raw_result` values may remain plugin-defined JSON unless the plugin contract supplies its own schema.

Proof checklist before claiming runtime JSON schemas work:

- Run `logicpearl run <gate> <input> --json` and validate the result against `schema/logicpearl-gate-result-v1.schema.json`.
- Run `logicpearl run <action_artifact> <input> --json` and validate the result against `schema/logicpearl-action-result-v1.schema.json`.
- Run `logicpearl pipeline run <pipeline> <input> --json` and validate the result against `schema/logicpearl-pipeline-result-v1.schema.json`.
- Confirm golden fixtures under `fixtures/runtime/` validate against the committed schemas.
- Confirm browser `evaluateJson()` returns schema-shaped snake_case JSON, while `evaluate()` still returns the browser-friendly BigInt shape.

## Plugin And Pipeline Execution

Plugin and pipeline manifests can execute local processes. Treat manifests from other repos, issues, or generated examples as untrusted unless the user explicitly says they trust them.

Default process-plugin behavior is conservative: timeouts are applied, manifest-relative scripts are allowed, and risky absolute or PATH-based entrypoints require explicit opt-ins. Do not weaken those defaults in code or docs without making the trust boundary explicit.

Plugin execution provenance should use
`schema_version: "logicpearl.plugin_run_provenance.v1"` when a plugin run is
recorded in build reports, plugin command JSON, or plugin-backed pipeline stage
results. Capture stable audit fields: `plugin_run_id`, `plugin_id`,
`plugin_version`, plugin name, stage, protocol version, manifest path/hash,
resolved entrypoint plus `entrypoint_hash`, input/request/output hashes,
trace/enricher `rows_emitted` when known, started/completed timestamps,
duration, timeout policy, execution policy, declared/allowed/enforced
capabilities, access posture, and stdout/stderr byte counts plus redacted hash
summaries.

Do not claim sandbox enforcement that the process runner does not provide.
Until there is a real OS/container sandbox, record network as `not_enforced`,
filesystem as `process_default`, and enforcement as `none`. This is provenance
and execution-policy transparency, not proof that plugins lacked access.

Production plugin manifests should declare stable `plugin_id` and
`plugin_version` fields. Keep them optional for compatibility with older plugin
manifests, and fall back to `name` only when `plugin_id` is absent.

---
> Source: [LogicPearlHQ/logicpearl](https://github.com/LogicPearlHQ/logicpearl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
