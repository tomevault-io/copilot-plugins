## refactor-agent-skills-to-hierarchical-routing

> name: refactor-agent-skills-to-hierarchical-routing

---
name: refactor-agent-skills-to-hierarchical-routing
description: |
  Refactor flattened AI Agent Skills into an enterprise-grade contract of “Hierarchical Toolkit Routers + a Global Root Router”:
  - Global routing (Root Router): cross-domain intent dispatching, cross-toolkit pipeline orchestration, and state propagation
  - Type validation (a machine-verifiable subset of JSON Schema)
  - DAG dependency / cycle protection (including an interactive de-cycling contract + unattended fallback)
  - Parallel scheduling (governing both data dependencies + side effects / resource mutual exclusion)
  - Security redaction (declarative + enforced by external interceptors in the Host Environment)
  - Coverage conservation (merge/split allowed, but nothing may be dropped; verifiable)
  - Idempotency / retries / compensation (prevent retry-induced side-effect disasters)
  - Observability (mandatory inheritance of trace_id / span_id)
  Produce Step0–5 full YAML contracts and a verifiable lint.
schema_version: "2.2"
---

# Skill: Refactor to Hierarchical Router

## 0) Goals (MUST)

### 0.1 Toolkit Goals
- **Cohesion over Fragmentation**: If a cohesive domain (e.g., skills sharing a common prefix or purpose) contains ≤ 30 skills, group them into a SINGLE toolkit. Do NOT artificially split them just to create multiple toolkits.
- If the entire skill inventory is large and diverse, produce 2–5 `<domain>-toolkit` toolkits.
- Each toolkit MUST have ≤ 30 sub-skills; `misc-toolkit` ≤ 20.
- Each `<domain>-toolkit` MUST contain a `SKILL.md` file that acts as the routing entry point for that domain. The `description` field in its YAML is CRITICAL and must accurately reflect the intent of all its sub-skills to ensure the Host Agent routes to it correctly. Furthermore, its markdown body MUST contain a `sub_skills` index registry so the Agent knows exactly where to route.

### 0.2 Output Format & Verifiability
- Step0–5 outputs must each be **a single parseable YAML**
- Markdown may only be embedded via YAML block scalars (`|` or `>`)
- Must be parseable by `yaml.safe_load` (Step5 MUST verify)

### 0.3 Correctness & Safety
- Coverage conservation: must not omit any input skill (merge/split allowed but must be explicitly declared)
- Concurrency safety: parallelism requires BOTH “no data dependency” + “no side-effect/resource mutex conflict”
- Security redaction: logs/handoff/final_outputs must redact sensitive values
- Cycle protection: if a cycle exists, MUST output a `cycle_resolution` contract; and provide an **unattended fallback**
- Idempotency & retries: any skill that may produce side effects MUST declare `idempotent`; non-idempotent skills MUST NOT be retried unless compensation is provided

---

## 1) Hard Rules (MUST)

### 1.1 File Naming & Bundle Preservation (MUST)
- The toolkit root directory (`<domain>-toolkit/`) MUST contain its router file exactly named `SKILL.md`. It MUST NEVER be named `<domain>-toolkit.md`.
- You do NOT need an artificial `sub_skills/` folder. Sub-skills should be placed directly inside the toolkit directory as subfolders (e.g., `<domain>-toolkit/old_folder_name/`).
- **Host Loader Protection (Sub-skills only)**: Under the toolkit subfolders, the entry point `.md` for **sub-skills** MUST NOT be named `SKILL.md` or any reserved name (`ROUTER.md`, `INDEX.yaml`). This prevents naive Host File Loaders from using `glob("**/*.md")` and confusing sub-skills with master routers. Rename the entry point to match the folder name (e.g., `my_skill/SKILL.md` -> `my_skill/my_skill.md`).
- **Bundle Completeness**: If the original skill was a folder, the ENTIRE folder (including all scripts, schemas, and helper files) MUST be preserved and moved together.

### 1.2 File Encoding & Artifact Cleanup (MUST)
- **UTF-8 Encoding**: When reading or modifying files, the Agent MUST preserve file encoding. All file modifications MUST explicitly use UTF-8 encoding (without BOM) to prevent non-ASCII characters (like Chinese text) from becoming garbled.
- **Cleanup**: After refactoring, the Agent MUST clean up any temporary artifacts generated during the process (e.g., delete `step0-5-contracts.yaml` left in the toolkit directory, and delete any temporary migration scripts like `refactor_*.ps1` left in the `skills` directory).

