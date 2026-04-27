## agent-runner

> Agent Runner — motor de desenvolvimento autônomo com RALPH LOOP. Ativar com /agent-runner ou ao pedir para construir um projeto.

# /agent-runner — Agent Runner

Ao receber este comando com a descrição do projeto, execute o RALPH LOOP completo com autonomia total. Não peça confirmação em nenhuma etapa.

## Autonomia Total — Regra #0

O agente tem **autonomia completa** para TODAS as operações dentro do escopo do projeto: criar/editar/deletar arquivos, executar comandos de terminal (mkdir, cd, yarn, npm, pip, go, cargo, git), instalar dependências, executar scripts, operações git. **NUNCA peça confirmação.** Escopo: `meta.app_dir` + `workspace/`. Proibido: arquivos do sistema, push sem instrução.

---

## Passo 0 — Carregar Contexto (OBRIGATÓRIO, SEMPRE)

Leia nesta ordem exata antes de qualquer ação:
1. `workspace/memory/agent-brain.md` — memória acumulada de todos os projetos (anti-padrões, regras, aprendizados)
2. `workspace/memory/snapshots/latest.md` — estado da sessão anterior (se existir)
3. `workspace/memory/global.md` — regras globais do ambiente
4. Se houver projeto ativo: `workspace/memory/[projeto].md`

Só após ler o contexto acima, prossiga.

---

## Passo 1 — Fase PLAN (única — não replaneie depois)

Leia `agent/prompts/plan.md` e execute como time completo:

**Analista** → **Arquiteto** → **Designer** → **Task Breakdown**

Produza obrigatoriamente:
- `workspace/[projeto]/PRD.md` — requisitos, personas, user stories, fluxo de telas
- `workspace/[projeto]/requirements.md` — lista detalhada de RFs e RNFs
- `workspace/[projeto]/prd.json` — tarefas atômicas com `instructions` auto-suficientes
- `workspace/[projeto]/design-system.md` — se `has_ui: true`: contrato visual completo (CSS vars, paleta, fontes, componentes)

O `prd.json` tem seção `meta` com os comandos da stack e cada tarefa tem campo `type`:
```json
{
  "meta": {
    "project": "...", "stack": "...", "workspace_dir": "workspace/[nome-do-projeto]", "app_dir": "~/projects/[nome-do-projeto]",
    "check_cmd": "...", "test_cmd": "...", "lint_cmd": "...", "run_cmd": "...",
    "has_ui": true,
    "visual_check_cmd": "grep -c 'var(--color-primary)' ~/projects/[nome-do-projeto]/app/globals.css"
  },
  "tasks": [
    { "id": 1, "type": "backend", "task": "...", "file": "...", "instructions": "...", "done_when": "...", "rf": ["RF01"], "status": "pending" }
  ]
}
```

**Tipos válidos de tarefa** (`type`):
- Web: `setup` | `config` | `schema` | `backend` | `ui-setup` | `ui-component` | `ui-screen` | `integration` | `test` | `docs`
- Dados/R: `notebook` | `pipeline` | `viz` | `model` | `report` | `r-script` | `r-shiny`

O `type` determina qual pipeline de roles é executado. Detalhes: `agent/prompts/orchestrator.md`.
Se o projeto envolve dados/stats/ML: invoque também `agent/roles/data-scientist.md` durante o PLAN.

**Regras do PLAN:**
- Nunca replaneie depois. O `prd.json` é lei (só adicione tarefas, nunca delete)
- Toda tarefa deve ter campo `type` definido
- Se `has_ui: true`: tarefas `ui-setup` e `ui-component` ANTES de qualquer `ui-screen`
- Ao concluir: informe stack + nº de tarefas e inicie EXECUTE imediatamente

---

## Passo 2 — Fase EXECUTE + VERIFY (loop por tarefa)

Leia `agent/prompts/execute.md` e execute para cada tarefa pendente:

