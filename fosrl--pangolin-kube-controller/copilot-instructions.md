## pangolin-kube-controller

> This file provides repository-wide instructions for coding agents working in this repository.

# AGENTS.md

## Purpose

This file provides repository-wide instructions for coding agents working in this repository.

The project is a Go-based Kubernetes controller that fetches Pangolin Traefik dynamic configuration from an external endpoint, transforms it into Kubernetes resources, and reconciles those resources into the cluster.

Use this file for global rules. More specific instructions in deeper directories should take precedence for files in those paths.

## Specialized agents and skills

Use the specialized agent profiles in `.github/agents/` when the task clearly matches a role.

- `controller-agent`: controller logic, reconciliation, Pangolin-to-Traefik transformation, Kubernetes runtime behavior, manifests
- `ci-agent`: GitHub Actions, Task-based CI, workflow failures, validation flow
- `docs-agent`: `README.md`, `docs/`, contributor and policy documentation
- `lint-agent`: mechanical lint and formatting fixes only
- `security-agent`: evidence-based security review and minimal remediation
- `test-agent`: unit, integration, E2E, fixtures, and test failure diagnosis

Use repository skills in `.agent/skills/` for repeatable workflows such as controller change review, CI triage, integration-test setup, manifest review, and documentation verification.

This root file defines repository-wide rules. Specialized agents add role-specific guidance. Skills provide reusable workflow instructions, examples, and resources that should be loaded only when relevant.

### Skill usage rules (important)

- Prefer skills over ad-hoc reasoning for repeatable workflows.
- If a matching skill exists in `.agent/skills/`, it MUST be used.
- Do not reimplement workflows already defined in skills.
- Load only relevant skills to avoid unnecessary context usage.

---

## Quick start

Run the smallest relevant set of checks first, then expand only if needed.

### Environment setup

- `go mod download`

### Common validation paths

- Build: `go build ./...`
- Fast tests: `task test`
- Full verification: `task test:crosspkg`
- Integration tests: `task test:integration`
- Coverage merge/report: `task coverage`
- Format: `task fmt`
- Lint: `task lint`
- CI reproduction: `task ci`

### Notes

- Integration tests require `setup-envtest`.
- Prefer targeted validation for small changes, and broader validation for cross-cutting changes.
- Do not claim a command passed unless you actually ran it.

---

## Repository snapshot

### Tech stack

- Language: Go
- Module: `pangolin-kube-controller`
- Primary domain: Kubernetes controller / operator-style reconciliation
- Main concerns: config fetch, transform, apply, garbage collection, observability, release automation
- Deployment model: Kubernetes-focused controller runtime; this repository is intended to run as a Kubernetes workload rather than a general standalone service.
- External config source: Pangolin.
- Reconciled output: Traefik-related Kubernetes resources.

### Key directories

- `cmd/controller/`: main controller entrypoint
- `cmd/healthcheck/`: healthcheck entrypoint
- `internal/controller/`: reconciliation loop, fetch, apply orchestration, backoff, leader election, readiness
- `internal/apply/`: server-side apply helpers and resource application logic
- `internal/config/`: environment/file config loading, defaults, normalization
- `internal/httpserver/`: `/healthz`, `/readyz`, `/metrics`, optional TLS
- `internal/kube/`: Kubernetes client and label/resource helpers
- `internal/observability/`: logging, Prometheus metrics, OpenTelemetry metrics
- `internal/reconcile/`: garbage collection
- `internal/transform/`: routing, protocol conversion, sanitization, config transformation
- `test/integration/`: integration tests
- `test/e2e/`: offline E2E tests
- `docs/`: project and process documentation
- `hack/`: scripts, task includes, and helper tools
- `.github/workflows/`: CI/CD pipelines
- `.github/agents/`: GitHub custom agent role files

### Runtime model

The controller typically:

1. loads configuration from env and/or file
2. polls the Pangolin config endpoint
3. detects whether the config changed
4. transforms config into Kubernetes-native objects
5. applies those resources using server-side apply
6. garbage-collects no-longer-managed resources
7. exposes health and metrics endpoints
8. optionally participates in leader election

---

## Golden rules

These apply to every agent, regardless of task.

- Controller behavior must remain idempotent and safe under retries.
- Verify before claiming. Never invent commands, outputs, files, behavior, or CI results.
- Keep diffs small. Avoid unrelated formatting, renames, or refactors.
- Preserve behavior unless the task explicitly asks for behavior changes.
- Preserve security posture. Do not weaken auth, permissions, validation, TLS handling, secret handling, or logging hygiene.
- Preserve least privilege in CI and deployment automation.
- Update tests when behavior changes.
- Update docs when user-facing behavior, commands, configuration, or workflows change.
- Prefer targeted checks first, then broader checks as needed.
- Never commit secrets, tokens, kubeconfigs, certificates, or private credentials.
- Never state that something is production-safe unless that conclusion is directly supported.
- Never introduce destructive reconciliation logic without explicit instruction.

