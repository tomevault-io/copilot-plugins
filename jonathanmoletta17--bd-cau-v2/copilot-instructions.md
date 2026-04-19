## bd-cau-v2

> - Simplicidade é fundamental

# Regras do Projeto BD_Cau_V2 - Para Cursor/Trae

## Filosofia do Projeto

### Less is More
- Simplicidade é fundamental
- Evitar over-engineering
- Código limpo > código "inteligente"

### Clean Slate Mindset
- Histórico Git recomeçado em 2025-12-06
- Decisões arquiteturais documentadas em ADR (Architecture Decision Records)
- Ver `MIGRATION_HISTORY.md` para contexto completo

### Conventional Commits Obrigatório
TODO commit DEVE seguir: `type(scope): message`
- Tipos: feat, fix, chore, docs, refactor, test, perf, style, ci
- Exemplo: `feat(agent): implement hybrid classification pipeline`
- Ver `CONTRIBUTING.md` para detalhes

---

## Estrutura do Projeto

### Monorepo com 7 Subprojetos
```
BD_Cau_V2/
├── glpi-data-service/             # Backend FastAPI + PostgreSQL
├── glpi-dtic-agent-classificator/ # Agente IA (Embeddings + LLM)
├── glpi-dtic-dashboard/           # Dashboard React DTIC (porta 3005)
├── glpi-dtic-smart-search/        # Busca inteligente DTIC (porta 3002)
├── glpi-sis-dashboard/            # Dashboard SIS (porta 3001)
├── glpi-sis-smart-search/         # Busca SIS (porta 3003)
└── glpi-sis-carregadores-dashboard/ # Carregadores (porta 3004)
```

### Não Referenciar (Projetos Órfãos)
❌ `glpi-data-service-v2` (substituído por v3)
❌ `rag-local` (removido)
❌ `crawl4ai_workspace` (removido)
❌ DOE-RS (projeto diferente)

---

## Stack Tecnológico

### Backend
- **Python 3.11+**: PEP 8, type hints, docstrings obrigatórias
- **FastAPI**: Async/await quando possível
- **PostgreSQL**: Schemas glpi, sis, dtic
- **SQLAlchemy**: ORM para queries complexas

### Frontend
- **React 18 + Vite**: Componentes funcionais
- **TypeScript**: Tipos explícitos sempre
- **Recharts**: Visualizações de dados
- **ESLint + Prettier**: Formatação automática

### IA/ML
- **Arquitetura Híbrida**: Embeddings (primário) + LLM (secundário)
- **Embeddings**: `intfloat/multilingual-e5-large`
- **LLM Local**: Llama 3 (via Ollama + RTX A4000)
- **LLM Cloud**: Gemini API (Google)

---

## Regras de Código

### Python
```python
# ✅ BOM
def classify_ticket(ticket_id: int, text: str) -> dict:
    """
    Classifica um ticket GLPI usando arquitetura híbrida.
    
    Args:
        ticket_id: ID do ticket no GLPI
        text: Texto descritivo do ticket
        
    Returns:
        dict com 'category', 'confidence', 'method'
    """
    # Implementação...
    pass

# ❌ RUIM
def classify(t, txt):  # Sem types, sem docstring
    return something
```

### TypeScript
```typescript
// ✅ BOM
interface TicketMetrics {
  total: number;
  em_progresso: number;
  fechados: number;
}

async function fetchMetrics(): Promise<TicketMetrics> {
  // Implementação...
}

// ❌ RUIM
async function fetchMetrics() {  // Sem tipo de retorno
  return await fetch(...)
}
```

---

## Proibições (NÃO FAZER!)

### Git
- ❌ **NÃO** criar novos repos `.git` em subpastas (monorepo!)
- ❌ **NÃO** fazer commits sem Conventional Commits
- ❌ **NÃO** commitar diretamente em `main` (usar PRs)
- ❌ **NÃO** fazer force push em branches compartilhadas

### Código
- ❌ **NÃO** commitar `.env` (usar `.env.example`)
- ❌ **NÃO** commitar `node_modules/`, `venv/`, `__pycache__/`
- ❌ **NÃO** adicionar deps sem atualizar `requirements.txt`/`package.json`
- ❌ **NÃO** usar `any` no TypeScript (ser específico)

### Arquitetura
- ❌ **NÃO** sugerir mudanças arquiteturais grandes sem criar ADR
- ❌ **NÃO** assumir que agente é "LLM puro" (é híbrido!)
- ❌ **NÃO** treinar em dados sujos (usar Golden Tickets)