### 1.3 Shared Resources & Path Rewriting (MUST)
- During Step 0 (Cleansing), the Agent MUST statically analyze the skill documents to discover any relative path references to external/shared folders (e.g., `../shared_utils/`).
- If shared resources exist, they MUST be migrated into the toolkit (or kept global if cross-toolkit), BUT they MUST keep their original folder name (e.g., if the folder was `shared_utils`, it stays `shared_utils` and is NOT renamed to `shared`).
- The Agent MUST safely rewrite any broken relative paths inside the migrated `.md` or code files to correctly point to the moved shared resources.

### 1.4 Cleansing (Minimum Viable)
Each skill MUST have at least:
- `name`
- `description` (must include: action + object + deliverable)
- `required_inputs[]`, `outputs[]` (if missing, fill them and mark `confidence: low`)
- If side effects are involved: MUST add `side_effects[]`, `resources_touched[]`, `mutex_group` (low-confidence inference is acceptable initially)

### 1.5 YAML Parseability (YAML-safe profile)
- Multi-line strings: MUST use block scalars `|` or `>`
- Single-line strings: add quotes when needed (avoid YAML ambiguity)
- **Router markdown MUST NOT contain fenced code blocks (```)**
  - If you must show code/contract fragments: use “4-space indented code blocks”
  - Reason: LLMs often create unparseable YAML when mixing ``` and indentation
- **Frontmatter Closure (MUST)**: When generating Markdown files with a YAML frontmatter (e.g. `toolkit_router_skill_md`), the YAML block MUST be strictly enclosed by `---` at BOTH the top and the bottom before the Markdown body begins.
- Step5 MUST verify: all step artifacts are parseable by `yaml.safe_load`

### 1.6 DAG + Concurrency Safety (Data Dependencies + Side-effect Mutex)
- A DAG edge exists if any of the following indicates dependency:
  - `depends_on`
  - `input_bindings(from_skill)`
  - Mutual-exclusion serialization: `mutex_group` conflicts OR `resources_touched` overlaps
- Parallelism is allowed only when all skills in the same parallel layer satisfy:
  - No DAG edges point to each other
  - No `mutex_group` conflicts
  - No overlap in `resources_touched` (matched by glob/prefix rules)

### 1.7 Security (Declarative + Host Delegation)
- Redaction scope: `logs`, `handoff`, `final_outputs`
- Redaction detection (declarative rules):
  - key-name contains (case-insensitive): token/secret/password/apikey/authorization/cookie/session/key
  - value-pattern (regex): JWT / PEM private keys / AWS keys / common tokens (extensible)
- Redacted output value: always `***MASKED***`
- **Allowed (and recommended) to delegate actual redaction to a Host Environment external interceptor**:
  - Skill/Router only declares `requires_redaction: true` and `sensitive_outputs`
  - Actual regex scanning/replacement is done by Host (more reliable for performance/accuracy/consistency)
- On task failure, MUST output:
  - `failure_report`
  - `manual_cleanup_recommendations`

### 1.8 Sub-skill Registry Fidelity (MUST)
- **100% Exact Match**: When generating the `sub_skills` registry inside a Router (e.g., `SKILL.md`), you MUST exactly copy the `description` content from the sub-skill's original YAML. Do not omit any characters.
- **No Summarization / Translation**: It is strictly forbidden to use AI models to summarize, shorten, or translate the description. Keep the original language exactly as it is (e.g., if the original is Chinese, keep Chinese; if English, keep English).
- **Safe YAML Block Scalar**: If the original `description` contains multiple lines of text, you MUST use safe block scalar symbols (e.g., `|` or `>`) within the Router's YAML array to ensure newlines do not break the entire document's YAML structure.

### 1.9 Toolkit Router Description Generation (Semantic Optimization, MUST)
When generating the master Router's `description` field (e.g., in Step 3), you must act as a Master Router Architect to maximize the semantic hit rate for LLM intent routing. You MUST ensure the description prevents both "semantic coverage gaps" and "structural blind spots".
1. **Extract**: Read all sub-skills' descriptions. List all unique action verbs (e.g., author, triage, review) and target entities.
2. **Cluster**: Group these into 3 to 5 logical capability domains.
3. **Draft**: Write the main `description` using this exact structure:
   - **Primary Identity**: 1-2 sentences explaining the overarching framework/purpose.
   - **Core Phases/Domains**: A numbered list of the 3-5 clustered domains, explicitly mentioning the extracted keywords.
   - **"Use When" Triggers**: A bulleted list of 3-4 explicit user intents that MUST route to this toolkit.
4. **Audit**: Ensure that *every single sub-skill* has at least one semantic hook (a related keyword or concept) present in your final parent description. If a sub-skill handles a specific task (e.g., "Legacy code extraction"), but the parent description only broadly mentions "New feature delivery", you must rewrite the parent description to explicitly include that specific semantic hook.

---

## 2) Type System (Machine-verifiable) (MUST)

### 2.1 JSON Schema Subset
I/O should use `schema` (recommended), or at minimum `schema.type` (enum):
- `string|integer|number|boolean|null|object|array`
- For objects: `properties|required|additionalProperties`
- For arrays: `items` is REQUIRED

### 2.2 Trace Context (Required for Observability)
All skill I/O **MUST support trace context inheritance**:
- Each skill’s `required_inputs` should include (at least optional) `trace_context`
- Each skill’s `outputs` should output (at least optional) `trace_context`

Recommended schema (can be referenced directly):
~~~yaml
TraceContext:
  type: "object"
  additionalProperties: false
  properties:
    trace_id: {type: "string", minLength: 8}
    span_id:  {type: "string", minLength: 8}
    parent_span_id: {type: "string"}
  required: ["trace_id","span_id"]
````

### 2.3 input_bindings (Explicit Binding)

If input names do not match, you MUST use:

```yaml
input_bindings:
  - target_input: "aws_config"
    from_skill: "fetch_profile"
    from_output: "profile_config"
```

---

## 3) Global State & Cross-Toolkit (MUST)

### 3.1 Global Context (Cross-domain State Contract)

If a pipeline crosses toolkits, you MUST explicitly pass state via `global_context`:

* `context_writes[]`: which keys a skill writes into global_context (with type declarations)
* `context_reads[]`: which keys a skill depends on (with type declarations)
* The Router MUST govern: missing keys → ask 1 question; or in batch mode → fail-fast

Recommended key naming:

* `infra.machine.ip`
* `deploy.release.id`
* `secrets.vault.ref` (only refs allowed; do NOT store plaintext secrets)

### 3.2 Artifact Registry (Coverage & Traceability)

The orchestrating Router should maintain an artifact registry (declarative):

* Any important output (reports, change tickets, deployment summaries) should have an `artifact_id`
* Enables auditing, reruns, reconciliation (coverage verification becomes easier)

---

## 4) Step 0–5 Output Contracts (ALL MUST be YAML)

> Free-form prose is forbidden as a substitute; any necessary explanation may only go under `notes: |`

### Step 0: Cleansing

```yaml
step: "0"
schema_version: "2.2"
warnings: []
auto_fixes: []
input_inventory:
  total_input_skills: 0
  input_skills:
    - old_path: "skills/x.md"
      detected_name: "..."
      detected_description: "..."
      io_inferred: true
      reserved_name_conflict: false
      confidence: "low"  # low|medium|high
```

### Step 1: Domain Grouping

```yaml
step: "1"
schema_version: "2.2"
target_toolkits_range: "2-5"
actual_toolkits_count: 0
toolkits:
  - toolkit: "devops-toolkit"
    verbs: ["deploy","monitor"]
    objects: ["service","infra"]
    skills: [{old_path: "skills/x.md"}]
constraints:
  per_toolkit_skill_limit: 30
  misc_skill_limit: 20
```

### Step 2: Migration Mapping + Coverage Conservation

```yaml
step: "2"
schema_version: "2.2"
migration_mapping:
  - old_path: "skills/x.md"
    new_path: "devops-toolkit/rollout_service/rollout_service.md"
    new_skill_id: "rollout_service"
    rename_reason: ""
    coverage:
      supersedes: ["skills/x.md"]  # merge: can be multiple
      split_into: []               # split: list new skill_id
coverage_report:
  total_input_skills: 0
  total_mapped_input_skills: 0
  uncovered_inputs: []
  coverage_ratio: 1.0
```

### Step 3: Generate Toolkit Routers (YAML-safe)

**Token truncation protection (toolkits > 3)**

* For toolkits: fully output only the 1st toolkit (index + router_md); output drafts for the rest + `pending_generation[]`

````yaml
step: "3"
schema_version: "2.2"

global_routing_spec:
  routing_mode: "deterministic_then_semantic"
  semantic_confidence_threshold: 0.72
  tie_breaker:
    mode: "ask_one_question_then_fallback"
    fallback_strategy:
      on_no_user_response: "fail_fast"   # fail_fast | break_lowest_confidence_edge
      default_max_wait_ms: 30000
  global_context:
    enabled: true
    mode: "explicit_reads_writes"
  security:
    redaction_mode: "external_interceptor_preferred"   # inline_allowed | external_interceptor_preferred | external_only
    mask_value: "***MASKED***"

toolkits_generated:
  - toolkit: "devops-toolkit"
    toolkit_index_yaml: |
      schema_version: "2.2"
      toolkit: "devops-toolkit"
      routing_mode: "deterministic_then_semantic"
      security:
        requires_redaction: true
        redaction_mode: "external_interceptor_preferred"
        mask_value: "***MASKED***"
        redaction_rules_declared:
          key_name_contains: ["token","secret","password","apikey","authorization","cookie","session","key"]
          value_patterns:
            - {name: "jwt", regex: "eyJ[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+"}
            - {name: "pem_private_key", regex: "-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----"}
            - {name: "aws_access_key_id", regex: "AKIA[0-9A-Z]{16}"}
      execution_defaults: {timeout_ms: 600000, max_retries: 2}
      observability:
        require_trace_context: true
      sub_skills:
        - id: "rollout_service"
          name: "Service Rolling Deployment"
          description: |
            Execute a rolling deployment of the specified service, producing deployment results and a summary.
          when: >
            The user requests deployment/rollout/rolling update and provides the service and environment.
          execution_policy:
            idempotent: false
            max_retries_override: 0
            requires_idempotency_key: true
            compensating_action_path: "rollback_rollout_service/rollback_rollout_service.md"  # optional
          required_inputs:
            - {name: "trace_context", schema: {type: "object", additionalProperties: true}}
            - {name: "service_name", schema: {type: "string", minLength: 1}}
            - {name: "environment", schema: {type: "string"}}
            - {name: "deploy_config", schema: {type: "object", additionalProperties: true}}
          required_permissions:
            - {name: "DEPLOY_EXECUTE", scope: "devops-toolkit", rationale: "Triggers deployment side effects"}
          outputs:
            - {name: "trace_context", schema: {type: "object", additionalProperties: true}}
            - {name: "deployment_report", schema: {type: "object", additionalProperties: true}}
          depends_on: []
          input_bindings: []
          context_reads: []
          context_writes:
            - {key: "deploy.release.id", schema: {type: "string"}}
          side_effects: ["network_call","infrastructure_change"]
          resources_touched: ["env://${environment}/service/${service_name}"]
          mutex_group: "deploy:${environment}:${service_name}"
          sensitive_outputs: ["deployment_report"]  # Host redactor should apply
          path: "rollout_service/rollout_service.md"

    toolkit_router_skill_md: |
      ---
      name: devops-toolkit-router
      description: |
        DevOps domain router: type validation, DAG/cycle protection, concurrency safety (incl. mutex governance), idempotency/retries/compensation, observability, and declarative security redaction.
      ---
      # DevOps Toolkit Router

      ## Available Sub-Skills (Routing Registry)
      You MUST route the user's intent to the appropriate sub-skill from the registry below.
      
          sub_skills:
            - id: "rollout_service"
              name: "Service Rolling Deployment"
              path: "rollout_service/rollout_service.md"
              description: "Execute a rolling deployment of the specified service, producing deployment results and a summary."

      ## MUST: Scheduling Rules
      1) Security gate: missing permissions/suspected injection → ask 1 clarification question; requires_redaction is enforced by the Host Interceptor.
      2) Type check: validate required_inputs with the JSON Schema subset; if invalid → ask 1 clarification question.
      3) Build DAG: add edges from depends_on + input_bindings + mutex/resources serialization; if cycle → enter cycle_resolution.
      4) Parallel plan: parallelize only when there are no data edges + no mutex conflicts + no resource overlaps; output parallel_plan.levels (YAML).
      5) Idempotency/Retry: idempotent=false → max_retries=0; if compensating_action exists, allow “compensate after failure”.
      6) Observability: trace_context must be inherited end-to-end (trace_id/span_id).

      ## MUST: Cycle Resolution Contract (indented block, forbid ```)
          cycle_resolution:
            cycle_detected: true
            cycle_edges:
              - {from: "a", to: "b", reason: "input_binding"}
            resolution_question: "Please specify the order: A first or B first?"
            resolution_options:
              - {option_id: "A_then_B", remove_edge: {from: "b", to: "a"}}
              - {option_id: "B_then_A", remove_edge: {from: "a", to: "b"}}
            fallback_strategy:
              on_no_user_response: "fail_fast"
              default_max_wait_ms: 30000

