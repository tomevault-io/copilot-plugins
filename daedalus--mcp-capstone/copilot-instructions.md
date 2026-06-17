## mcp-capstone

> MCP server that exposes Capstone disassembly framework functionality for binary disassembly, reverse engineering, and instruction analysis.

# MCP Capstone

MCP server that exposes Capstone disassembly framework functionality for binary disassembly, reverse engineering, and instruction analysis.

## When to use this skill

Use this skill when you need to:
- Disassemble binary code for reverse engineering
- Analyze machine code instructions
- Study assembly code structure
- Perform binary exploitation tasks
- Analyze malware

## Tools

- `get_version` - Get Capstone version
- `check_support` - Check architecture support
- `list_architectures` - List all architectures
- `disasm` - Full disassembly with details
- `disasm_lite` - Lightweight disassembly
- `disasm_quick` - Quick disassembly
- `get_architectures` - All architecture constants
- `get_modes` - All mode constants
- `get_options` - All option constants
- `get_operands` - All operand type constants
- `get_groups` - All instruction group constants
- `get_errors` - All error constants

## Supported Architectures

x86 (16/32/64-bit), ARM, AArch64, MIPS, PowerPC, RISC-V, SPARC, SystemZ, M68K, M680X, TriCore, TMS320C64X, WebAssembly, XCore, BPF, EVM

## Install

```bash
pip install mcp-capstone
```

---
> Source: [daedalus/mcp-capstone](https://github.com/daedalus/mcp-capstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
