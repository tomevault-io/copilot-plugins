## archlens

> > Gilt für jede Gemini/Antigravity-Session in diesem Projekt.

# GEMINI.md — ArchLens Agent-Regeln

> Gilt für jede Gemini/Antigravity-Session in diesem Projekt.

---

## 1. Projekt-Einstieg (Pflicht bei jedem Start)

Bevor du irgendetwas tust, lies in dieser Reihenfolge:

1. `AGENTS.md` — Architektur, Bounded Contexts, Model Lock
2. `context/decisions.md` — Alle bisherigen Entscheidungen (append-only)
3. `context/todo.md` — Offene Aufgaben und Prioritäten
4. `context/brief.md` — Aktueller Projektstatus

**Nie ohne decisions.md arbeiten.** Du wirst sonst Entscheidungen wiederholen.

---

## 2. Die 6 Architektur-Gebote

Diese sind in AGENTS.md definiert. Hier die Kurzform für schnellen Zugriff:

1. **Zero-Code-Egress** — Kundencode verlässt nie die CI-Umgebung
2. **Determinismus vor LLM** — AST → Graph → Heuristik → LLM (letzter Ausweg)
3. **Graph-Diff ist das Produkt** — Nicht der Graph, sondern das Signal
4. **Config-as-Code** — `.archlens.yml` im Kunden-Repo
5. **Bounded Components** — `action/`, `api/`, `worker/`, `dashboard/` sind isoliert
6. **Dogfooding** — Ab Woche 2 scannt ArchLens sich selbst

---

## 3. Coding-Regeln

### Python (api/, worker/, action/)
- Python 3.11+
- Type Hints überall (strict mypy)
- Pydantic v2 für Schemas
- FastAPI mit dependency injection
- Docstrings: Google-Style
- Tests: pytest, Ziel >80% Coverage

### TypeScript (dashboard/)
- Next.js 14+ App Router
- Strict TypeScript
- shadcn/ui Komponenten
- Kein `any` — niemals

### Allgemein
- Sprache in Code/Kommentaren: **Englisch**
- Sprache in Context-Dateien: **Deutsch**
- Commit Messages: Conventional Commits (feat:, fix:, refactor:)
- Keine Magic Numbers — Konstanten in `shared/constants/`

---

## 4. Graphify-Nutzung

Graphify ist eine Blackbox-Dependency. Nicht forken, nicht patchen.

```python
# RICHTIG: Import als Library
from graphify.analyze import build_graph, graph_diff
from graphify.cluster import leiden_cluster

# FALSCH: Shell-Aufruf
subprocess.run(["graphify", "."])  # Nur als Fallback wenn Import scheitert
```

Wenn der Python-Import scheitert → `subprocess.run` als temporärer Fallback, aber sofort Issue erstellen.

---

## 5. Feature-Phasen (NICHT überspringen)

| Phase | Features | Gate |
|-------|----------|------|
| 0 (Spike) | graph_diff Signal-to-Noise validieren | Go/No-Go Entscheidung |
| 1 (MVP) | `.archlens.yml`, PR-Kommentar, Mermaid, Blast Radius, Dashboard | Erster zahlender Kunde |
| 2 (Memory) | Boiling Frog, ADR-Match, Accept-Debt, Reviewer-Tagging | 10+ aktive Kunden |
| 3 (Enterprise) | CTO Dashboard, AI Hints, Multi-SCM, SSO | Enterprise-Pipeline |

**REGEL:** Kein Phase-2-Feature darf in Phase 1 implementiert werden. Kein "ich bau das schnell mit rein". Scope-Disziplin ist das Produkt.

---

## 6. Anti-Drift-Regeln (Meta: Wir essen unser eigenes Dogfood)

- [ ] Vor jedem PR: Prüfe ob der Change einen neuen Cross-Component-Import einführt
- [ ] Vor jedem Feature: Prüfe ob es in die aktuelle Phase gehört
- [ ] Bei Unsicherheit: In `context/decisions.md` loggen und Mensch fragen
- [ ] Keine "temporären" Architektursünden — es gibt kein temporär in Software

---

## 7. Secrets & Config

- API Keys: Umgebungsvariablen, nie in Code
- Lokale Entwicklung: `.env` (in .gitignore)
- Production: GitHub Secrets + Cloud Provider Secrets Manager
- Graphify braucht: Claude API Key (nur für Nicht-Code-Extraktion, Phase 2+)

---

## 8. Wichtige Entscheidungs-Referenzen

Alle architektonischen Entscheidungen sind in `context/decisions.md` geloggt.
Wenn du eine Frage hast die dort beantwortet ist → nicht nochmal fragen.
Wenn du eine neue Entscheidung triffst → dort loggen.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jkunisch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
