## ibeia-skills

> Repositório que hospeda skills do Claude Code para publicação no GitHub. Este arquivo explica como o repo funciona por dentro.

# CLAUDE.md — ibeia-skills

Repositório que hospeda skills do Claude Code para publicação no GitHub. Este arquivo explica como o repo funciona por dentro.

## Fonte única de verdade (junction, não cópia)

As skills **moram neste repositório**, em `skills/`. Na máquina do mantenedor, o diretório global do Claude aponta para cá via *junction* do Windows:

```
~/.claude/skills/dont-make-me-think  ──junction──►  <repo>/skills/dont-make-me-think
~/.claude/skills/gamification        ──junction──►  <repo>/skills/gamification
~/.claude/skills/loop-goal           ──junction──►  <repo>/skills/loop-goal
```

Consequência importante: **não existe etapa de sincronização**. Editar a skill "no global" e editá-la "no repo" é o mesmo arquivo físico. Depois de editar, basta `commit` + `push` para publicar.

O Git enxerga **conteúdo real** dentro de `skills/` (a junction fica do lado do `~/.claude`, não dentro do repo), então o repositório é totalmente utilizável por quem clonar.

## Estrutura

```
ibeia-skills/
├── README.md               ← visão pública + instruções de instalação
├── CLAUDE.md               ← este arquivo
├── setup-junctions.ps1     ← recria as junctions em uma nova máquina
├── .gitignore
└── skills/
    ├── dont-make-me-think/  (SKILL.md + reference/)
    ├── gamification/        (SKILL.md + reference/)
    └── loop-goal/           (SKILL.md)
```

Cada skill segue o padrão do Claude Code: um `SKILL.md` com frontmatter (`name`, `description`) e, opcionalmente, uma pasta `reference/` com arquivos carregados sob demanda.

## Fluxo de trabalho

1. **Editar** — altere os arquivos diretamente em `skills/` (ou via `~/.claude/skills/<skill>`, é o mesmo arquivo). Nunca duplique nem copie conteúdo entre global e repo: eles são unificados por junction.
2. **Commitar** — um commit por unidade lógica; mensagens em conventional commits (`feat:`, `fix:`, `docs:`…).
3. **Publicar** — `git push` apenas quando quiser publicar. Push é sempre manual.

## Nova máquina: recriar as junctions

Ao clonar em outra máquina Windows, rode uma vez para religar o global a este repo:

```powershell
./setup-junctions.ps1
```

O script cria/atualiza uma junction em `~/.claude/skills/<skill>` para cada pasta em `skills/`. Ele nunca apaga uma pasta real — só recria links.

## Regras

- **Não copiar/duplicar skills.** A junction garante fonte única; cópias divergem.
- **Editar in-place**, mantendo o estilo e a versão de cada skill.
- **Push só a pedido.** O commit local pode ser automático; a publicação, não.

---
> Source: [renatoasse/ibeia-skills](https://github.com/renatoasse/ibeia-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