---

## Working style

### Change philosophy

- Prefer minimal, reviewable patches.
- Follow existing patterns in the touched package.
- Do not introduce a new abstraction unless it materially reduces complexity.
- Do not move files or restructure directories unless the task explicitly requires it.

### Validation philosophy

Run validation proportional to the change:

- Markdown-only change: markdown lint for touched docs
- Go logic change: targeted `go test` plus broader package or repo tests as needed
- Controller logic change: include relevant tests under `internal/controller`, `internal/apply`, `internal/reconcile`, `internal/transform`
- CI/workflow change: validate YAML and reproduce with the closest available local commands
- Release/security/config changes: validate carefully and keep blast radius minimal

---

## Commands

Only use commands that are relevant to your change.

### Setup

- `go mod download`

### Build

- `go build ./...`
- `task build`

### Test

- Fast test path: `task test`
- Cross-package coverage test path: `task test:crosspkg`
- Integration tests: `task test:integration`
- Coverage/reporting: `task coverage`

### Targeted Go test examples

- `go test ./internal/...`
- `go test ./cmd/...`
- `go test ./test/integration/...`
- `go test -run TestName ./path/to/package`

### Formatting and linting

- `task fmt`
- `task lint`
- `golangci-lint run --timeout=5m`
- `markdownlint-cli2 "**/*.md" "!**/vendor/**" "!**/node_modules/**"`
- `yamllint -c .yamllint.yaml .`
- `hadolint Dockerfile`
- `hadolint Dockerfile.scratch`
- `shfmt -d -s $(git ls-files '*.sh' || true)`

### CI reproduction and diagnostics

- `task ci`
- `go vet ./...`

### Security-focused checks

Use only when relevant and available in the environment:

- `gosec -exclude-dir=internal/testschema ./...`

If the local environment does not already have a tool installed, prefer repo-native task commands first. Do not pin or install extra tooling unless the task actually requires it.

### Release and versioning

Use release commands only when explicitly asked to work on release flow or release artifacts.

- `task release VERSION=X.Y.Z`

Do not create or simulate releases unless the task requires it.

---

## Local execution guidance

Use repo-native build paths where possible.

### Typical build output

The documented build flow produces a controller binary such as:

- `bin/pangolin-kube-controller`

### Example local run pattern

After building, run the produced controller binary with the environment/config required by the controller.

Do not assume a stable CLI flag surface unless you have verified it in code or docs. Prefer documented project commands over guessed runtime invocations.

---

## Coding standards

### Go

- Keep files `gofmt`-formatted.
- Follow existing package structure and naming.
- Wrap errors with context using `%w` where appropriate.
- Keep functions focused and avoid unnecessary refactors.
- Preserve reconciliation semantics and idempotency.

Good:

```go
if err := s.srv.Shutdown(shutdownCtx); err != nil {
  return fmt.Errorf("shutdown metrics server: %w", err)
}
```

Bad:

```go
if err := s.srv.Shutdown(shutdownCtx); err != nil {
  // Bad: loses original error for inspection/unwrapping
  return fmt.Errorf("shutdown metrics server: %v", err)
}
```

### Logging

- Use structured logging patterns already present in the repo.
- Keep messages factual and operationally useful.
- Never log secrets, raw credentials, tokens, or private keys.
- Be especially careful when touching redaction-related code under `internal/observability/logging/`.

### YAML and Markdown

- Respect `.yamllint.yaml` and `.markdownlint-cli2.yaml`.
- Keep docs accurate, specific, and command-verifiable.
- Do not add aspirational statements as if they were already true.

---

## High-risk areas

Be extra careful in these paths:

- `internal/controller/`
- `internal/apply/`
- `internal/reconcile/`
- `internal/config/`
- `internal/httpserver/`
- `internal/observability/`
- `.github/workflows/`
- `hack/scripts/`
- `Taskfile.yml`
- `SECURITY.md`

Changes here can affect runtime behavior, cluster safety, release flow, or security posture.

---

## Git and PR workflow

### Branching

- Use feature or fix branches from `dev`.
- Naming examples: `feat/...`, `fix/...`, `docs/...`, `chore/...`

### Commits

