## nx-mcp

> NX MCP is a two-process Siemens NX automation server. The external FastMCP

# CLAUDE.md

## Project

NX MCP is a two-process Siemens NX automation server. The external FastMCP
sidecar owns stdio and validation; an NX-journal-owned authenticated loopback
bridge owns NXOpen and the current NX session.

## Commands

```powershell
python -m pip install -e ".[dev]"
python -m pre_commit install --install-hooks
python -m pytest -q -p no:cacheprovider -m "not real_nx" --basetemp .pytest-tmp
python -m nx_mcp.real_smoke --workspace D:\NX_MCP_WORKSPACE --iterations 20
```

## Production path

- `server.py`: sidecar entry point; explicitly exposes the certified surface.
- `certified.py`: typed FastMCP tools and workspace validation.
- `bridge.py`: authenticated loopback protocol, descriptor client, and object IDs.
- `nx_bridge.py`: live NX command executor and manual bridge lifecycle.
- `contracts.py`: Pydantic input/output and error contracts.

Modules under `tools/` are 0.1 legacy source. Default production startup must
not import them; `experimental.py` may load them only behind explicit flags.

## Conventions

- Keep NXOpen imports inside the NX bridge lifecycle; the sidecar must import
  without NX installed.
- Register public tools explicitly. Every input and output has a typed schema.
- File operations must pass `Workspace` validation on both process boundaries.
- Pass opaque object IDs between calls; never pass display-name strings to NX
  builders where an NX object is required.
- Every model mutation gets a visible undo mark and rolls back on failure.
- Builders use create → configure → commit → `Destroy()` in `finally`.
- Preserve the native NX exception code as `nx_code`; use stable domain error
  codes for callers.
- New default tools require local MCP/bridge tests and a real-NX test on the
  supported build.
- `fake_nx` tests verify only the simulated NXOpen seam; the `real_nx` marker
  is the release acceptance gate on the dedicated NX runner.

---
> Source: [DreamEnding/NX_MCP](https://github.com/DreamEnding/NX_MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
