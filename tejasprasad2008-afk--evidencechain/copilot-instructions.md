## evidencechain

> EvidenceChain is a custom MCP server that exposes 21 forensic analysis tools

# EvidenceChain Agent Configuration

## Overview

EvidenceChain is a custom MCP server that exposes 21 forensic analysis tools
for autonomous incident response on the SANS SIFT Workstation. The agent
connects via stdio transport and uses the tools to investigate disk images,
memory captures, and enrichment data.

## MCP Server

```json
{
  "mcpServers": {
    "evidencechain": {
      "command": "python3",
      "args": ["-m", "evidencechain"],
      "cwd": "/path/to/evidencechain/src",
      "env": {
        "EVIDENCE_BASE_DIR": "/cases",
        "VT_API_KEY": "",
        "ABUSEIPDB_API_KEY": "",
        "OTX_API_KEY": ""
      }
    }
  }
}
```

## Investigation Workflow

When investigating a case, follow this sequence:

### Phase 1: Evidence Mounting
1. Call `mount_evidence` with the disk image path
2. Note the returned mount point

### Phase 2: Disk Analysis
3. Call `get_filesystem_timeline` to build a MAC timeline
4. Call `parse_mft` to check for timestomping ($SI vs $FN comparison)
5. Call `parse_prefetch` to identify executed programs (PROVES execution)
6. Call `parse_amcache` to get SHA1 hashes of executed programs
7. Call `parse_event_logs` with key Event IDs (4624, 4688, 7045, 1116)
8. Call `parse_registry` on SYSTEM, SOFTWARE, NTUSER.DAT hives
9. Call `extract_file` for any suspicious binaries found

### Phase 3: Memory Analysis
10. Call `memory_process_list` — compare pslist vs psscan for hidden processes
11. Call `memory_network_connections` — identify C2 channels
12. Call `memory_injected_code` — find RWX regions (caveat: .NET false positives)
13. Call `memory_services` — cross-reference with registry persistence
14. Call `memory_command_lines` — extract attacker commands
15. Call `memory_dump_process` for suspicious PIDs

### Phase 4: Enrichment
16. Call `compute_hashes` on extracted files
17. Call `enrich_indicators` with hashes, IPs, domains from earlier phases
18. Call `yara_scan` on extracted files with SIFT's YARA rules

### Phase 5: Self-Correction
19. Call `run_self_correction` to trigger the 4-pass correction engine
20. If the response includes a reinvestigation plan, execute the listed tools
21. Call `run_self_correction` again until convergence

### Phase 6: Reporting
22. Call `generate_report` to produce Markdown + JSON reports
23. Call `unmount_evidence` to clean up

## Critical Forensic Rules

These rules are enforced architecturally, not just by prompt:

- **ShimCache proves PRESENCE, not EXECUTION** — Never claim a program ran
  based solely on ShimCache. Corroborate with Prefetch or Amcache.
- **Amcache proves EXECUTION + provides SHA1** — Use the hash for TI lookups.
- **Prefetch proves EXECUTION** with up to 8 timestamps per binary.
- **Registry proves PERSISTENCE was configured** — Not that it executed.
- **MFT $SI vs $FN mismatch indicates timestomping** — Flag as anti-forensics.
- **psscan-only processes are potentially hidden** — Investigate further.
- **malfind RWX regions in .NET/JIT processes are expected** — Not malicious.

## Evidence Semantics

Every tool call produces EvidenceAtoms with explicit `proves`, `suggests`,
and `cannot_prove` sets. The self-correction engine uses these semantics to:

1. Detect overclaims (claiming execution from presence-only evidence)
2. Find contradictions across sources (disk vs memory conflicts)
3. Score confidence with evidence-weighted calculations
4. Plan reinvestigation when contradictions are unresolved

## Security Guardrails

These are **architectural** (enforced in code), not prompt-based:

- All evidence is mounted read-only (`ro,loop,noatime`)
- Path validation: reads restricted to `/cases`, `/mnt`, analysis dirs
- Command denylist: `rm`, `dd`, `wget`, `curl`, `ssh`, `python`, `bash`, etc.
- Output capped at 100KB to prevent context window overflow
- Write operations restricted to `analysis/`, `reports/`, `audit/` directories
- No shell execution — all tools are typed functions via MCP

## Lint and Typecheck

```bash
cd /path/to/evidencechain
python3 -m py_compile src/evidencechain/server.py
python3 tests/test_block2_smoke.py
python3 tests/test_block3_smoke.py
python3 tests/test_block4_smoke.py
python3 tests/test_block5_smoke.py
python3 tests/test_block6_smoke.py
python3 tests/test_block7_integration.py
```

---
> Source: [tejasprasad2008-afk/evidencechain](https://github.com/tejasprasad2008-afk/evidencechain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
