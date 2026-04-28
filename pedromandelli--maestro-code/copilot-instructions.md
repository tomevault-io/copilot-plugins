## maestro-code

> description: Governa a criação, gerenciamento e ciclo de vida de tarefas individuais de desenvolvimento

---
description: Governa a criação, gerenciamento e ciclo de vida de tarefas individuais de desenvolvimento
globs:
alwaysApply: false
---
# Tasks Management

Whenever you use this rule, start your message with the following:

"Managing development tasks using Task Magic..."

## Propósito
Este arquivo define como criar, gerenciar e executar tarefas individuais de desenvolvimento dentro do sistema Task Magic.

## Estrutura de Arquivos

### Tarefas Ativas
- **Localização**: `.ai/tasks/task{id}_{nome}.md`
- **Nomenclatura**: 
  - `{id}`: Número sequencial (001, 002, etc.)
  - `{nome}`: Descrição concisa em kebab-case
  - Exemplo: `task001_setup-docker-environment.md`

### Master Checklist
- **Localização**: `.ai/TASKS.md`
- **Propósito**: Visão geral sincronizada de todas as tarefas ativas
- **Sincronização**: Deve ser atualizado sempre que tarefas são criadas/modificadas

## Template para Tarefas

```markdown
# Task {ID}: {Nome da Tarefa}

## Status: [PENDING|INPROGRESS|COMPLETED]
## Priority: [LOW|MEDIUM|HIGH|CRITICAL]
## Assigned: [Nome do desenvolvedor ou "Unassigned"]
## Created: YYYY-MM-DD
## Updated: YYYY-MM-DD

## Descrição
[Descrição clara e concisa do que precisa ser feito]

## Contexto
[Background/contexto necessário para entender a tarefa]

## Critérios de Aceitação
- [ ] Critério 1
- [ ] Critério 2
- [ ] Critério 3

## Dependências
- **Bloqueia**: [Lista de tarefas que dependem desta]
- **Bloqueada por**: [Lista de tarefas que devem ser concluídas primeiro]

## Notas Técnicas
[Considerações técnicas, APIs a usar, padrões a seguir, etc.]

## Definição de Pronto
- [ ] Código implementado
- [ ] Testes passando
- [ ] Code review aprovado
- [ ] Documentação atualizada
- [ ] Deploy realizado (se aplicável)

## Log de Progresso
### YYYY-MM-DD
- [Nota sobre progresso, decisões, bloqueios, etc.]
```

## Estados de Tarefa

### PENDING
- Tarefa criada mas não iniciada
- Todas as dependências foram mapeadas
- Pode estar aguardando outras tarefas

### INPROGRESS  
- Trabalho ativo sendo realizado
- Deve ter atualizações regulares no log de progresso
- Responsável definido

### COMPLETED
- Todos os critérios de aceitação atendidos
- Definição de pronto cumprida
- Pronta para arquivamento

## Diretrizes para Criação

### Tamanho de Tarefa
- **Ideal**: 1-3 dias de trabalho
- **Máximo**: 1 semana
- **Se maior**: Use `@.cursor/rules/.task-magic/expand.mdc` para quebrar em subtarefas

### Critérios de Qualidade
1. **Específica**: Objetivo claro e bem definido
2. **Mensurável**: Critérios de aceitação verificáveis
3. **Alcançável**: Realista dentro do contexto do projeto
4. **Relevante**: Conectada aos objetivos do projeto
5. **Temporal**: Com estimativa de esforço

### Boas Práticas
- Uma tarefa = uma responsabilidade
- Inclua links para planos relacionados
- Mantenha log de progresso atualizado
- Use labels/tags para categorização

## Sincronização com TASKS.md

Sempre que uma tarefa for criada, modificada ou movida, o arquivo `.ai/TASKS.md` deve ser atualizado para refletir:

```markdown
## Tarefas Ativas

### 🔴 High Priority
- [ ] **Task001**: Setup Docker Environment - PENDING
- [x] **Task002**: Create Landing Page - COMPLETED

### 🟡 Medium Priority  
- [ ] **Task003**: Implement Contact Form - INPROGRESS

### 🟢 Low Priority
- [ ] **Task004**: Add Analytics - PENDING
```

## Comandos de Gerenciamento

### Criar Nova Tarefa
1. Determine o próximo ID sequencial
2. Crie arquivo em `.ai/tasks/`
3. Atualize `.ai/TASKS.md`
4. Link para planos relacionados se aplicável

### Atualizar Status
1. Modifique status no arquivo da tarefa
2. Adicione entrada no log de progresso
3. Sincronize `.ai/TASKS.md`

### Arquivar Tarefa Concluída
1. Use `@.cursor/rules/.task-magic/memory.mdc`
2. Move arquivo para `.ai/memory/tasks/`
3. Atualiza `.ai/memory/TASKS_LOG.md`
4. Remove de `.ai/TASKS.md`

## Conexão com Outros Sistemas
- **Plans**: Tarefas devem referenciar planos relacionados
- **Memory**: Tarefas concluídas são arquivadas para contexto histórico
- **Expand**: Tarefas grandes devem ser quebradas em subtarefas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pedromandelli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
