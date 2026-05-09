## genesis-meta-system

> > **Sistema Gerador de Sistemas** - Transforma descricoes de dominios em sistemas Claude Code Native completos e funcionais.

# GENESIS - Meta-Sistema Claude Code Native

> **Sistema Gerador de Sistemas** - Transforma descricoes de dominios em sistemas Claude Code Native completos e funcionais.

---

## PRIMEIRA ACAO EM QUALQUER SESSAO

```
1. LER: STATE.yaml
2. VERIFICAR: current.phase e current.next_action
3. USAR: Comandos slash /GENESIS:tasks:*
4. GERAR: Novos sistemas com /GENESIS:tasks:generate-system
```

**Versao Atual:** 2.0.0 (State-of-Art)
**Estado:** Ver STATE.yaml
**Standalone:** Este e um projeto GENESIS standalone pronto para uso

---

## Overview

GENESIS e um **Meta-Sistema** que gera sistemas Claude Code Native completos a partir de descricoes de dominio (domain briefs).

**Core Transformation:**
```
DE: Descricao de dominio (texto livre ou estruturado)
PARA: Sistema Claude Code Native funcional com:
      - Agents especializados
      - Tasks executaveis
      - Workflows orquestrados
      - Knowledge bases
      - Slash commands
      - Validacao automatica
```

**Domain Type:** HYBRID (COGNITIVE + TECHNICAL)

---

## Status v2.0.0 (State-of-Art)

### Componentes

| Componente | Total | Status |
|------------|-------|--------|
| Agents | 5 | COMPLETE |
| Tasks (pipeline) | 22 | COMPLETE |
| Templates | 6 | COMPLETE |
| Knowledge | 12 | COMPLETE |
| Slash Commands | **11** | COMPLETE |
| Skills | 1 | COMPLETE |
| Hooks | 3 | READY |
| SOPs | 4 | COMPLETE |
| Test Suite | 20 tests | COMPLETE |

---

## Comandos Disponiveis (11 total)

### Comandos Principais (v2.0.0 State-of-Art)

```bash
# Gerar sistema completo (v2.0.0)
# - TodoWrite para tracking visual
# - AskUserQuestion em todos os gates
# - Task tool para geracao paralela
# - Extended thinking para classificacao/arquitetura
/GENESIS:tasks:generate-system

# Validar sistema com scoring REAL (v2.0.0)
# - Glob para contagem de arquivos
# - Grep para verificacao de referencias
# - Calculo automatico de scores
/GENESIS:tasks:validate-system
```

### Comandos de Gerenciamento

```bash
# Listar sistemas gerados
/GENESIS:tasks:list-systems

# Ver detalhes de um sistema
/GENESIS:tasks:show-system

# Exportar sistema para deploy
/GENESIS:tasks:export-system

# Deletar sistema (com confirmacao)
/GENESIS:tasks:delete-system
```

### Comandos de Controle

```bash
# Ver status atual
/GENESIS:tasks:check-status

# Retomar geracao interrompida
/GENESIS:tasks:resume-generation

# Abortar geracao
/GENESIS:tasks:abort-generation

# Executar test suite (20 testes)
/GENESIS:tasks:run-tests
```

### Ativacao de Agente

```bash
# Ativar meta-orchestrator
@meta-orchestrator
```

---

## Features State-of-Art (v2.0.0)

### 1. TodoWrite Integration
```yaml
# Ao executar /generate-system:
- Cria 11 items de tracking (fases + gates)
- Atualiza status em tempo real
- Usuario ve progresso visual
```

### 2. AskUserQuestion em Gates
```yaml
# Em cada gate critico:
- G0: Validar entendimento do brief
- G1: Confirmar classificacao de dominio
- G2: Aprovar arquitetura proposta
- G8: Aprovacao final do sistema
```

### 3. Task Tool para Paralelismo
```yaml
# Fases P3-P6 rodam em paralelo:
- P3: Gerar agents simultaneamente
- P4: Gerar workflows
- P5: Gerar tasks
- P6: Gerar arquivos de suporte
```

### 4. Extended Thinking
```yaml
# Em decisoes criticas:
- P1 (Classify): Analisa multiplos fatores
- P2 (Structure): Considera tradeoffs de arquitetura
```

