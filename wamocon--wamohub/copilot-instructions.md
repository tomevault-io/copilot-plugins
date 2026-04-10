## wamohub

> <th width="50%">DE Deutsch</th>

﻿
---

# GitHub Copilot Customisation

<table>
<tr>
<th width="50%">DE Deutsch</th>
<th width="50%">EN English</th>
</tr>
<tr>
<td>Dieses Projekt nutzt GitHub Copilot Agents und Instructions, um eine strukturierte und produktive KI-gestützte Entwicklung zu ermöglichen.</td>
<td>This project uses GitHub Copilot Agents and Instructions to enable a structured and productive AI-assisted development workflow.</td>
</tr>
</table>

---

## Übersicht / Overview

| Typ / Type | Pfad / Path | Beschreibung / Description |
|---|---|---|
| **Global Instructions** | `.github/copilot-instructions.md` | Projektweite Basisregeln für alle Copilot-Interaktionen. / Project-wide baseline rules for all Copilot interactions. |
| **Instructions** | `.github/instructions/*.instructions.md` | Dateimuster-spezifische Coding-Richtlinien. / File-pattern-scoped coding guidelines. |
| **Agents** | `.github/agents/*.agent.md` | Spezialisierte KI-Personas für verschiedene Phasen. / Specialised AI personas for different development phases. |

---

## `copilot-instructions.md` — Globale Basisregeln / Global Baseline Rules

**Pfad / Path:** `.github/copilot-instructions.md`

<table>
<tr>
<th width="50%">DE Was ist das?</th>
<th width="50%">EN What is it?</th>
</tr>
<tr>
<td>Diese Datei wird bei <strong>jeder</strong> Copilot-Interaktion automatisch geladen. Sie definiert den Tech-Stack, kritische Regeln (async APIs, Server Components, Supabase), und Code-Stil-Konventionen für das gesamte Projekt.</td>
<td>This file is automatically loaded on <strong>every</strong> Copilot interaction. It defines the tech stack, critical rules (async APIs, Server Components, Supabase), and code style conventions for the whole project.</td>
</tr>
</table>

<table>
<tr>
<th width="50%">DE Wann bearbeiten?</th>
<th width="50%">EN When to edit?</th>
</tr>
<tr>
<td>
- Tech-Stack ändert sich (z. B. neue Bibliothek wird Standardmuster)<br>
- Neue projektweite Regeln sollen für alle Copilot-Interaktionen gelten<br>
- Team-Konventionen ändern sich<br>
<strong>⚠️ Kurz halten</strong> — diese Datei wird immer in den Kontext geladen.
</td>
<td>
- Tech stack changes (e.g. a new library becomes a standard pattern)<br>
- New project-wide rules need to apply to all Copilot interactions<br>
- Team conventions change<br>
<strong>⚠️ Keep it short</strong> — this file is always loaded into context.
</td>
</tr>
</table>

---

## Instructions — Datei-spezifische Richtlinien / File-Scoped Guidelines

<table>
<tr>
<th width="50%">DE Was sind Instructions?</th>
<th width="50%">EN What are Instructions?</th>
</tr>
<tr>
<td>Instructions sind Richtlinien, die automatisch geladen werden, wenn Copilot an Dateien arbeitet, die dem <code>applyTo</code>-Glob-Muster entsprechen. Sie ergänzen die globalen Regeln mit datei-spezifischen Details.</td>
<td>Instructions are guidelines that are automatically loaded when Copilot works on files matching the <code>applyTo</code> glob pattern. They extend the global rules with file-type-specific details.</td>
</tr>
</table>

### Vorhandene Instructions / Available Instructions

| Datei / File | `applyTo` | Zweck / Purpose |
|---|---|---|
| `nextjs.instructions.md` | `**/*.tsx, **/*.ts, **/*.jsx, **/*.js` | Next.js 16 App Router Patterns, async APIs, Server/Client Components |
| `tailwind.instructions.md` | `**/*.tsx, **/*.jsx, **/*.css` | Tailwind CSS v4 Utility-First Patterns, Responsive Design |
| `typescript.instructions.md` | `**/*.ts, **/*.tsx` | TypeScript Strict Mode, Naming Conventions, Type Safety |
| `supabase.instructions.md` | `**/supabase/**, **/*supabase*` | Supabase Client Setup, RLS, Migrations, Schema Awareness |

