## coding-tools-mcp

> This subtree owns the Cloudflare Worker control plane for starting coding-tools sandboxes.

# Sandbox control guide

This subtree owns the Cloudflare Worker control plane for starting coding-tools sandboxes.

## Contract boundary

The Worker dispatch payload and `.github/workflows/start-sandbox.yml` workflow inputs form a shared contract. Do not change one side without checking the other.

Run `python3 scripts/check_dispatch_inputs.py` after changing Worker dispatch inputs or the sandbox workflow.

Keep this component in the same repository as the workflow unless the control-plane API is explicitly versioned and cross-repository compatibility is introduced.

---
> Source: [xyTom/coding-tools-mcp](https://github.com/xyTom/coding-tools-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
