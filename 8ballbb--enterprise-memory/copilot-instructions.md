## enterprise-memory

> **Phase:** documentation complete, pre-build (before M0). This is a docs-only repo today; there

# Enterprise Memory — working instructions

**Phase:** documentation complete, pre-build (before M0). This is a docs-only repo today; there
is no application code yet.

> This is the **docs-phase** root `CLAUDE.md`. The full harness version (root + four nested
> `CLAUDE.md` files, plus the CI membership check) is a **M0 / WP 0.2** deliverable, specified in
> [`build/claude_code_harness.md`](build/claude_code_harness.md) §1. Expand this file there;
> don't create the nested ones until their code directories exist.

## Authoritative source

**[`spec/`](spec/) is the specification. Nothing else is.**
- Start at **[`spec/core/index.md`](spec/core/index.md)**, then `01_problem.md` →
  **`spec/core/01a_concepts.md`** — the vocabulary the rest of the corpus assumes, with a worked
  example. `00_builders_guide.md` is build logistics, not orientation.
- The conformance surface (C-*/VS-*/G-*) lives in `spec/core/02_requirements.md`. It is a checklist
  to consult per milestone, not a document to read through.
- Repo map + statuses: [`INDEX.md`](INDEX.md).

**Layer resolution** (`spec/CONVENTIONS.md`): decision layer (`core/`) wins over
annex (`annexes/`) wins over contract *intent* (`contracts/`). A contract-vs-decision
disagreement is a **bug to fix, not to interpret**. Every threshold/SLO has exactly one normative
home; cite the ID, don't restate the value.

**Do not build from** anything in `archive/` (superseded v1.3.1 spec/contracts + design trail).
`build/roadmap.md` and `build/claude_code_harness.md` are adopted/normative; `decisions/` and
`reference/` are consult-only Reference.

## Invariant "never" list (security-load-bearing — keep verbatim)

- Credentials are **never** stored inline in config — always a `credential_ref` resolved via the
  secrets backend.
- ACL filtering runs in the retrieval layer **before** any retrieved content reaches an LLM
  prompt or response (C-21a).
- Virtual-source content is **never** admitted to derivation: it never creates or corroborates
  entities, edges, or facts, and is quarantined from Branch-B extraction (VS-1).
- EM **never** substitutes an elevated service credential for a lesser-privileged caller when
  delegating to a virtual source (VS-2).
- `GraphAdapter.execute()` is **never** reachable from any API endpoint.
- Backend/provider SDK imports **never** leave their home directory (`storage/adapters/…`,
  `llm/providers/…`).
- Every outbound fetch enforces SSRF defense: HTTPS-only, block RFC-1918/link-local/loopback,
  check **every** resolved IP.
- Content **never** sets its own trust tier.
- Cross-workspace **and** cross-org probes return **404, never 403** — no existence oracle (C-3).

## Working conventions

- Session/authoring discipline and anchors: `spec/CONVENTIONS.md`.
- When changing a cross-cutting behaviour, check its blast-radius row in `spec/CONCERNS.md`.
- Keep `spec/TRACEABILITY.md` and `spec/build/conformance_matrix.md` consistent with
  any invariant you touch.

---
> Source: [8ballbb/enterprise_memory](https://github.com/8ballbb/enterprise_memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