### Eigene Instructions erstellen / Creating Custom Instructions

<table>
<tr>
<th width="50%">DE Anleitung</th>
<th width="50%">EN Guide</th>
</tr>
<tr>
<td>Erstelle eine <code>.instructions.md</code>-Datei in <code>.github/instructions/</code> mit YAML-Frontmatter. Das <code>applyTo</code>-Feld akzeptiert Glob-Muster und bestimmt, für welche Dateien die Regeln gelten.</td>
<td>Create a <code>.instructions.md</code> file in <code>.github/instructions/</code> with YAML frontmatter. The <code>applyTo</code> field accepts glob patterns and determines which files the rules apply to.</td>
</tr>
</table>

```yaml
---
applyTo: "**/*.tsx"
---
# Your instructions here
```

---

## Agents — KI-Personas / AI Personas

<table>
<tr>
<th width="50%">DE Was sind Agents?</th>
<th width="50%">EN What are Agents?</th>
</tr>
<tr>
<td>Agents sind spezialisierte KI-Personas. Du rufst sie in VS Code über das Chat-Panel mit <code>@agent-name</code> auf. Jeder Agent hat eine klar definierte Rolle, einen Workflow und Regeln.</td>
<td>Agents are specialised AI personas. You invoke them in VS Code via the Chat panel using <code>@agent-name</code>. Each agent has a clearly defined role, workflow, and rules.</td>
</tr>
</table>

---

### `@planner` — Planer

<table>
<tr>
<th width="50%">DE</th>
<th width="50%">EN</th>
</tr>
<tr>
<td><strong>Zweck:</strong> Technische Planung und Anforderungsanalyse vor dem Coding.</td>
<td><strong>Purpose:</strong> Technical planning and requirements analysis before coding.</td>
</tr>
<tr>
<td><strong>Wann verwenden:</strong><br>— Vor dem Start eines neuen Features<br>— Wenn die Anforderungen unklar sind<br>— Vor Refactoring oder Migrations-Aufgaben</td>
<td><strong>When to use:</strong><br>— Before starting a new feature<br>— When requirements are unclear<br>— Before refactoring or migration tasks</td>
</tr>
<tr>
<td><strong>Was er tut:</strong><br>✔ Codebase erkunden und Kontext sammeln<br>✔ Betroffene Dateien und Module identifizieren<br>✔ Nummerierten Implementierungsplan erstellen<br>✘ <strong>Schreibt keinen Code</strong></td>
<td><strong>What it does:</strong><br>✔ Explore codebase and gather context<br>✔ Identify affected files and modules<br>✔ Create a numbered implementation plan<br>✘ <strong>Does not write code</strong></td>
</tr>
<tr>
<td><strong>Verwendung:</strong> <code>@planner Füge eine Authentifizierungsseite hinzu</code></td>
<td><strong>Usage:</strong> <code>@planner Add an authentication page</code></td>
</tr>
</table>

---

### `@developer` — Entwickler

