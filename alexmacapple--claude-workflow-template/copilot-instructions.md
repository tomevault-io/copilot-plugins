## claude-workflow-template

> **Workspace** : (votre workspace)

# Claude Workspace - Memoire globale

**Workspace** : (votre workspace)

---

## Vision

Orchestrer efficacement les projets avec Claude Code en appliquant le principe de **contexte minimal** et une **memoire hierarchique**.

**Heritage** : Ce fichier herite de la memoire personnelle `~/.claude/CLAUDE.md` (langue, style, preferences).

**Reference technique** : Voir `GUIDELINES-CLAUDE-CODE.MD` pour l'architecture detaillee (Skills, Agents, Rules, hooks).

---

## Structure du workspace

```
./
├── CLAUDE.MD                    # Cette memoire (vous etes ici)
├── GUIDELINES-CLAUDE-CODE.MD    # Reference technique Claude Code
├── .claude/
│   ├── skills/                  # Workflows invocables (/commande)
│   ├── agents/                  # Sous-agents specialises
│   ├── rules/                   # Regles par domaine (auto-chargees)
│   └── commands/                # Slash commands
├── docs-public/                 # Documentation publique
├── active/                      # Projets en cours
├── templates/                   # Templates reutilisables
└── archive/                     # Projets termines
```

---

## Commandes disponibles

### Skills principaux

- `/review [fichier]` : Evalue un skill, PRD ou rule avec score /100, tableau de conformites et actions correctives. Grilles de notation integrees (PRD, Skill, Rule).
- `/verifier-regles-md` : Audit de conformite Markdown complet

### Autres commandes

- `/init nom-du-projet` : Initialiser un nouveau projet dans `active/`
- `./check-markdown-rules.sh` : Verification CLI rapide (un fichier ou tout le workspace)

---

## Conventions

- **Projets** : `kebab-case` (ex: `mon-super-projet`)
- **Skills** : `kebab-case` dans `.claude/skills/*/SKILL.md`
- **Agents** : `kebab-case-agent.md` dans `.claude/agents/`
- **Rules** : `domain-rules.md` dans `.claude/rules/`
- **Commits** : Style conventionnel (voir `git-rules.md`)

---

## Personnalisation

Adaptez ce fichier a votre contexte :
- Ajoutez vos conventions specifiques
- Documentez vos commandes personnalisees
- Referencez vos projets actifs

---

**Version** : 1.1.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alexmacapple) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