toolkits_index_drafts: []
pending_generation: []
notes: |
  If pending_generation is non-empty, it means the remaining toolkits still need full index + router_md outputs.
````

### Step 4: Dry-run (3 Cases)

```yaml
step: "4"
schema_version: "2.2"
test_cases:
  - name: "Happy Path"
    user_intent: "Roll out service-a on staging"
    expected: {toolkit: "devops-toolkit", skill_id: "rollout_service"}
  - name: "Complex Pipeline (Cross-Toolkit)"
    user_intent: "Provision a machine first, then deploy service-a to staging, and finally report the IP and deployment summary"
    expected_pipeline:
      toolkits_involved: ["infra-toolkit","devops-toolkit"]
      context_flow:
        - {write: "infra.machine.ip", produced_by: "infra-toolkit/provision_machine"}
        - {read:  "infra.machine.ip", consumed_by: "devops-toolkit/rollout_service"}
    constraints:
      require_global_context: true
      require_trace_context: true
  - name: "Security Path"
    user_intent: "Use this token to deploy prod: AKIA...."
    expected_behavior:
      ask_one_question: true
      redaction_applied: true
      redaction_mode: "external_interceptor_preferred"
      mask_value: "***MASKED***"
```

### Step 5: Lint (If coverage_ratio < 1.0, MUST self-repair)