### 5. Scoring Automatico Real
```yaml
# /validate-system usa:
- Glob: Conta arquivos reais (nao estimativas)
- Grep: Verifica referencias existem
- Formula: Calcula scores numericos
```

### 6. Test Suite Completo
```yaml
# /run-tests executa:
- 5 smoke tests
- 5 generation tests
- 5 validation tests
- 5 utility tests
```

---

## Estrutura de Arquivos (Standalone)

```
D:/deploy-ready/genesis/
├── STATE.yaml              # CONSCIENCIA - ler SEMPRE primeiro
├── CLAUDE.md               # Este arquivo
├── README.md               # Documentacao principal
├── config.yaml             # Configuracao do sistema
│
├── .claude/
│   ├── commands/GENESIS/
│   │   ├── tasks/              # 11 slash commands
│   │   │   ├── generate-system.md      (v2.0.0)
│   │   │   ├── validate-system.md      (v2.0.0)
│   │   │   ├── check-status.md
│   │   │   ├── resume-generation.md
│   │   │   ├── abort-generation.md
│   │   │   ├── list-systems.md
│   │   │   ├── show-system.md
│   │   │   ├── export-system.md
│   │   │   ├── delete-system.md
│   │   │   └── run-tests.md
│   │   └── agents/
│   │       └── meta-orchestrator.md
│   ├── skills/
│   │   └── genesis-core.md
│   └── settings.local.json
│
├── agents/                 # 5 agents especializados
│   ├── domain-analyst.md
│   ├── system-architect.md
│   ├── component-generator.md
│   ├── system-validator.md
│   └── meta-orchestrator.md
│
├── tasks/                  # 22 tasks do pipeline
│   ├── 00-interpret/       # P0: Interpret
│   ├── 01-classify/        # P1: Classify
│   ├── 02-structure/       # P2: Structure
│   ├── 03-agents/          # P3: Agents
│   ├── 04-workflows/       # P4: Workflows
│   ├── 05-tasks/           # P5: Tasks
│   ├── 06-support/         # P6: Support
│   ├── 07-generate/        # P7: Generate
│   └── 08-validate/        # P8: Validate
│
├── workflows/              # Orquestracao
│   └── main-workflow.yaml
│
├── templates/              # Templates para geracao
│   ├── system-output/      # 6 templates
│   │   ├── agent-template.md
│   │   ├── task-template.md
│   │   ├── claude-md-template.md
│   │   ├── workflow-template.yaml
│   │   ├── config-template.yaml
│   │   └── settings-template.json
│   └── INSTRUCTIONS.md     # Guia de preenchimento
│
├── knowledge/              # 12 arquivos de conhecimento
│   ├── domain-types.yaml
│   ├── component-patterns.yaml
│   ├── agent-personas.yaml
│   ├── workflow-patterns.yaml
│   ├── anti-patterns.yaml
│   ├── glossary.yaml
│   ├── principles-reference.md
│   ├── decomposition-rules.yaml
│   ├── checklist-patterns.yaml
│   ├── elicitation-patterns.yaml
│   ├── task-tool-patterns.yaml
│   └── examples/
│
├── hooks/                  # 3 hooks de automacao
│   ├── install-hooks.sh
│   ├── pre-commit-validate.sh
│   ├── post-generate-check.sh
│   └── README.md
│
├── ops/                    # Operacoes
│   ├── sops/               # 4 SOPs
│   └── checklists/
│
├── checklists/             # Quality checklists
│   └── quality/
│
├── docs/                   # Documentacao completa
│   ├── README.md
│   ├── QUICK-START.md
│   ├── USER-GUIDE.md
│   ├── EXAMPLES.md
│   ├── TROUBLESHOOTING.md
│   ├── FAQ.md
│   ├── CHANGELOG.md
│   ├── ARCHITECTURE.md
│   ├── CONTRIBUTING.md
│   ├── API-REFERENCE.md
│   ├── PRINCIPLES.md
│   └── STATE-OF-ART-FEATURES.md
│
└── outputs/                # Generated systems go here
    └── systems/            # Each generated system
```

---

## Pipeline de Geracao (8 Fases)