- Follow Conventional Commits where possible:

  - `feat: ...`
  - `fix: ...`
  - `docs: ...`
  - `chore: ...`
  - `refactor: ...`
  - `test: ...`

### Pull requests

- Target `dev` unless explicitly instructed otherwise.
- Keep PR scope focused.
- Ensure relevant lint/tests for the change have been run.
- Summarize what changed, why it changed, and how it was validated.

### Definition of done

Before considering work complete:

- relevant checks were run
- changed behavior is covered by tests where appropriate
- docs were updated if needed
- no secrets were introduced
- no security posture was weakened
- the diff is limited to the requested task

---

## Boundaries

### Always

- Keep changes scoped to the requested task.
- Prefer the smallest safe patch.
- Add or update tests when behavior changes.
- Update docs for changed commands, config, or workflows.
- Preserve existing security guarantees.

### Ask first

- Changes to GitHub workflow permissions
- Changes involving secrets handling
- Changes to release flow, tagging, publishing, or image push logic
- Changes to `SECURITY.md`
- Changes that intentionally alter controller behavior or public-facing semantics
- Large file moves or repository restructuring

### Never

- Add credentials or secrets to the repository
- Disable or bypass security checks just to make CI pass
- Remove tests only to make failures disappear
- Broaden token permissions without clear need
- Claim a command or workflow was verified when it was not
- Modify generated, vendored, or external dependency content without explicit reason

---

## Repo-specific guidance

This repository should be treated as a Kubernetes controller project first, not as a generic Go service.

Primary runtime assumptions:

- Kubernetes-first deployment model
- Pangolin as external configuration source
- Traefik-related Kubernetes resources as reconciled output

Preserve these invariants unless the task explicitly changes them:

- reconciliation idempotency
- safe retries and bounded backoff
- leader election semantics
- readiness and health semantics
- safe apply and garbage-collection behavior
- Traefik resource compatibility

### Controller changes

When touching reconciliation logic, fetch logic, apply logic, routing, or garbage collection:

- preserve idempotency
- preserve safe retry behavior
- preserve backoff behavior unless explicitly changing it
- preserve leader election semantics unless explicitly changing them
- prefer additive tests over broad rewrites
- verify impact on managed Traefik resource types

### Config changes

When touching `internal/config/`:

- preserve backward compatibility unless instructed otherwise
- verify defaults, env parsing, and normalization together
- document any new config knobs or changed semantics

### Metrics and observability

When touching metrics, health, readiness, or logging:

- preserve metric meaning whenever possible
- avoid silent renames of metrics or endpoints
- avoid introducing noisy or secret-bearing logs
- validate `/healthz`, `/readyz`, and `/metrics` logic if affected

### CI and release changes

When touching `.github/workflows/`, `.github/actions/`, `hack/`, or release tasks:

- preserve least privilege
- preserve action pinning and supply-chain hygiene
- avoid unnecessary matrix expansion or runtime cost increases
- keep release logic deterministic and auditable

---

## Agent roster

The repository currently includes specialized GitHub agent role files under `.github/agents/`. The guidance below aligns with those roles.

### controller-agent

Mission:

- work on core controller and runtime behavior for this Go-based Kubernetes controller
- preserve idempotency, safe retries, leader election, readiness, health, and reconciliation safety
- keep Pangolin-to-Traefik transformation behavior correct and test-backed

Preferred write scope:

- `cmd/controller/`
- `cmd/healthcheck/`
- `internal/controller/`
- `internal/apply/`
- `internal/reconcile/`
- `internal/config/`
- `internal/httpserver/`
- `internal/kube/`
- `internal/transform/`
- `internal/observability/`

Validation:

- `go build ./...`
- `task test`
- targeted `go test` commands for touched controller packages
- `task test:crosspkg`
- `task test:integration`
- `go vet ./...`
- `task lint`

Output expectation:

- summary
- behavior impact
- commands run
- files changed
- risks or follow-up

Special boundaries:

- preserve runtime safety properties and reconciliation invariants
- ask first before changing leader election, readiness, health, metrics semantics, GC scope, or deployment topology
- never introduce destructive or non-idempotent controller behavior without explicit justification

### docs-agent

Mission:

- improve or correct documentation based only on verified repository behavior

Preferred write scope:

- `README.md`
- `docs/`
- contributor-facing documentation

Validation:

- `markdownlint-cli2 "**/*.md" "!**/vendor/**" "!**/node_modules/**"`

Output expectation:

- summary
- files changed
- commands run
- any remaining documentation gaps

Special boundaries:

- do not add commands or behavior claims that were not verified
- ask first before editing `SECURITY.md` or release-policy material

---

### test-agent

Mission:

