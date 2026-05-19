## prompt-engineering

> **Author:** Anderson Henrique da Silva

# AI.md - Entry Point do Laboratório

**Author:** Anderson Henrique da Silva
**Location:** Minas Gerais, Brasil

> "Eu sou o mapa. Leia a Constituição, depois escolha um Agente."

---

## O Que É Este Repositório

Este é o **Prompt Engineering Lab** - um laboratório de pesquisa científica sobre Human-AI Collaboration.

**Objetivo:** Produzir um paper acadêmico sobre colaboração humano-IA, com metodologia reproduzível e artefatos concretos.

**Fundações:**
- **DMMF** - Developer Mental Model Framework (perfil cognitivo)
- **DSRM** - Design Science Research Methodology (workflow de pesquisa)
- **4D Framework** - AI Fluency da Anthropic (governance)

---

## Antes de Começar

### 1. Leia a Constituição
```
governance/constitution.md
```
Contém as leis do laboratório: o que AI pode/não pode fazer, padrões de qualidade, tech stack.

### 2. Entenda o Contexto Pessoal (se disponível)
```
artifacts/prompts/personal/SKILL.md
```
Quem é o pesquisador, como prefere se comunicar, estilo cognitivo.

### 3. Escolha um Agente (se aplicável)
```
artifacts/agents/
├── santos-dumont.md    # Tech Lead - Arquitetura e delegação
├── machado-de-assis.md # Documentarista - Comunicação
├── vital-brazil.md     # QA - Validação e testes
├── aleijadinho.md      # Code Reviewer - Refinamento
├── coimbra.md          # DevOps - Automação
└── nise-da-silveira.md # Neural Pattern Analyst
```

---

## Estrutura do Laboratório

```
prompt-engineering-lab/
│
├── AI.md                      ← VOCÊ ESTÁ AQUI (Entry Point)
│
├── governance/                # LEIS DO LABORATÓRIO
│   └── constitution.md        # Tech stack, regras, limites, 4D Framework
│
├── artifacts/                 # ARTEFATOS PRODUZIDOS
│   ├── agents/                # Agentes brasileiros (personas)
│   ├── prompts/
│   │   ├── personal/          # Contexto pessoal (gitignored)
│   │   ├── technical/         # Skills do marketplace + custom
│   │   └── legacy/            # Prompts antigos (migração)
│   └── workflows/             # Processos orquestrados
│
├── src/                       # CÓDIGO PYTHON
│   ├── metrics/               # Análise de interações
│   ├── calibration/           # Dashboard e auto-calibração
│   ├── experiments/           # A/B testing framework
│   ├── versioning/            # Gerenciamento de versões
│   └── pipeline/              # Orquestração de componentes
│
├── evidence/                  # BASE DE EVIDÊNCIAS
│   ├── patterns/              # Padrões efetivos identificados
│   ├── antipatterns/          # O que não funciona
│   ├── models/                # Modelos de análise
│   └── sources/               # Fontes externas (anthropic-cookbook, etc)
│
├── experiments/               # A/B TESTING
│   ├── hypothesis/            # Hipóteses a testar
│   └── results/               # Resultados
│
├── research/                  # METODOLOGIA & DADOS
│   ├── methodology/           # DMMF, DSRM, 4D docs
│   ├── data/                  # Dados de pesquisa
│   └── analysis/              # Scripts de análise
│
├── papers/                    # OUTPUT CIENTÍFICO
│   └── human-ai-collaboration/
│
├── tests/                     # VALIDAÇÃO (pytest)
│
├── docs/                      # DOCUMENTAÇÃO
│   ├── architecture/          # Decisões arquiteturais
│   └── references/            # PDFs, livros, artigos
│
└── .config/                   # CONFIGURAÇÕES DE FERRAMENTAS
    ├── .clinerules
    ├── .cursorrules
    └── GEMINI.md
```

---

## Fluxo de Trabalho (DSRM)

```
┌─────────────────────────────────────────────────────────────┐
│                    CICLO DE PESQUISA                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. PROBLEMA        → research/methodology/                 │
│       ↓                                                     │
│  2. OBJETIVOS       → papers/*/outline.md                   │
│       ↓                                                     │
│  3. DESIGN          → artifacts/ (Santos Dumont lidera)     │
│       ↓                                                     │
│  4. DEMONSTRAÇÃO    → experiments/ (Coimbra executa)        │
│       ↓                                                     │
│  5. AVALIAÇÃO       → tests/ + evidence/ (Vital Brazil)     │
│       ↓                                                     │
│  6. COMUNICAÇÃO     → papers/ (Machado de Assis)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Start

### Para Tarefas de Código
```bash
# 1. Ative o ambiente
source .venv/bin/activate

# 2. Rode os testes (Vital Brazil aprova)
pytest tests/ -v

# 3. Execute pipeline
python -m src.pipeline.integration_pipeline --full
```

### Para Documentação
```
1. Leia governance/constitution.md
2. Ative Machado de Assis (artifacts/agents/machado-de-assis.md)
3. Siga os padrões de comunicação
```

### Para Arquitetura/Design
```
1. Leia governance/constitution.md
2. Ative Santos Dumont (artifacts/agents/santos-dumont.md)
3. Documente decisões em docs/architecture/
```

### Para Qualidade/Testes
```
1. Leia governance/constitution.md
2. Ative Vital Brazil (artifacts/agents/vital-brazil.md)
3. Mantenha coverage >76%
```

---

## Agentes Disponíveis

| Agente | Papel | Quando Usar |
|--------|-------|-------------|
| **Santos Dumont** | Tech Lead | Arquitetura, decisões técnicas, delegação |
| **Machado de Assis** | Documentarista | README, docs, comunicação, papers |
| **Vital Brazil** | QA Engineer | Testes, validação, qualidade |
| **Aleijadinho** | Code Reviewer | Refatoração, code review, refinamento |
| **Coimbra** | DevOps | CI/CD, automação, deploy |
| **Nise da Silveira** | Neural Analyst | Padrões cognitivos, análise de interações |

---

## Regras de Ouro (4D Framework)

### Delegation
> AI faz trabalho repetitivo. Humano decide e valida.

### Description
> Seja explícito. Contexto → Objetivo → Restrições → Formato.

### Discernment
> Questione outputs. Verifique fatos. Teste antes de integrar.

### Diligence
> Sem secrets em prompts. Sem código não testado. Sem autoria falsa.

---

## Links Importantes

- **Constituição:** `governance/constitution.md`
- **Paper Principal:** `papers/human-ai-collaboration/outline.md`
- **Agentes:** `artifacts/agents/`
- **Testes:** `pytest tests/ -v`
- **Pipeline:** `python -m src.pipeline.integration_pipeline`

---

## Autor

**Anderson Henrique da Silva**
- Localização: Minas Gerais, Brasil
- Background: Filosofia → Ciência da Computação
- Foco: AI/ML Engineering, Multi-Agent Systems, AI Safety

---

*"O laboratório é a oficina. Os agentes são as ferramentas. O paper é o produto."*

---
> Source: [anderson-ntlabs/prompt-engineering](https://github.com/anderson-ntlabs/prompt-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