<table>
<tr>
<th width="50%">DE</th>
<th width="50%">EN</th>
</tr>
<tr>
<td><strong>Zweck:</strong> Strukturierte Feature-Implementierung mit Qualitätsprüfungen.</td>
<td><strong>Purpose:</strong> Structured feature implementation with quality checks.</td>
</tr>
<tr>
<td><strong>Wann verwenden:</strong><br>— Nach der Planung mit <code>@planner</code><br>— Zur Implementierung von Features, Seiten, API-Routen<br>— Für Datenbankänderungen und Migrationen</td>
<td><strong>When to use:</strong><br>— After planning with <code>@planner</code><br>— To implement features, pages, API routes<br>— For database changes and migrations</td>
</tr>
<tr>
<td><strong>Vierphasiger Prozess:</strong><br>1. <strong>Vorbereitung</strong> — Plan lesen, Code verstehen<br>2. <strong>Implementierung</strong> — Schrittweise, Fehler sofort beheben<br>3. <strong>Verifikation (PFLICHT)</strong> — <code>typecheck</code> → <code>lint</code> → <code>build</code> → lokal testen<br>4. <strong>Dokumentation</strong> — Handbook aktualisieren</td>
<td><strong>Four-phase process:</strong><br>1. <strong>Preparation</strong> — Read plan, understand code<br>2. <strong>Implementation</strong> — Step by step, fix errors immediately<br>3. <strong>Verification (MANDATORY)</strong> — <code>typecheck</code> → <code>lint</code> → <code>build</code> → test locally<br>4. <strong>Documentation</strong> — Update handbook</td>
</tr>
<tr>
<td><strong>Verwendung:</strong> <code>@developer Implementiere diesen Plan: [Plan einfügen]</code></td>
<td><strong>Usage:</strong> <code>@developer Implement this plan: [paste plan]</code></td>
</tr>
</table>

---

### `@reviewer` — Code-Reviewer

<table>
<tr>
<th width="50%">DE</th>
<th width="50%">EN</th>
</tr>
<tr>
<td><strong>Zweck:</strong> Code-Review und Qualitätssicherung vor dem PR.</td>
<td><strong>Purpose:</strong> Code review and quality assurance before a PR.</td>
</tr>
<tr>
<td><strong>Wann verwenden:</strong><br>— Bevor ein PR erstellt wird<br>— Nach einer Implementierung zur Qualitätsprüfung<br>— Zur Sicherheits- und Performance-Analyse</td>
<td><strong>When to use:</strong><br>— Before creating a PR<br>— After implementation for quality check<br>— For security and performance audits</td>
</tr>
<tr>
<td><strong>Was er tut:</strong><br>✔ Strukturierte Checkliste (Qualität, Next.js 16, Supabase-Sicherheit, Styling)<br>✔ Alle Checks ausführen (typecheck, lint, build)<br>✔ Review-Bericht mit Status (✅ / ⚠️ / ❌)</td>
<td><strong>What it does:</strong><br>✔ Structured checklist (quality, Next.js 16, Supabase security, styling)<br>✔ Run all checks (typecheck, lint, build)<br>✔ Review report with status (✅ / ⚠️ / ❌)</td>
</tr>
<tr>
<td><strong>Verwendung:</strong> <code>@reviewer Überprüfe die Änderungen in src/app/dashboard/</code></td>
<td><strong>Usage:</strong> <code>@reviewer Review the changes in src/app/dashboard/</code></td>
</tr>
</table>

---

## Empfohlener Workflow / Recommended Workflow

<table>
<tr>
<th width="50%">DE Schritt</th>
<th width="50%">EN Step</th>
</tr>
<tr>
<td>1. <strong><code>@planner</code></strong> — Aufgabe analysieren und Plan erstellen lassen</td>
<td>1. <strong><code>@planner</code></strong> — Let the Planner analyse the task and create a plan</td>
</tr>
<tr>
<td>2. <strong><code>@developer</code></strong> — Plan an den Developer übergeben zur Implementierung</td>
<td>2. <strong><code>@developer</code></strong> — Hand the plan to the Developer for implementation</td>
</tr>
<tr>
<td>3. <strong><code>@reviewer</code></strong> — Code prüfen lassen, bevor du einen PR erstellst</td>
<td>3. <strong><code>@reviewer</code></strong> — Let the Reviewer check the code before creating a PR</td>
</tr>
</table>

---

## Eigene Agents erstellen / Creating Custom Agents

<table>
<tr>
<th width="50%">DE Anleitung</th>
<th width="50%">EN Guide</th>
</tr>
<tr>
<td>Erstelle eine <code>.agent.md</code>-Datei in <code>.github/agents/</code> mit YAML-Frontmatter. Definiere Rolle, Workflow und Regeln.</td>
<td>Create a <code>.agent.md</code> file in <code>.github/agents/</code> with YAML frontmatter. Define role, workflow, and rules.</td>
</tr>
</table>