---

## Padrões de Implementação

### Agente de Classificação
1. **Tentar Embeddings PRIMEIRO** (rápido, 80-90% dos casos)
2. **Se confiança < 0.75**: Usar LLM como "juiz"
3. **Se confiança < 0.50**: Marcar como "Requer Human Review"

### API Endpoints
- **Naming**: `/api/v1/{resource}/{action}`
- **Status Codes**: 200 (OK), 201 (Created), 400 (Bad Request), 404 (Not Found), 500 (Server Error)
- **Response Format**: Sempre JSON `{ "data": ..., "error": null }` ou `{ "data": null, "error": "..." }`

### Testes
- **Backend**: pytest, coverage mínimo 70%
- **Frontend**: Vitest para lógica, manual para UI
- **Agente**: Criar Golden Tickets para validação

---

## Documentação Obrigatória

### Quando Criar ADR (Architecture Decision Record)
- Mudanças arquiteturais significativas
- Trade-offs complexos que precisam ser justificados
- Decisões que afetam múltiplos subprojetos

**Local**: `docs/architecture/decisions/ADR-XXX_nome.md`

**Template**:
```markdown
# ADR-XXX: [Título da Decisão]

> **Status:** Proposto / Aceito / Depreciado
> **Data:** YYYY-MM-DD

## Contexto
[Problema ou situação]

## Decisão
[O que foi decidido]

## Consequências
[Benefícios e trade-offs]
```

### Quando Atualizar README.md
- Adicionar novo subprojeto
- Mudar comandos de setup
- Adicionar nova dependência crítica

---

## Fluxo de Trabalho (GitFlow Simplificado)

```
main       → DTICução (protegida, apenas via PR)
  ↓
develop    → Integração (branch padrão)
  ↓
feature/*  → Features (criar a partir de develop)
```

### Criando Feature
```bash
git checkout develop
git pull
git checkout -b feature/GLPI-123-nova-feature
# Desenvolver...
git commit -m "feat(scope): descrição"
git push origin feature/GLPI-123-nova-feature
# Criar PR para develop
```

---

## Recursos GPU (RTX A4000 - 16GB)

### Quando Usar GPU Local
- ✅ Embedding generation (batch de tickets)
- ✅ LLM inference (Llama 3 13B cabe)
- ✅ Fine-tuning de embeddings

### Quando NÃO Usar
- ❌ APIs simples (FastAPI)
- ❌ Queries PostgreSQL
- ❌ Frontend builds

### Setup
```bash
# Via Ollama (recomendado)
cd local-ai-stack
docker-compose up ollama

# Baixar modelo
ollama pull llama3:13b
```

---

## Contexto Importante

### Migração Clean Slate (2025-12-06)
- Histórico Git anterior **perdido** (0 repos recuperáveis)
- Baseline commit: `7d9773a`
- Documentação preservada em `docs/`
- Ver `MIGRATION_HISTORY.md` para detalhes completos

### Agente Híbrido (ADR-001)
- **NÃO** é chatbot
- **NÃO** usa apenas LLM
- **SIM** usa Embeddings como motor primário
- **SIM** usa LLM apenas como fallback

---

## Dicas para Trae

### Se Usuário Pedir "Analisar Código do Agente"
1. Ir para `glpi-dtic-agent-classificator/docs/specs/`
2. Ler `ADR-001_Rewrite_To_Hybrid_Architecture.md` PRIMEIRO
3. Entender arquitetura híbrida antes de sugerir mudanças

### Se Usuário Mencionar "v2"
ATENÇÃO: Pode ser projeto antigo! Perguntar se é:
- `glpi-data-service-v2` (antigo, não usar) OU
- `glpi-data-service-v3` (atual, usar este)

### Se Usuário Mencionar "RAG" ou "DOE-RS"
Estes são **projetos órfãos** (não existem mais aqui).  
Confirmar se usuário está em projeto diferente.

---

## Comandos Úteis

### Setup Rápido
```bash
# Backend
cd glpi-data-service && pip install -r requirements.txt

# Frontend
cd glpi-dtic-dashboard/frontend && npm install

# Agente
cd glpi-dtic-agent-classificator && pip install -r requirements.txt
```

### Testes
```bash
# Backend
pytest -v

# Frontend
npm test

# Linting
black . && flake8
```

---

**Lembre-se**: Este projeto tem filosofia "Less is More". Simplifique, não complexifique!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonathanmoletta17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