- add or repair tests to increase confidence without unnecessary production-code changes

Preferred write scope:

- `*_test.go`
- `test/integration/**`
- `test/e2e/**`
- test helpers and fixtures where justified

Validation:

- `task test`
- `task test:crosspkg`
- `task test:integration`
- targeted `go test` commands as needed

Output expectation:

- what failed or was missing
- what tests changed
- commands run
- remaining risk or follow-up

Special boundaries:

- keep tests deterministic
- do not delete tests just to make CI green
- ask first before changing production behavior solely to satisfy a test

---

### security-agent

Mission:

- identify security risks and propose the smallest safe fix

Preferred approach:

- report clearly
- patch minimally
- avoid new dependencies unless necessary and approved

Validation:

- relevant repo-native lint/test commands
- security tooling only when relevant and available

Output expectation:

- finding
- impact
- evidence
- minimal remediation

Special boundaries:

- never weaken auth, validation, TLS, redaction, or permissions
- ask first before changing CI security posture or scanner configuration
- never add leaky telemetry or secret-bearing logs

---

### ci-agent

Mission:

- diagnose CI failures and propose minimal, least-privilege fixes

Preferred write scope:

- `.github/workflows/`
- `.github/actions/`
- related tooling config only when required

Validation:

- `task ci`
- `go vet ./...`
- relevant lint commands for touched files

Output expectation:

- root cause
- minimal patch
- validation steps
- any permission or security implications

Special boundaries:

- preserve least privilege
- preserve or improve action pinning
- ask first before changing permissions, secrets usage, deployment behavior, or release publication logic
- never disable checks without explicit instruction

---

### lint-agent

Mission:

- resolve lint and formatting issues with minimal behavioral impact

Preferred write scope:

- source files and docs required to satisfy linting
- linter config only when the existing rule is clearly incorrect or the task explicitly requires it

Validation:

- `task fmt`
- `task lint`
- targeted underlying linter commands where useful

Output expectation:

- lint issue category
- files changed
- commands run
- whether any rule/config discussion remains

Special boundaries:

- prefer fixing code/docs over weakening rules
- ask first before changing lint configuration
- do not bundle unrelated cleanup into lint-only changes

---

## Decision model (agent vs skill)

Use the following decision logic:

1. If the task is a repeatable workflow → use a skill
2. If the task is role-specific reasoning → use a specialized agent
3. If both apply → use the agent and load the relevant skill

Examples:

- Fix CI → use `ci-agent` + `ci-triage` skill
- Change controller logic → use `controller-agent` + `controller-change-review`
- Update docs → use `docs-agent` + `docs-verification`

---

## Skills system (OpenSkills compatible)

This repository supports portable agent skills via the OpenSkills standard.

If skills are installed via OpenSkills, they are exposed in this file as an `<available_skills>` block.

Usage:

- List skills: `npx openskills list`
- Load skill: `npx openskills read <skill-name>`
- Sync skills into AGENTS.md: `npx openskills sync`

Rules:

- Prefer OpenSkills-compatible skills when available.
- Do not duplicate skill logic inside agents or inline reasoning.
- Skills should remain reusable and tool-agnostic.

Recommended installation:

```bash
npx openskills install anthropics/skills --universal
npx openskills sync
```

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:

- Invoke: `npx openskills read <skill-name>` (run in your shell)
  - For multiple: `npx openskills read skill-one,skill-two`
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:

- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>algorithmic-art</name>
<description>Creating algorithmic art using p5.js with seeded randomness and interactive parameter exploration. Use this when users request creating art using code, generative art, algorithmic art, flow fields, or particle systems. Create original algorithmic art rather than copying existing artists' work to avoid copyright violations.</description>
<location>project</location>
</skill>

<skill>
<name>brand-guidelines</name>
<description>Applies Anthropic's official brand colors and typography to any sort of artifact that may benefit from having Anthropic's look-and-feel. Use it when brand colors or style guidelines, visual formatting, or company design standards apply.</description>
<location>project</location>
</skill>

<skill>
<name>canvas-design</name>
<description>Create beautiful visual art in .png and .pdf documents using design philosophy. You should use this skill when the user asks to create a poster, piece of art, design, or other static piece. Create original visual designs, never copying existing artists' work to avoid copyright violations.</description>
<location>project</location>
</skill>

<skill>
<name>claude-api</name>
<description>"Build apps with the Claude API or Anthropic SDK. TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`/`claude_agent_sdk`, or user asks to use Claude API, Anthropic SDKs, or Agent SDK. DO NOT TRIGGER when: code imports `openai`/other AI SDK, general programming, or ML/data-science tasks."</description>
<location>project</location>
</skill>

