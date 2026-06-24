## longue-vue

> > **Scope:** This file is for an AI agent helping a human **operator** *use* a

# AGENTS.md

> **Scope:** This file is for an AI agent helping a human **operator** *use* a
> running longue-vue instance — answering questions about inventory, risk, and
> classification over the **read-only** MCP interface. It is **not** a
> development guide. Agents working on the codebase should read `CLAUDE.md` and
> `docs/adr/` instead.

## What longue-vue is

longue-vue is a Kubernetes + cloud-VM **CMDB** aligned with ANSSI
**SecNumCloud** (ADR-0001). It inventories clusters, nodes, workloads, pods,
storage, cloud VMs, and the **Applications** that group them, and enriches them
with end-of-life (EOL) and image-version signal. Your job as an agent: answer
an operator's questions about **what exists, what's at risk, what depends on
what, and how it's classified**. You have **read-only** access — there are no
mutating tools; never claim to have changed anything.

## Connecting (MCP)

longue-vue exposes a Model Context Protocol server (ADR-0014). Connection
facts (defaults from `cmd/longue-vue/main.go`):

- **Enablement:** tool calls are gated by the `mcp_enabled` setting. An admin
  toggles it in the UI, or it is seeded at boot from `LONGUE_VUE_MCP_ENABLED`.
  If every tool returns "disabled", MCP is off — ask the operator to enable it.
- **Endpoint:** `LONGUE_VUE_MCP_ADDR`, default `127.0.0.1:8090`.
- **Transport:** `LONGUE_VUE_MCP_TRANSPORT`, default `sse`.
- **Auth:** present a Personal Access Token (`longue_vue_pat_…`) as the bearer
  token. SSE validates it on every call. The token needs **read** scope (admin
  implies read); `vm-collector` tokens are rejected (ADR-0015 §5).
- **Transport security:** TLS via `LONGUE_VUE_MCP_TLS_CERT`/`_KEY`; plaintext
  is refused unless `LONGUE_VUE_MCP_ALLOW_PLAINTEXT=true`.

**Troubleshooting** an auth error or empty results — check, in order:
1. Is `mcp_enabled` on?
2. Does the token carry **read** scope and is it not a `vm-collector` token?
3. Do the endpoint and transport match the server's (`addr`, `sse`/TLS)?

## Domain model primer

Enough to map a question to the right tool — follow the ADR for depth.

- **Kubernetes FK chain:** `clusters` → `namespaces` / `nodes` /
  `persistent_volumes` → `workloads` / `pods` / `services` / `ingresses` /
  `persistent_volume_claims`. Pods resolve their owning workload via
  `ownerReferences`.
- **VMs:** `virtual_machines` is a **separate top-level table** from `nodes`
  (cloud VMs, not K8s nodes), grouped under `cloud_accounts` (ADR-0015).
- **Applications (ADR-0029):** the first-class SecNumCloud applicative entity.
  Hierarchy: **Application Block → Application → workloads / VMs**. Each asset
  links to at most one Application.
- **EOL signal (ADR-0012 / 0019 / 0022):** clusters, nodes, and VMs carry
  `longue-vue.io/eol.*` annotations from endoflife.date; workloads derive EOL
  from per-container **image-versions** enrichment (behind-latest = outdated).
- **DICT classification (ADR-0029):** the SecNumCloud security-need axes
  Disponibilité / Intégrité / Confidentialité / Traçabilité (`sec_*`, 0–4) live
  **on the Application**. Linked workloads/VMs surface a read-only
  `effective_dict` (application-wins).

## Available tools

All read-only. UUIDs come from prior `list_*` results — never invent them.

### Inventory

