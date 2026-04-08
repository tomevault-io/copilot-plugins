## lnxdrive

> > **This file is automatically loaded by Gemini CLI.**

# LNXDrive Monorepo — Agent Instructions (Gemini)

> **This file is automatically loaded by Gemini CLI.**
> It contains project navigation, DevTrail rules, and tool integration guidelines.

---

## 1. Project Structure (Monorepo)

This is a monorepo containing all LNXDrive components:

| Directory | Description | Tech Stack |
|-----------|-------------|------------|
| `lnxdrive-engine/` | Core daemon + library crates | Rust 1.75+, Cargo workspace (12 crates), tokio, zbus, sqlx |
| `lnxdrive-gnome/` | GNOME Shell/Nautilus/GOA integration | Meson + Rust (gtk4-rs), GJS (ES modules), C11 (Nautilus), Python |
| `lnxdrive-gtk3/` | XFCE/MATE UI | Rust, Cargo, GTK3 |
| `lnxdrive-plasma/` | KDE Plasma integration | C++, CMake, Qt/KDE |
| `lnxdrive-cosmic/` | COSMIC desktop UI | Rust, Cargo |
| `lnxdrive-packaging/` | Distribution packages | Flatpak, AppImage, Debian, AUR |
| `lnxdrive-guide/` | Design & development guide | Markdown (Spanish) |
| `lnxdrive-testing/` | Container/VM test infrastructure | Podman, QEMU/libvirt, shell scripts |

### Key Paths
- **Design guide index**: `lnxdrive-guide/Guía-de-diseño-y-desarrollo.md`
- **DevTrail root**: `.devtrail/` (single instance for entire monorepo)
- **D-Bus name**: `com.strangedaystech.LNXDrive` on `/com/strangedaystech/LNXDrive`

---

## 2. DevTrail — Documentation Guidelines

### Fundamental Principle

> **"No significant change without a documented trace."**

### Your Identity as an Agent

- **Identify yourself** as: `gemini-cli-v1.0` (or your version)
- **Declare** your confidence level: `high | medium | low`
- **Record** identification in the `agent:` field of metadata

### Documentation Reporting

At the end of each task, report DevTrail status:

```
DevTrail: Created AILOG-2026-03-04-001-implement-feature.md
DevTrail: No documentation required (minor change / <10 lines)
DevTrail: Documentation pending - review required
```

### When to Document

| Situation | Action |
|-----------|--------|
| >10 lines of code in business logic | Create AILOG |
| Decision between technical alternatives | Create AIDEC |
| Changes in security/authentication | Create AILOG + mark `risk_level: high` |
| Personal data (GDPR/PII) | Create AILOG + request ETH |
| Integration with external service | Create AILOG |
| Change in public API or DB schema | Create AILOG |

**DO NOT DOCUMENT:** Trivial changes (whitespace, typos, formatting), sensitive information.

### Agent-Logs Organization (Monorepo)

AILOGs are organized by component in `.devtrail/07-ai-audit/agent-logs/`:

| Subdirectory | Component |
|--------------|-----------|
| `daemon/` | Core daemon (lnxdrive-engine/) |
| `gnome/` | GNOME integration (lnxdrive-gnome/) |
| `guide/` | Documentation & design (lnxdrive-guide/) |
| `gtk3/` | GTK3 UI (lnxdrive-gtk3/) — create when needed |
| `plasma/` | KDE Plasma (lnxdrive-plasma/) — create when needed |
| `cosmic/` | COSMIC UI (lnxdrive-cosmic/) — create when needed |
| `packaging/` | Distribution (lnxdrive-packaging/) — create when needed |
| `testing/` | Testing infra (lnxdrive-testing/) — create when needed |

### File Naming

```
[TYPE]-[YYYY-MM-DD]-[NNN]-[description].md
```

### Minimum Metadata

```yaml
---
id: AILOG-2026-03-04-001
title: Brief description
status: accepted
created: 2026-03-04
agent: gemini-cli-v1.0
confidence: high | medium | low
review_required: true | false
risk_level: low | medium | high | critical
---
```

### Autonomy Limits

| Type | I can do | Requires human |
|------|----------|----------------|
| **AILOG** | Create freely | - |
| **AIDEC** | Create freely | - |
| **ETH** | Create draft | Approval |
| **ADR** | Create draft | Review |
| **REQ** | Propose | Validation |
| **INC** | Contribute analysis | Conclusions |
| **TDE** | Identify | Prioritize |

### Quick Type Reference

| Prefix | Name | Location |
|--------|------|----------|
| `AILOG` | AI Action Log | `.devtrail/07-ai-audit/agent-logs/<component>/` |
| `AIDEC` | AI Decision | `.devtrail/07-ai-audit/decisions/` |
| `ETH` | Ethical Review | `.devtrail/07-ai-audit/ethical-reviews/` |
| `ADR` | Architecture Decision Record | `.devtrail/02-design/decisions/` |
| `REQ` | Requirement | `.devtrail/01-requirements/` |
| `TES` | Test Plan | `.devtrail/04-testing/` |
| `INC` | Incident Post-mortem | `.devtrail/05-operations/incidents/` |
| `TDE` | Technical Debt | `.devtrail/06-evolution/technical-debt/` |

---

## 3. Git Operations

> **CRITICAL: Never commit directly to `main` branch.**

All changes must go through feature/fix branches and Pull Requests.

| Branch Prefix | Purpose |
|---------------|---------|
| `feature/` or `feat/` | New features |
| `fix/` | Bug fixes |
| `hotfix/` | Urgent production fixes |
| `docs/` | Documentation only |
| `refactor/` | Code refactoring |
| `test/` | Test changes |

**Conventional Commits:** `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`

---

## 4. LNXDrive Design Guide Reference

> Comprehensive design and development guide at: `lnxdrive-guide/`

| Task | Documents to Load |
|------|-------------------|
| **Understand the project** | `lnxdrive-guide/01-Vision/01-resumen-ejecutivo.md` |
| **Architecture** | `lnxdrive-guide/03-Arquitectura/01-arquitectura-hexagonal.md` |
| **Implement Core/Daemon** | `lnxdrive-guide/04-Componentes/07-motor-sincronizacion.md` |
| **Implement GNOME UI** | `lnxdrive-guide/04-Componentes/02-ui-gnome.md` |
| **Implement KDE UI** | `lnxdrive-guide/04-Componentes/03-ui-kde-plasma.md` |
| **Write tests** | `lnxdrive-guide/06-Testing/01-estrategia-testing.md` |
| **Check roadmap** | `lnxdrive-guide/09-Referencia/02-hoja-de-ruta.md` |

**Main index**: `lnxdrive-guide/Guía-de-diseño-y-desarrollo.md`

---

## 5. Human Review Required

Mark `review_required: true` when:
- `confidence: low`
- `risk_level: high | critical`
- Security decisions
- Irreversible changes

---

*DevTrail v1.0.0 | LNXDrive Monorepo*
*[Strange Days Tech](https://strangedays.tech) — Because every change tells a story.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StrangeDaysTech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/StrangeDaysTech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