```yaml
---
name: MyAgent
description: >
  Description of what this agent does.
---
# Agent: MyAgent

## Role
...

## Workflow
...
```

---

## Wenn ein Agent schlecht antwortet / When an Agent Responds Poorly

<table>
<tr>
<th width="50%">DE Problem & Lösung</th>
<th width="50%">EN Problem & Solution</th>
</tr>
<tr>
<td><strong>Agent ignoriert Regeln aus der Instructions-Datei</strong><br>→ Prüfe das <code>applyTo</code>-Glob-Muster — stimmt es mit der Datei überein, die du bearbeitest? Teste mit: <code>**/*.ts</code> statt <code>src/**/*.ts</code></td>
<td><strong>Agent ignores rules from an Instructions file</strong><br>→ Check the <code>applyTo</code> glob pattern — does it match the file you are editing? Try broader patterns: <code>**/*.ts</code> instead of <code>src/**/*.ts</code></td>
</tr>
<tr>
<td><strong>Agent befolgt den Workflow nicht (z. B. überspringt typecheck)</strong><br>→ Öffne die <code>.agent.md</code>-Datei und mache die Anweisung strikter. Ersetze "sollte" durch "muss". Füge am Ende eine Zusammenfassung hinzu: <em>"Bevor du antwortest, liste alle abgeschlossenen Schritte auf."</em></td>
<td><strong>Agent does not follow the workflow (e.g. skips typecheck)</strong><br>→ Open the <code>.agent.md</code> file and make the instruction stricter. Replace "should" with "must". Add a reminder at the end: <em>"Before responding, list all completed steps."</em></td>
</tr>
<tr>
<td><strong>Agent schreibt schlechten Next.js-Code (z. B. falsche API-Nutzung)</strong><br>→ Füge ein konkretes Beispiel in die <code>nextjs.instructions.md</code> ein. Copilot folgt Beispielen besser als abstrakten Regeln.</td>
<td><strong>Agent writes bad Next.js code (e.g. wrong API usage)</strong><br>→ Add a concrete code example to <code>nextjs.instructions.md</code>. Copilot follows examples better than abstract rules.</td>
</tr>
<tr>
<td><strong>Agent "vergisst" den Kontext nach langen Gesprächen</strong><br>→ Starte ein neues Chat-Fenster. Langer Kontext verdrängt Instructions. Übergib den Plan explizit: <em>"Hier ist der Plan: [Plan]. Bitte implementiere Schritt 3."</em></td>
<td><strong>Agent "forgets" context after long conversations</strong><br>→ Start a new chat window. Long context pushes out instructions. Pass the plan explicitly: <em>"Here is the plan: [plan]. Please implement step 3."</em></td>
</tr>
<tr>
<td><strong>Agent antwortet auf Englisch statt Deutsch (oder umgekehrt)</strong><br>→ Füge in <code>copilot-instructions.md</code> eine Sprachanweisung hinzu: <em>"Antworte immer auf Deutsch."</em> — oder sprich den Agent in der gewünschten Sprache an.</td>
<td><strong>Agent responds in German instead of English (or vice versa)</strong><br>→ Add a language instruction to <code>copilot-instructions.md</code>: <em>"Always respond in English."</em> — or address the agent in your preferred language.</td>
</tr>
<tr>
<td><strong>Agent überschreitet den Plan / macht ungebetene Änderungen</strong><br>→ Füge in der <code>.agent.md</code> unter "Rules" hinzu: <em>"Ändere nur Dateien, die explizit im Plan genannt sind. Keine ungebetenen Refactors."</em></td>
<td><strong>Agent exceeds the plan / makes unrequested changes</strong><br>→ Add to the <code>.agent.md</code> under "Rules": <em>"Only modify files explicitly listed in the plan. No unrequested refactoring."</em></td>
</tr>
</table>

---

## Referenzen / References

- [GitHub Copilot Customisation Docs](https://docs.github.com/en/copilot/customizing-copilot)
- [awesome-copilot](https://github.com/github/awesome-copilot) — Beispiele und Best Practices / Examples and best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wamocon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Wamocon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
