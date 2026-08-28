## stacksmith

> StackSmith is an infrastructure delivery agent plugin.

# AGENTS.md

## Project

StackSmith is an infrastructure delivery agent plugin.

Its purpose is simple:

> Turn infrastructure work items into tested, reviewable OpenTofu pull requests.

The agent may implement and verify infrastructure changes, but production changes remain a human responsibility.

## MVP Scope

The current MVP is intentionally narrow.

Supported stack:

- DeepSeek Harness as the target plugin runtime
- OpenTofu
- AWS
- Floci as the disposable AWS-compatible test environment
- ClickUp as the work-item source
- GitHub as the SCM provider
- TypeScript / Node.js for StackSmith itself

Do not add support for additional clouds, IaC engines, task trackers, or SCM providers unless explicitly requested.

In particular, do not introduce:

- Terraform-specific dependencies
- Pulumi
- Azure
- GCP
- Kubernetes orchestration
- Testcontainers
- databases
- Redis
- message brokers
- backend services
- web UI
- control-plane infrastructure

unless the task explicitly requires them.

## Core Workflow

StackSmith targets this workflow:

```text
ClickUp task
    ↓
inspect existing repository
    ↓
implement OpenTofu change
    ↓
tofu fmt
    ↓
tofu validate
    ↓
sandbox plan/apply against Floci
    ↓
verify resulting infrastructure
    ↓
run OpenTofu plan against target directory
    ↓
review resulting diff
    ↓
branch / commit / push
    ↓
create pull request
    ↓
STOP
```

Production apply is deliberately outside StackSmith's responsibility.

## Safety Boundary

The most important project rule is:

> StackSmith must never apply infrastructure changes to a real target environment.

Allowed against Floci:

- `tofu init`
- `tofu fmt`
- `tofu validate`
- `tofu plan`
- `tofu apply`
- `tofu destroy`
- AWS API/CLI inspection

Allowed against a real target:

- `tofu init`
- `tofu validate`
- `tofu plan`
- read-only inspection required to produce or interpret a plan

Forbidden against a real target:

- `tofu apply`
- `tofu destroy`
- destructive state manipulation
- resource deletion through cloud APIs
- mutating AWS CLI/API commands
- any equivalent operation that modifies real infrastructure

Do not rely only on prompts or comments to enforce this boundary.

Where practical, dangerous capabilities should not exist in the production execution interface at all.

## Architecture

Keep StackSmith divided into two conceptual areas.

```text
StackSmith Core
    |
    +-- ClickUp
    +-- OpenTofu
    +-- Floci
    +-- Verification
    +-- Git
    +-- GitHub
    |
    +-- DeepSeek Harness adapter
```

The core should remain as independent from DeepSeek Harness internals as reasonably possible.

DeepSeek Harness is currently an early/developer-preview dependency. Its APIs may change.

Therefore:

- isolate Harness-specific code under `src/dsh/`
- keep business/domain logic outside `src/dsh/`
- avoid leaking Harness-specific types through the core
- prefer thin adapters over deep framework coupling

A future Harness API change should ideally require changes only in the adapter layer.

## Repository Structure

Prefer this structure:

```text
src/
  core/
  dsh/
  clickup/
  opentofu/
  floci/
  verification/
  git/
  github/

skills/
  stacksmith/

tests/
  unit/
  fixtures/

examples/
docs/
```

Do not create new top-level packages without a clear need.

StackSmith is currently a single plugin.

Do not split it into multiple npm packages or plugins unless explicitly requested.

## Implementation Principles

### Keep the MVP small

Prefer the smallest implementation that proves the workflow.

Avoid speculative abstractions.

Do not create extension points merely because another provider might be added later.

A small interface is appropriate where there is already a real boundary, such as:

- work-item providers
- IaC execution
- SCM interaction

Do not generalize unrelated code prematurely.

### Prefer existing tools

Use existing command-line tools where they already solve the problem well.

Current preferred tools:

- `tofu`
- `git`
- `gh`
- `aws`

Do not replace them with large SDK dependencies without a concrete reason.

For example:

- prefer `gh` over introducing Octokit for the MVP
- prefer AWS CLI inspection over importing many AWS SDK packages
- prefer OpenTofu CLI over building a custom IaC execution engine

### Structured output over stdout parsing

Never parse human-oriented OpenTofu plan output if structured data is available.

Preferred plan flow:

```bash
tofu plan -out=stacksmith.tfplan
tofu show -json stacksmith.tfplan
```

Parse the JSON representation.

At minimum, distinguish:

- create
- update
- delete
- replace

Unexpected deletes or replacements must prevent automatic PR completion unless explicitly handled by the task or future policy configuration.

For the MVP, treat deletes and replacements conservatively.

### Validate external data

External boundaries should be schema validated.

Examples:

- ClickUp API responses
- StackSmith configuration
- OpenTofu JSON output
- verification results