<skill>
<name>doc-coauthoring</name>
<description>Guide users through a structured workflow for co-authoring documentation. Use when user wants to write documentation, proposals, technical specs, decision docs, or similar structured content. This workflow helps users efficiently transfer context, refine content through iteration, and verify the doc works for readers. Trigger when user mentions writing docs, creating proposals, drafting specs, or similar documentation tasks.</description>
<location>project</location>
</skill>

<skill>
<name>docx</name>
<description>"Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx files). Triggers include: any mention of 'Word doc', 'word document', '.docx', or requests to produce professional documents with formatting like tables of contents, headings, page numbers, or letterheads. Also use when extracting or reorganizing content from .docx files, inserting or replacing images in documents, performing find-and-replace in Word files, working with tracked changes or comments, or converting content into a polished Word document. If the user asks for a 'report', 'memo', 'letter', 'template', or similar deliverable as a Word or .docx file, use this skill. Do NOT use for PDFs, spreadsheets, Google Docs, or general coding tasks unrelated to document generation."</description>
<location>project</location>
</skill>

<skill>
<name>frontend-design</name>
<description>Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.</description>
<location>project</location>
</skill>

<skill>
<name>internal-comms</name>
<description>A set of resources to help me write all kinds of internal communications, using the formats that my company likes to use. Claude should use this skill whenever asked to write some sort of internal communications (status reports, leadership updates, 3P updates, company newsletters, FAQs, incident reports, project updates, etc.).</description>
<location>project</location>
</skill>

<skill>
<name>mcp-builder</name>
<description>Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).</description>
<location>project</location>
</skill>

<skill>
<name>pdf</name>
<description>Use this skill whenever the user wants to do anything with PDF files. This includes reading or extracting text/tables from PDFs, combining or merging multiple PDFs into one, splitting PDFs apart, rotating pages, adding watermarks, creating new PDFs, filling PDF forms, encrypting/decrypting PDFs, extracting images, and OCR on scanned PDFs to make them searchable. If the user mentions a .pdf file or asks to produce one, use this skill.</description>
<location>project</location>
</skill>

<skill>
<name>pptx</name>
<description>"Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions \"deck,\" \"slides,\" \"presentation,\" or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill."</description>
<location>project</location>
</skill>

<skill>
<name>skill-creator</name>
<description>Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.</description>
<location>project</location>
</skill>

<skill>
<name>slack-gif-creator</name>
<description>Knowledge and utilities for creating animated GIFs optimized for Slack. Provides constraints, validation tools, and animation concepts. Use when users request animated GIFs for Slack like "make me a GIF of X doing Y for Slack."</description>
<location>project</location>
</skill>

<skill>
<name>template</name>
<description>Replace with description of the skill and when Claude should use it.</description>
<location>project</location>
</skill>

<skill>
<name>theme-factory</name>
<description>Toolkit for styling artifacts with a theme. These artifacts can be slides, docs, reportings, HTML landing pages, etc. There are 10 pre-set themes with colors/fonts that you can apply to any artifact that has been creating, or can generate a new theme on-the-fly.</description>
<location>project</location>
</skill>

<skill>
<name>web-artifacts-builder</name>
<description>Suite of tools for creating elaborate, multi-component claude.ai HTML artifacts using modern frontend web technologies (React, Tailwind CSS, shadcn/ui). Use for complex artifacts requiring state management, routing, or shadcn/ui components - not for simple single-file HTML/JSX artifacts.</description>
<location>project</location>
</skill>

<skill>
<name>webapp-testing</name>
<description>Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs.</description>
<location>project</location>
</skill>

<skill>
<name>xlsx</name>
<description>"Use this skill any time a spreadsheet file is the primary input or output. This means any task where the user wants to: open, read, edit, or fix an existing .xlsx, .xlsm, .csv, or .tsv file (e.g., adding columns, computing formulas, formatting, charting, cleaning messy data); create a new spreadsheet from scratch or from other data sources; or convert between tabular file formats. Trigger especially when the user references a spreadsheet file by name or path — even casually (like \"the xlsx in my downloads\") — and wants something done to it or produced from it. Also trigger for cleaning or restructuring messy tabular data files (malformed rows, misplaced headers, junk data) into proper spreadsheets. The deliverable must be a spreadsheet file. Do NOT trigger when the primary deliverable is a Word document, HTML report, standalone Python script, database pipeline, or Google Sheets API integration, even if tabular data is involved."</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>

---
> Source: [fosrl/pangolin-kube-controller](https://github.com/fosrl/pangolin-kube-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
