## project-palindrome

> Tools and actions contract for read/write separation, schemas, registration, and safe command execution


# Tools & Actions Contract

This rule defines the contract for tools and actions in this repo: clear read/write separation, consistent schemas and registration, idempotent actions with explicit change summaries, and safe SSH/command execution.

## Read vs write separation

- **Read tools**:
  - Must be strictly read-only (no persistence, no infra mutation).
  - Names should clearly indicate read-only behavior: `get*`, `list*`, `describe*`, `inspect*`, `fetch*`.
  - May perform in-memory transforms only.
- **Write tools / actions**:
  - Must clearly indicate mutation: `create*`, `update*`, `set*`, `delete*`, `patch*`, `apply*`.
  - Must document what they change and the blast radius.

```ts
// ❌ BAD: ambiguous, mixes read and write
export const tool = { name: 'manageVm', /* ... */ };

// ✅ GOOD: explicit separation
export const getVmStatusTool = { name: 'getVmStatus', /* read-only */ };
export const updateVmConfigTool = { name: 'updateVmConfig', /* write */ };
```

## New tool requirements

Every new tool (read or write) must satisfy **all** of the following:

- **Schema present**:
  - Validate inputs using JSON Schema, zod, or the project’s existing schema system.
  - All external inputs must be validated before use.
- **Deterministic output shape + typed result**:
  - Define a typed result (e.g. TypeScript type/interface or zod schema).
  - Avoid ad-hoc shape changes based on branches; if variants are needed, use tagged unions.
- **Registered in the tool registry**:
  - Ensure `tools.json` (and any `tool_definition_*.json` registries) include the tool.
  - Registry entries should reference the schema and output type where applicable.

```ts
// ✅ GOOD: schema + typed result
const InputSchema = z.object({
  vmId: z.string().min(1),
});

type GetVmStatusResult = {
  vmId: string;
  status: 'running' | 'stopped' | 'error';
};
```

## Actions (write paths)

For `src/actions/**` and any write-capable tools:

- **Idempotent where possible**:
  - Repeated calls with the same inputs should converge to the same state.
  - If true idempotency is impossible, clearly document side-effects and preconditions.
- **Twin/tool validation before write**:
  - Read current state from the twin/graph/underlying store before mutating.
  - Validate that the intended change is safe and consistent with the twin.
  - Fail fast if validation fails; do not partially apply changes.
- **“What changed” result**:
  - Return a structured summary of changes (diff/summary), not just `"ok"`.
  - Include identifiers of affected resources and before/after values where safe.

```ts
// ✅ GOOD: action result includes change summary
type UpdateVmConfigChange = {
  field: string;
  before: unknown;
  after: unknown;
};

type UpdateVmConfigResult = {
  vmId: string;
  changes: UpdateVmConfigChange[];
  notes?: string;
};
```

## SSH / command execution

- **Only via approved allowlists**:
  - All SSH and command execution must go through explicit command allowlists.
  - No generic “run any shell command” tools or payloads.
- **Least privilege**:
  - Use the minimum required user/role, directory, and flags.
  - Avoid `sudo`/root where possible; if required, document why.
- **No free-form shell suggestions**:
  - Tools must not accept arbitrary shell strings from prompts or user text.
  - Accept only structured, validated parameters mapped to allowed commands.

```ts
// ❌ BAD: free-form shell
execute({ command }: { command: string }) {
  return ssh.run(command);
}

// ✅ GOOD: structured, allowlisted
executeDiskCheck({ target }: { target: 'root' | 'data' }) {
  const cmd = target === 'root' ? 'df -h /' : 'df -h /data';
  return ssh.run(cmd); // via approved allowlist path
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Phasm22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