```
┌─────────────────────────────────────────────────────────────────┐
│                    GENESIS PIPELINE v2.0                        │
└─────────────────────────────────────────────────────────────────┘

  P0: INTERPRET ──► G0 ──► P1: CLASSIFY ──► G1 ──► P2: STRUCTURE ──► G2
       │                        │                        │
       └── TodoWrite           └── Extended             └── Extended
           initialized             Thinking                 Thinking

  ──► P3-P6: GENERATE (PARALLEL) ──► G7 ──► P7: ASSEMBLE ──► P8: VALIDATE ──► G8
           │                              │                        │
           └── Task tool                  └── Files               └── Glob/Grep
               (4 parallel                    written                 scoring
                blocks)

Gates:
  G0: Brief understanding (AskUserQuestion)
  G1: Classification confirmation (AskUserQuestion - GO/NO-GO)
  G2: Architecture approval (AskUserQuestion - GO/NO-GO)
  G7: Pre-assembly validation
  G8: Final approval (AskUserQuestion - CRITICAL)
```

---

## Agents

| Agent | Papel | Fases |
|-------|-------|-------|
| @meta-orchestrator | Coordena todo o pipeline | Todas |
| @domain-analyst | Classifica dominios | P0, P1 |
| @system-architect | Projeta arquitetura | P2, P3, P4 |
| @component-generator | Gera arquivos | P5, P6, P7 |
| @system-validator | Valida qualidade | P8 |

---

## Principios Fundamentais (P1-P20)

### Criticos (Sempre Aplicados)

| Principio | Descricao | Implementacao |
|-----------|-----------|---------------|
| P1 | Parallelism-First | Task tool em P3-P6 |
| P2 | State-Externalized | STATE.yaml |
| P4 | Checkpoints-Strategic | 5 gates no pipeline |
| P12 | Clarify-Before-Execute | AskUserQuestion |
| P18 | Elicitation-Non-Negotiable | Gates nunca pulados |

### Todos os 20 Principios

Ver completo: `knowledge/principles-reference.md`

---

## Anti-Patterns

```python
NEVER_DO = [
    "skip_checkpoint",           # NUNCA pular gates
    "assume_classification",     # SEMPRE confirmar tipo
    "broken_references",         # NUNCA referenciar inexistente
    "estimate_scores",           # SEMPRE calcular com Glob/Grep
    "ignore_state",              # SEMPRE ler STATE.yaml primeiro
    "manual_parallelism",        # SEMPRE usar Task tool
    "skip_todowrite",            # SEMPRE tracking visual
]
```

---

## Validacao de Qualidade

### Metricas

| Metrica | Threshold | Target | Status |
|---------|-----------|--------|--------|
| Completeness | >= 95% | 98% | PASS |
| Consistency | = 100% | 100% | PASS |
| Principles | >= 90% | 95% | PASS |
| **Overall** | >= 89% | **~98%** | **PASS** |

### Scoring Automatico

```python
# Formula em validate-system.md v2.0.0:
COMPLETENESS = (files_found / files_expected) * 100
CONSISTENCY = (valid_refs / total_refs) * 100
PRINCIPLES = (passed_checks / 20) * 100
OVERALL = (COMPLETENESS * 0.35) + (CONSISTENCY * 0.35) + (PRINCIPLES * 0.30)
```

---

## Quick Start

```bash
# 1. Abrir projeto no Claude Code
cd D:/deploy-ready/genesis
claude

# 2. Gerar novo sistema
/GENESIS:tasks:generate-system

# 3. Validar sistema
/GENESIS:tasks:validate-system outputs/systems/my-system/

# 4. Listar sistemas
/GENESIS:tasks:list-systems
```

---

## Quick Reference

```bash
# Iniciar nova sessao
cat STATE.yaml

# Gerar novo sistema
/GENESIS:tasks:generate-system

# Validar sistema existente
/GENESIS:tasks:validate-system outputs/systems/my-system/

# Listar sistemas
/GENESIS:tasks:list-systems

# Executar testes
/GENESIS:tasks:run-tests

# Ativar orquestrador
@meta-orchestrator
```

---

*GENESIS v2.0.0 - State-of-Art CERTIFIED*
*Meta-Sistema Claude Code Native*
*Score: 98% | 11 comandos | 20 testes | Standalone Edition*
*Ler STATE.yaml para estado atual*

---
> Source: [joaolozano-lendario/genesis-meta-system](https://github.com/joaolozano-lendario/genesis-meta-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
