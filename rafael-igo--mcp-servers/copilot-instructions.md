## mcp-servers

> Este projeto usa 8 MCPs (Model Context Protocol Servers) rodando via Docker para prover funcionalidades especializadas.

# Cursor Rules - MCPs Project

## Context
Este projeto usa 8 MCPs (Model Context Protocol Servers) rodando via Docker para prover funcionalidades especializadas.

## MCPs Disponíveis

**Sempre verifique se Docker está rodando primeiro:**
- Use `docker-admin.check_docker_status` antes de usar qualquer MCP
- Use `docker-admin.auto_heal` se houver problemas

### 1. excel-server
Leitura e processamento de Excel
- `read_excel_tabs(file_path)` - Lê todas as abas
- `read_excel_with_formulas(file_path, sheet_name?)` - Lê com fórmulas
- `get_excel_metadata(file_path)` - Metadados

### 2. agente-orchestrator
Orquestração de agentes especializados
- `list_agents()` - Lista agentes disponíveis
- `invoke_agent(agent_name, task)` - Invoca agente especializado
- `get_agent_docs(agent_name)` - Documentação do agente

**Agentes disponíveis:**
- Negócio: comercial-igo, diretoria-igo, marketing-igo, operacao-igo
- Técnicos: arquiteto-igo, backend, frontend-igo, qa-testes
- Módulos: transfer, rooming-list, checkin, rsvp, credenciamento, tracking, analytics-kpi

### 3. memory-manager
Gerenciamento de contexto e memória
- `load_context()` - Carrega contexto do projeto
- `save_context(module, status, details)` - Salva contexto
- `update_progress(task, status, notes?)` - Atualiza progresso
- `add_decision(decision, context, alternatives, chosen, reason)` - Registra ADR
- `get_next_steps()` - Próximos passos
- `get_memory_summary()` - Resumo completo

### 4. checklist-validator
Validação e gerenciamento de checklists
- `list_checklists()` - Lista todos os checklists
- `validate_checklist(checklist_path)` - Valida checklist
- `get_pending_tasks(checklist_path?)` - Tarefas pendentes
- `mark_completed(checklist_path, task_pattern)` - Marca como completa

### 5. agente-insights
Captura e análise de insights
- `capture_insight(idea, insight_type?, complexity?, modules?)` - Captura insight
- `get_insights(status?, insight_type?, limit?)` - Lista insights
- `update_insight_status(insight_id, new_status, notes?)` - Atualiza status
- `make_decision(insight_id, decision_status, rationale, priority?, effort_estimate?)` - Decide sobre insight
- `get_statistics()` - Estatísticas

### 6. agente-resumo
Status e resumos do projeto
- `get_project_status(include_details?)` - Status geral
- `get_module_status(module_name)` - Status de módulo
- `update_module_progress(module_name, progress, status?, notes?)` - Atualiza módulo
- `get_next_steps(limit?)` - Próximos passos
- `generate_report(report_type?, audience?)` - Gera relatório
- `get_metrics()` - Métricas

### 7. docker-admin
Administração de Docker e MCPs
- `check_docker_status()` - Verifica Docker
- `auto_heal()` - Auto-correção completa
- `health_check()` - Verifica saúde
- `manage_mcps(action)` - Gerencia todos (start/stop/restart/rebuild/status)
- `manage_mcp(name, action)` - Gerencia MCP específico
- `get_mcp_status()` - Status de containers
- `sync_mcp_config()` - Sincroniza .mcp.json

### 8. igo-openai-gateway
Gateway OpenAI com GPT-5.2 e reasoning
- `run_agent(agent_name, task, model?, reasoning_effort?, include_context?)` - Executa agente
- `run_development_agent(agent_name, task, reasoning_effort?, verbosity?)` - Agente de dev
- `run_code_analysis(code, analysis_type?, language?, reasoning_effort?)` - Análise de código
- `run_architectural_review(description, context?, reasoning_effort?)` - Revisão arquitetural
- `generate_tests(code, test_type?, framework?, reasoning_effort?)` - Gera testes
- `list_available_agents(category?)` - Lista agentes por categoria

