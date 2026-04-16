## srpm-scanner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository mirrors RHEL-compatible kernel sources from Rocky Linux SRPMs into a chronologically tagged Git repository. It's designed for downstream tooling (MCP/LLM servers) that need tagged snapshots for semantic search, symbol lookup, and call graphs.

## Repository Structure

- **`main` branch**: Minimal docs only (README.md, CHANGELOG.md, index.json)
- **`rhel9` / `rhel10` branches**: Full kernel source snapshots, one commit per SRPM version
- **Tags**: `rhel<major>-<version-release>` (e.g., `rhel9-5.14.0-570.52.1.el9_6`)

## Running the Import

### Via Container (Recommended)
```bash
# Requires podman or docker
./rhel-kernel-import-oci.sh /path/to/target/repo
```

This builds Rocky 9 and Rocky 10 container images, then runs the import for both RHEL streams.

### Direct Execution (Inside Container)
```bash
./import-rhel-kernel-srpms.sh /path/to/repo
```

Runs inside the container. Requires rpm-build tools and Rocky Linux environment.

## Key Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MAJORS` | `9 10` | RHEL major versions to import |
| `MODE` | `prep` | `prep` = full source tree; `sources` = raw SRPM contents |
| `ARCH` | `x86_64` | Target architecture for rpmbuild |
| `DEBUG` | `0` | Set to `1` for verbose output |
| `KEEP_SRPMS` | `1` | Keep downloaded SRPMs after import |
| `CONFIG_ARCHES` | `x86_64 ppc64le s390x` | Architectures for Kconfig export |
| `CONFIG_GENERATE` | `1` | Run `make olddefconfig` to resolve configs |

## Architecture

### Workflow
1. **Discovery**: Scrapes Rocky Linux mirrors/vault for kernel SRPMs
2. **Download**: Fetches missing SRPMs to `.work/srpms/`
3. **Prep**: Runs `rpm -Uvh` + `rpmbuild -bp` to expand source
4. **Config Export**: Optionally extracts per-arch Kconfig files to `metadata/configs/`
5. **Commit**: Creates orphan branch commit + annotated tag per version
6. **Index**: Updates `main` branch with CHANGELOG.md and index.json

### Key Functions in import-rhel-kernel-srpms.sh
- `gather_urls()`: Discovers SRPM URLs from Rocky mirrors
- `prep_srpm()`: Expands SRPM via rpmbuild -bp, returns source tree path
- `export_configs()`: Extracts/generates Kconfig files per architecture
- `commit_one()`: Creates commit + tag on the appropriate rhel branch
- `update_main()`: Refreshes CHANGELOG.md and index.json on main branch

### Directory Layout (Runtime)
```
.work/
├── srpms/          # Downloaded SRPMs (cached)
├── tmp/            # rpmbuild workdirs (pruned after 12h)
├── logs/           # Per-SRPM install/prep logs
└── config-export/  # Temporary Kconfig staging
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/randomparity) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