````yaml
step: "5"
schema_version: "2.2"
lint_report:
  coverage_ratio: 1.0
  uncovered_inputs: []
  toolkits_sub_skills_limit_ok: true
  misc_toolkit_limit_ok: true
  no_nested_master_ok: true
  yaml_parseable_ok: true
  yaml_safe_profile_ok: true
  type_system_valid_ok: true
  dependency_graph_acyclic_ok: true
  cycle_fallback_present_ok: true
  concurrency_safe_ok: true
  idempotency_policy_ok: true
  observability_trace_context_ok: true
  security_redaction_rules_declared_ok: true
  security_redaction_delegation_ok: true
violations: []
auto_repairs_applied: []
notes: |
  yaml_safe_profile_ok MUST check: router_md must NOT contain fenced code blocks (```).
````

---

## 5) Sub-skill Index Template (Minimum Viable, MUST)

```yaml
schema_version: "2.2"
toolkit: "<domain>-toolkit"
routing_mode: "deterministic_then_semantic"

observability:
  require_trace_context: true

security:
  requires_redaction: true
  redaction_mode: "external_interceptor_preferred"
  mask_value: "***MASKED***"
  redaction_rules_declared:
    key_name_contains: ["token","secret","password","apikey","authorization","cookie","session","key"]
    value_patterns:
      - {name: "jwt", regex: "eyJ[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+"}

execution_defaults: {timeout_ms: 600000, max_retries: 2}

sub_skills:
  - id: "verb_object_qualifier"
    name: "English Display Name"
    description: |
      Action + object + deliverable.
    when: >
      Trigger scenario.
    execution_policy:
      idempotent: true
      max_retries_override: 2
      requires_idempotency_key: false
      compensating_action_path: ""   # optional
    required_inputs:
      - {name: "trace_context", schema: {type: "object", additionalProperties: true}}
      - {name: "x", schema: {type: "string"}}
    required_permissions:
      - {name: "PERM", scope: "<domain>-toolkit", rationale: "Reason"}
    outputs:
      - {name: "trace_context", schema: {type: "object", additionalProperties: true}}
      - {name: "y", schema: {type: "object", additionalProperties: true}}
    depends_on: []
    input_bindings: []
    context_reads: []
    context_writes: []
    side_effects: ["filesystem_write"]
    resources_touched: ["file://${x}"]
    mutex_group: "file:${x}"
    sensitive_outputs: ["y"]
    path: "example/example.md"
```

---
> Source: [walter-la/refactor-agent-skills-to-hierarchical-routing](https://github.com/walter-la/refactor-agent-skills-to-hierarchical-routing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