## Workflow Obrigatório

### 1. INÍCIO DE SESSÃO
```
1. docker-admin.check_docker_status()
2. memory-manager.load_context()
3. agente-resumo.get_next_steps()
```

### 2. DURANTE DESENVOLVIMENTO
- **Sempre use agentes especializados** para tarefas específicas
- **Registre decisões** importantes: `memory-manager.add_decision()`
- **Atualize progresso**: `memory-manager.update_progress()`
- **Capture sugestões**: `agente-insights.capture_insight()`

### 3. FIM DE SESSÃO
```
1. memory-manager.save_context(module, status, details)
2. checklist-validator.mark_completed() para tarefas finalizadas
3. agente-resumo.add_next_step() para próximos passos
```

## Regras de Código

### Python
- Use type hints em todas as funções
- Docstrings em formato Google
- Testes com pytest
- Formatação: black + isort
- Linting: ruff

### TypeScript/JavaScript
- TypeScript estrito
- ESLint + Prettier
- Testes: Jest ou Vitest
- Componentes funcionais com hooks

### Docker
- Multi-stage builds
- Non-root user
- Health checks obrigatórios
- Logs para stdout/stderr

## Boas Práticas

### 1. Delegue para Especialistas
Não faça tudo sozinho. Use agentes:
- Arquitetura → `agente-orchestrator.invoke_agent("agente-arquiteto-igo", task)`
- Frontend → `agente-orchestrator.invoke_agent("agente-frontend-igo", task)`
- Backend → `agente-orchestrator.invoke_agent("agente-backend", task)`
- QA → `agente-orchestrator.invoke_agent("agente-qa-testes", task)`

### 2. Contexto é Rei
- Sempre carregue no início: `memory-manager.load_context()`
- Sempre salve no final: `memory-manager.save_context()`

### 3. Use Auto-Healing
Primeiro comando em caso de problemas:
```
docker-admin.auto_heal()
```

### 4. Análises Profundas
Para decisões complexas, use reasoning alto:
```
igo-openai-gateway.run_architectural_review(description, reasoning_effort="xhigh")
```

### 5. Documente Decisões
Toda decisão técnica importante deve ser registrada:
```
memory-manager.add_decision(
  decision="Escolha de framework X",
  context="Contexto da decisão",
  alternatives="Opções consideradas",
  chosen="Opção escolhida",
  reason="Razão da escolha"
)
```

## Troubleshooting

### MCPs não respondem
```
docker-admin.auto_heal()
```

### Container específico com problema
```
docker-admin.manage_mcp("nome-container", "restart")
```

### Ver logs
```
docker-admin.get_logs("nome-container", lines=100)
```

### Sincronizar config
```
docker-admin.sync_mcp_config()
```

## Comandos Úteis

### Status completo
```
docker-admin.health_check()
agente-resumo.get_project_status(include_details=true)
```

### Listar tudo pendente
```
checklist-validator.get_pending_tasks()
agente-resumo.get_next_steps()
```

### Gerar relatório executivo
```
agente-resumo.generate_report(report_type="executive", audience="management")
```

## IMPORTANTE

1. **Sempre** verifique Docker antes de usar MCPs
2. **Sempre** carregue contexto no início da sessão
3. **Sempre** salve contexto no final
4. **Sempre** use auto-heal primeiro em caso de problemas
5. **Sempre** delegue para agentes especializados
6. **Sempre** capture insights do usuário
7. **Sempre** documente decisões técnicas importantes

## Estrutura de Pastas
```
mcp-servers/
├── .mcp.json                 # Config MCP
├── .cursorrules              # Este arquivo
├── docker-compose.yml        # Containers
├── docs/
│   ├── agentes/              # Prompts agentes
│   └── memoria/              # Memória projeto
└── [8 MCPs em Docker]
```

Para mais detalhes, veja [GUIDELINES.md](GUIDELINES.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafael-igo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