Prefer Zod for runtime validation.

### Process execution

Use the project's process execution abstraction rather than scattering raw `child_process` usage throughout the codebase.

`execa` is the preferred process execution library.

Commands should:

- capture stdout
- capture stderr
- preserve exit codes
- avoid shell interpolation where unnecessary
- avoid logging secrets
- return structured results to callers

## ClickUp

ClickUp is the only supported work-item provider in the MVP.

Normalize task data into StackSmith-owned types before exposing it to the rest of the system.

A work item should contain only what StackSmith needs, for example:

```ts
interface WorkItem {
  id: string;
  title: string;
  description: string;
  acceptanceCriteria: string[];
  comments: string[];
}
```

Do not leak raw ClickUp response types through the application.

Use the ClickUp REST API directly unless an SDK becomes necessary.

## OpenTofu

OpenTofu is the only supported IaC engine in the MVP.

Prefer CLI-driven execution.

The expected local validation sequence is:

```bash
tofu fmt -check
tofu init
tofu validate
```

Sandbox verification may additionally use:

```bash
tofu plan
tofu apply
```

Real target environments must stop at:

```bash
tofu plan
```

StackSmith must not expose a real-target `apply()` capability.

## Floci

Floci is currently treated as an externally available sandbox.

Do not implement container lifecycle management yet.

Assume StackSmith receives or discovers the required Floci endpoint and sandbox credentials through configuration/environment.

The MVP does not require:

- Docker orchestration
- Testcontainers
- sandbox scheduling
- multi-tenant sandbox management

Floci exists to answer:

> Does this infrastructure change actually apply and produce the expected resource state?

## Verification

Do not trust successful `tofu apply` alone.

After sandbox apply, verify actual resource state.

The first implementation may use AWS CLI/API inspection.

Examples:

```text
S3 bucket exists
S3 public access is blocked
S3 encryption is enabled
S3 versioning is enabled

RDS is not publicly accessible

security group rules match expectations
```

Prefer deterministic assertions over LLM-based judgment.

The verifier should return structured results.

Example:

```ts
interface VerificationResult {
  passed: boolean;
  checks: VerificationCheck[];
}
```

An agent claiming success is not proof of success.

Infrastructure state is the proof.

## Git and GitHub

Use normal Git operations for repository changes.

Use GitHub CLI (`gh`) for pull-request creation in the MVP.

The expected end state is a PR containing:

- work-item reference
- summary of changes
- validation evidence
- sandbox verification outcome
- real target plan summary
- explicit statement that production was not modified

Do not commit:

- credentials
- tokens
- `.env`
- state files
- OpenTofu plan files
- generated secrets

## Testing

Use Vitest for StackSmith unit tests.

Tests should prioritize:

- parsers
- command construction
- safety rules
- external-response normalization
- plan summarization
- verification logic

Prefer deterministic tests.

Avoid mocking the entire world when a small fixture will do.

Golden/end-to-end infrastructure scenarios will be added incrementally.

Initial target scenarios include:

1. Create a private encrypted S3 bucket
2. Enable versioning on an existing bucket
3. Add IAM access to a bucket
4. Add a private PostgreSQL database
5. Correct an overly permissive security group

## Dependencies

Keep dependencies minimal.

Current preferred runtime dependencies:

- `zod`
- `execa`

Current development dependencies:

- TypeScript
- Vitest
- ESLint
- Prettier
- tsx
- Node.js types

Before introducing another dependency, ask:

1. Is this solving a current MVP problem?
2. Can an existing CLI or platform capability already do it?
3. Is the maintenance cost justified?

## Code Quality

Before considering a task complete, run the applicable checks:

```bash
npm run typecheck
npm test
npm run build
npm run format:check
```

When infrastructure examples are modified, also run the applicable OpenTofu checks.

Do not mark a task complete while known tests or validations are failing.

## Documentation

Update documentation when a change affects:

- user setup
- environment variables
- plugin installation
- workflow behavior
- safety boundaries
- architecture
- supported capabilities

Avoid documentation for speculative future features.

## Definition of Done

A StackSmith implementation task is complete when:

- requested behavior is implemented
- relevant tests pass
- type checking passes
- build passes
- no production mutation capability has been introduced
- documentation is updated where necessary
- the implementation remains within current MVP scope

For end-to-end infrastructure tasks, completion additionally requires:

- OpenTofu validation succeeds
- Floci apply succeeds
- infrastructure assertions pass
- real target plan succeeds
- dangerous or unexpected changes are surfaced
- the resulting PR contains sufficient evidence for human review

## Final Principle

StackSmith is not an autonomous production operator.

It is an infrastructure development agent.

Its job ends with a verified, reviewable pull request.

---
> Source: [ByteFlow-Bilisim/stacksmith](https://github.com/ByteFlow-Bilisim/stacksmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