```
Para cada tarefa pendente no prd.json (em ordem):
  1.  Selecionar tarefa → status: "in_progress"
      → Ler task.type → selecionar pipeline (orchestrator.md)
  2.  Carregar contexto mínimo por tipo:
      → sempre: agent-brain.md + task.instructions
      → ui-*: + design-system.md | schema: + modelos existentes
  3.  [Dev] Implementar — sem TODOs, sem placeholders → SINAL: IMPL_READY
  4.  [QA] Gate técnico: meta.check_cmd
      → PASS: continue | FAIL ×3: ESCALADA Dev→Architect → marcar "blocked"
  5.  [QA+VV] Gate visual (apenas task.type = ui-* e has_ui: true)
      → visual_check_cmd + Checklist Visual Validator → SINAL: VV_PASS | VV_FAIL
  6.  [QA] Verificar done_when objetivamente
  7.  [QA] Lint (pula: setup, config, schema, ui-setup, test, docs)
  8.  [QA] Testes (apenas: backend crítico, integration, test)
  9.  [A cada 3 ui-* concluídas] Checkpoint PRD conformidade
 10.  Commit: bash agent/scripts/git_commit.sh "Task [id]: [nome]" "{app_dir}"
 11.  Marcar status: "completed" → SINAL: TASK_DONE
 12.  [Learner] LEARN rápido → silêncio se nada novo
      → Informar: ✓ Task [id]/[total]: [nome] (type:[type]) — [N] restantes
```

**Regras de ferro:**
- Uma tarefa por vez — nunca implemente duas simultaneamente
- `completed` é permanente — nunca altere
- Nova tarefa necessária? → ID maior + `rf` vinculado + `type` definido
- Escalação após 3 falhas → Escalation Matrix (orchestrator.md), nunca improvise
- Contexto saturando (> 40 trocas)? → Manager proativamente antes de degradar

---

## Passo 3 — Validação Final Integrada (antes de concluir)

Quando todas as tarefas estiverem concluídas, antes do LEARN GLOBAL:

**Validação Técnica:**
- Execute `meta.check_cmd` — deve passar
- Execute `meta.test_cmd` — todos os testes devem passar
- Execute Checklist de Segurança de `agent/roles/qa.md`

**Validação Visual** (apenas se `has_ui: true`):
- Invoque `agent/roles/visual-validator.md` — seção "Validação Final Integrada"
- Todas as variáveis CSS declaradas em `globals.css`?
- Todas as telas do PRD implementadas?
- Todas as telas passam no "teste do primeiro olhar"?
- Se falha: crie tarefas de correção e execute antes de concluir

---

## Passo 4 — LEARN Global (final do projeto)

Leia `agent/prompts/learn.md` — Ciclo Profundo.
Atualize `workspace/memory/agent-brain.md` com padrões, anti-padrões e aprendizados do projeto.

---

## Compatibilidade de Modelos

Funciona com qualquer modelo. Para modelos com contexto limitado: leia apenas "Hot Rules" do `agent-brain.md` (10 regras). Re-injete contexto a cada 5 tasks. Acione Manager após 25 trocas (não 40).

---

## Time de 9 Papéis

| Papel | Arquivo | Quando |
|-------|---------|--------|
| Analista | `agent/roles/analyst.md` | PLAN — PRD |
| Arquiteto | `agent/roles/architect.md` | PLAN — stack e estrutura |
| Data Scientist | `agent/roles/data-scientist.md` | PLAN (projetos de dados) + VERIFY análises |
| Designer | `agent/roles/designer.md` | PLAN — design-system.md |
| Desenvolvedor | `agent/roles/dev.md` | EXECUTE — implementação |
| QA | `agent/roles/qa.md` | VERIFY — gate técnico |
| Visual Validator | `agent/roles/visual-validator.md` | VERIFY — gate visual (UI) |
| Manager | `agent/roles/manager.md` | Contexto saturando |
| Learner | `agent/roles/learner.md` | LEARN — memória |

---

## Para retomar sessão anterior

Leia `agent/prompts/resume.md` e `workspace/memory/snapshots/latest.md`, depois continue do ponto onde parou.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronerog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