| Tool | Answers | Key filters |
|---|---|---|
| `list_clusters` / `get_cluster` | What clusters exist / one cluster | `name`; `id` |
| `list_nodes` / `get_node` | Nodes in a cluster / one node | `cluster_id`; `id` |
| `list_namespaces` / `get_namespace` | Namespaces / one namespace | `cluster_id`; `id` |
| `list_workloads` / `get_workload` | Workloads / one workload | `namespace_id`, `kind`, `image`; `id` |
| `list_pods` / `get_pod` | Pods / one pod | `namespace_id`, `node_name`, `workload_id`, `image`; `id` |
| `list_services` | Services in a namespace | `namespace_id` |
| `list_ingresses` | Ingresses in a namespace | `namespace_id` |
| `list_persistent_volumes` | PVs in a cluster | `cluster_id` |
| `list_persistent_volume_claims` | PVCs in a namespace | `namespace_id` |

### VMs & cloud

| Tool | Answers | Key filters |
|---|---|---|
| `list_virtual_machines` / `get_virtual_machine` | Cloud VMs / one VM | `cloud_account_id`/`_name`, `region`, `role`, `power_state`, `name`, `image`, `application`, `application_version`, `include_terminated`; `id` |
| `list_cloud_accounts` / `get_cloud_account` | Cloud accounts / one account | `id` |
| `list_vm_applications_distinct` | Distinct products running on VMs | — |

### Risk

| Tool | Answers | Key filters |
|---|---|---|
| `get_eol_summary` | What's reaching/past end-of-life | — |
| `get_impact_graph` | What depends on / is affected by an asset | `entity_type`, `id`, `depth` (default 2, max 5) |

### Images

| Tool | Answers | Key filters |
|---|---|---|
| `search_images` | Find a container image by name | `query` (required) |
| `list_image_versions` | Tracked image repos & latest tags | `registry`, `image_repo`, `variant`, `has_error` |
| `get_image_version` | One image repo's version detail | `image_repo` |
| `get_image_versions_summary` | Roll-up of image freshness | — |

### Applications

| Tool | Answers | Key filters |
|---|---|---|
| `list_applications` / `get_application` | Applications + DICT / one Application | `name`, `application_block_name`, `criticality`, `has_dict`, `dict_min`; `id` OR `name` |
| `list_application_blocks` | Top-level blocks | `name`, `owner` |

## Operator recipes

- **"What's reaching end-of-life?"** → `get_eol_summary` for the roll-up; drill
  into a specific product with `list_virtual_machines`/`list_workloads` filtered
  by `image`/`application`.
- **"What breaks if this node/VM/workload goes away?"** → `get_impact_graph`
  with the asset's `entity_type` + `id` (raise `depth` up to 5 for transitive
  reach).
- **"Show me Application X and its classification."** → `get_application` by
  `name` (kebab-cased server-side) or `id`; read the `sec_*` DICT axes and
  `criticality`.
- **"Which VMs run product Y (version Z)?"** → `list_virtual_machines` with
  `application=Y` (and `application_version=Z`); use
  `list_vm_applications_distinct` first if unsure of the exact product name.
- **"Is image Z behind the latest?"** → `search_images` to locate it, then
  `get_image_version` by `image_repo`; `get_image_versions_summary` for the
  fleet-wide picture.
- **"What's running in namespace N?"** → `list_namespaces` (find the `id`) →
  `list_workloads` / `list_pods` / `list_services` filtered by `namespace_id`.

## Guardrails

- **Read-only.** No mutation tools exist. Do not suggest you created, changed,
  or deleted anything.
- **Pagination.** List endpoints are cursor-paginated — do not assume the first
  page is the whole set; page through when the operator needs a complete count.
- **No fabrication.** Cite asset names and UUIDs from tool output. Never invent
  IDs; obtain them from a `list_*` call first.
- **Writes are out of scope.** If the operator explicitly needs to mutate state,
  point them to the REST API documented at the Swagger UI (`/docs/`) — do not
  attempt it through MCP.

---
> Source: [sthalbert/Longue-Vue](https://github.com/sthalbert/Longue-Vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
