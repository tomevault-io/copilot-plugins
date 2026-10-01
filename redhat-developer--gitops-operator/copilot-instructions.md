## gitops-operator

> Before adding to this file, carefully consider that poorly written AGENTS.md files can hurt task success rates, while also increasing token usage.

# Repository AI Guidelines

Before adding to this file, carefully consider that poorly written AGENTS.md files can hurt task success rates, while also increasing token usage.

## E2E Testing Guidelines (`test/openshift/e2e/ginkgo/**/*`)

When creating or modifying Ginkgo E2E test files:

### General Guidelines

- **Avoid `k8sClient.Update(...)`**: This call will occasionally fail due to Kubernetes resources being modified between the Get and Update calls.
  - Instead, use fixture-specific `Update` functions. Most fixture packages include an `Update` function, for example for `ArgoCD` CR: use `argocdFixture.Update(...)`. For generic resource update, see `k8sFixture.Update(...)`.

### Parallel Tests (`test/openshift/e2e/ginkgo/parallel/*_test.go`)

- Use `Describe("GitOps Operator Parallel E2E Tests", func() { ... })` as the top-level describe block
- Call `fixture.EnsureParallelCleanSlate()` in the `BeforeEach` function. (not `EnsureSequentialCleanSlate`)

### Sequential Tests (`test/openshift/e2e/ginkgo/sequential/*_test.go`)

- Use `Describe("GitOps Operator Sequential E2E Tests", func() { ... })` as the top-level describe block
- Call `fixture.EnsureSequentialCleanSlate()` in the `BeforeEach` function. (not `EnsureParallelCleanSlate`)

---
> Source: [redhat-developer/gitops-operator](https://github.com/redhat-developer/gitops-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
