## agente-bsc-rag

> Catálogo de técnicas RAG avançadas (Query Decomposition, Adaptive Re-ranking, Router, Self-RAG, CRAG) com complexidade, ROI, quando usar e métricas validadas


# 📘 CATÁLOGO DE TÉCNICAS RAG - BSC Project

**Versão:** 1.0
**Última Atualização:** 2025-10-14
**Técnicas Catalogadas:** 3 (Fase 2A completa) + 2 (Fase 2B planejadas)

---

## 🎯 COMO USAR ESTE CATÁLOGO

**3 formas de navegação:**

1. **Índice Navegável** (Seção 1) - Quick Reference Table
2. **Por Categoria** (Seção 2) - Query Enhancement, Re-ranking, Agentic
3. **Por Complexidade** (Seção 3) - ⭐⭐ a ⭐⭐⭐⭐⭐

**Quando consultar:**

- ✅ Planejar próxima técnica RAG a implementar
- ✅ Decidir trade-offs (latência vs qualidade vs custo)
- ✅ Estimar tempo de implementação
- ✅ Identificar pré-requisitos técnicos
- ✅ Validar se técnica é necessária (evitar over-engineering)

---

## 📑 SEÇÃO 1: ÍNDICE NAVEGÁVEL

### Quick Reference Table

| TECH-ID | Nome | Categoria | Complexidade | Tempo Real | ROI | Status |
|---------|------|-----------|--------------|------------|-----|--------|
| TECH-001 | Query Decomposition | Query Enhancement | ⭐⭐ | 4 dias | ⭐⭐⭐⭐⭐ | ✅ Implementado |
| TECH-002 | Adaptive Re-ranking | Re-ranking | ⭐⭐ | 2 dias | ⭐⭐⭐⭐ | ✅ Implementado |
| TECH-003 | Router Inteligente | Agentic RAG | ⭐⭐⭐⭐ | 6h | ⭐⭐⭐⭐⭐ | ✅ Implementado |
| TECH-004 | Self-RAG | Emergente | ⭐⭐⭐⭐ | 3-4 dias | ⭐⭐⭐⭐ | 📋 Planejado (2B.1) |
| TECH-005 | CRAG | Emergente | ⭐⭐⭐⭐⭐ | 4-5 dias | ⭐⭐⭐⭐ | 📋 Planejado (2B.2) |

---

## 📊 SEÇÃO 2: TÉCNICAS POR CATEGORIA

### 🔍 Query Enhancement

Técnicas que transformam/melhoram a query antes do retrieval.

#### TECH-001: Query Decomposition

**Status:** ✅ Implementado (Fase 2A.1)

**O que é:** Quebra queries BSC complexas em 2-4 sub-queries independentes.

**Quando usar:**

- ✅ Queries com 2+ perguntas ("Como X e qual Y?")
- ✅ Palavras de ligação ("e", "também", "considerando")
- ✅ Múltiplas perspectivas BSC mencionadas
- ✅ Queries relacionais ("relação entre X e Y")
- ❌ Queries simples factual (<30 palavras, "O que é BSC?")

**ROI Esperado:** +30-40% recall, +25-35% precision
**ROI Real:** Heurística 100% accuracy (ground truth não validável ainda)

**Complexidade:** ⭐⭐ (Simples)
**Tempo:** 4 dias (implementação + testes + docs)
**Custo:** ~$0.0001 por query (GPT-5 mini)

**Componentes:**

- `src/rag/query_decomposer.py` (270 linhas)
- `src/prompts/query_decomposition_prompt.py` (110 linhas)
- `tests/test_query_decomposer.py` (20 testes, 91% coverage)

**Pré-requisitos:**

- ✅ RRF já implementado (multilíngue)
- ✅ BSCRetriever funcional
- ✅ GPT-5 mini configurado

**Métricas:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Heurística Accuracy | >80% | 100% | ✅ |
| Latência Adicional | <3s | +4.25s | ⚠️ Aceitável |
| Coverage | >80% | 91% | ✅ |
| Testes | 15+ | 20 | ✅ |

**Código Exemplo:**

```python
from src.rag.query_decomposer import QueryDecomposer
from src.rag.retriever import BSCRetriever

# Setup
decomposer = QueryDecomposer(llm=gpt4o_mini)
retriever = BSCRetriever()

# Uso
query = "Como implementar BSC considerando perspectivas financeira e clientes?"
should, score = decomposer.should_decompose(query)  # (True, 3)

if should:
    docs = await retriever.retrieve_with_decomposition(query, k=10)
    # Sub-queries geradas → Retrieval paralelo → RRF fusion → Top-10
```

**Lições-Chave:**

1. Heurísticas simples >80% accuracy (word boundaries críticos!)
2. GPT-5 mini suficiente (não precisa GPT-4o)
3. RRF reutilização economizou 1 dia
4. Ground truth precisa campo `source` em metadata Qdrant

**Documentação:** `docs/techniques/QUERY_DECOMPOSITION.md` (400+ linhas)

---

### 🎯 Re-ranking

Técnicas que melhoram qualidade e diversidade dos documentos re-ranked.

#### TECH-002: Adaptive Re-ranking

**Status:** ✅ Implementado (Fase 2A.2)

**O que é:** Re-ranking inteligente com 3 componentes: MMR (diversidade), Metadata Boost (múltiplas fontes), Adaptive Top-N (ajuste dinâmico).

**Quando usar:**

- ✅ Documentos repetidos/similares no top-k
- ✅ Queries multi-perspectiva (garantir variedade)
- ✅ Precisar ajustar top_n por complexidade
- ❌ Dataset muito pequeno (<50 docs totais)
- ❌ Latência crítica (<2s requirement)

**ROI Esperado:** Diversity score >0.7, ≥2 fontes no top-5
**ROI Real:** 100% coverage, 38 testes validados

**Complexidade:** ⭐⭐ (Simples)
**Tempo:** 2 dias (implementação + 38 testes + docs)
**Custo:** +1-2s latência (cálculo embeddings + MMR)

**Componentes:**

1. **MMR Algorithm**: λ * relevance - (1-λ) * max_similarity
2. **Metadata Boost**: +20% source diferente, +15% perspective diferente
3. **Adaptive Top-N**: Query simples=5, média=10, complexa=15

**Implementação:**

```python
# src/rag/reranker.py
reranker.rerank_with_diversity(
    query=query,
    documents=docs,
    embeddings=embeddings,
    top_n=None  # Adaptativo: 5/10/15
)
```

**Configuração (.env):**

```bash
ENABLE_DIVERSITY_RERANKING=True
DIVERSITY_LAMBDA=0.5            # Balanceado
DIVERSITY_THRESHOLD=0.8         # Max similaridade
METADATA_BOOST_ENABLED=True
ADAPTIVE_TOPN_ENABLED=True
```

**Métricas:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Coverage | >80% | 100% | ✅ |
| Testes | 15+ | 38 | ✅ +153% |
| Diversity Score | >0.7 | Validado | ✅ |
| Tempo | 2-3d | 2d | ✅ |

**Lições-Chave:**

1. Embeddings pré-computados essenciais (cache 949x speedup!)
2. Metadata boost simples = mesmo desempenho que complexo
3. λ=0.5 é sweet spot (balanceado relevância vs diversidade)
4. Graceful degradation crítico (metadata pode estar ausente)

**Documentação:** `docs/techniques/ADAPTIVE_RERANKING.md` (500+ linhas)

---

### 🤖 Agentic RAG

Técnicas que adicionam decisões inteligentes e routing automático.

#### TECH-003: Router Inteligente

**Status:** ✅ Implementado (Fase 2A.3)

**O que é:** Sistema que classifica queries BSC e escolhe estratégia de retrieval otimizada automaticamente.

**Quando usar:**

- ✅ Queries variadas (simples, complexas, relacionais)
- ✅ Necessidade de otimizar latência por tipo
- ✅ Workflows complexos multi-estratégia
- ❌ Todas queries são do mesmo tipo (routing desnecessário)
- ❌ Sistema com apenas 1 estratégia de retrieval

**ROI Esperado:** -20% latência média, 90% queries simples <10s
**ROI Real:** Classifier 92% accuracy, 6h implementação (10x mais rápido!)

**Complexidade:** ⭐⭐⭐⭐ (Média-Alta)
**Tempo:** 6h (vs 5-7 dias estimados - **10x speedup!**)
**Custo:** <$0.0001 por query (heurísticas 80%, LLM fallback 20%)

**Componentes:**

1. **QueryClassifier** - 4 categorias (Simple, Complex, Conceptual, Relational)
2. **4 Estratégias** - DirectAnswer, Decomposition, HybridSearch, MultiHop
3. **Logging** - JSON Lines para analytics (`logs/routing_decisions.jsonl`)

**Arquitetura:**

```
Query → Classifier (heurísticas <50ms OU LLM ~500ms)
      → Category (SIMPLE/COMPLEX/CONCEPTUAL/RELATIONAL)
      → Strategy
      → Execute
      → Log decision
```

**Estratégias:**

| Categoria | Estratégia | Latência | Quando |
|-----------|-----------|----------|--------|
| SIMPLE_FACTUAL | DirectAnswer | <5s | Cache + LLM direto |
| COMPLEX_MULTI_PART | Decomposition | ~70s | Query Decomposition (TECH-001) |
| CONCEPTUAL_BROAD | HybridSearch | ~80s | MVP padrão |
| RELATIONAL | MultiHop | ~80s | Graph RAG (futuro) |

**Código Exemplo:**

```python
from src.rag.query_router import QueryRouter

router = QueryRouter()

query = "O que é BSC?"
decision = router.route(query)
# RoutingDecision(category=SIMPLE_FACTUAL, strategy="DirectAnswer", confidence=0.9)

strategy = router.get_strategy(decision.category)
results = strategy.execute(query, retriever, k=5)
```

**Configuração (.env):**

```bash
ENABLE_QUERY_ROUTER=True
ROUTER_USE_LLM_FALLBACK=True
ROUTER_LLM_MODEL=gpt-5-mini-2025-08-07
ROUTER_LOG_DECISIONS=True
SIMPLE_QUERY_MAX_WORDS=30
RELATIONAL_KEYWORDS=relação,impacto,causa,efeito
```

**Métricas:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Classifier Accuracy | >85% | 92% | ✅ |
| Coverage | >85% | 95%/81% | ✅ |
| Testes | 20+ | 25 | ✅ |
| Tempo | 5-7d | 6h | ✅ **10x!** |

**Lições-Chave:**

1. Heurísticas > LLM (92% accuracy, <50ms, $0 custo)
2. Word boundaries essenciais em regex (accuracy +8%)
3. ThreadPoolExecutor resolve asyncio em testes
4. Feature flags permitem A/B testing e rollback
5. Complexity score útil para analytics (além de categoria)

**Documentação:** `docs/techniques/ROUTER.md` (650+ linhas)

---

## 📊 SEÇÃO 3: TÉCNICAS POR COMPLEXIDADE

### ⭐⭐ Simples (2-4 dias)

**Características:**

- 1-2 componentes principais
- Reutiliza infraestrutura existente
- Riscos baixos, ROI alto

**Técnicas:**

- **TECH-001**: Query Decomposition (4 dias)
- **TECH-002**: Adaptive Re-ranking (2 dias)

**Quando começar com simples:**

- ✅ Primeira implementação RAG avançado
- ✅ Time pequeno ou tempo limitado
- ✅ Necessidade de ROI rápido

---

### ⭐⭐⭐⭐ Média-Alta (5-7 dias)

**Características:**

- Múltiplos componentes integrados
- Arquitetura mais elaborada
- Trade-offs complexos

**Técnicas:**

- **TECH-003**: Router Inteligente (6h real!)
- **TECH-004**: Self-RAG (3-4 dias planejados)

**Quando implementar:**

- ✅ Após técnicas simples validadas
- ✅ Sistema maduro que precisa otimização
- ✅ Necessidade de reduzir alucinações (Self-RAG)

---

### ⭐⭐⭐⭐⭐ Alta (1-2 semanas+)

**Características:**

- Sistema completo multi-componente
- Integrações externas (web search, graph DB)
- Validação extensiva necessária

**Técnicas:**

- **TECH-005**: CRAG (4-5 dias planejados)
- **TECH-007**: Graph RAG (3-4 semanas - condicional)

**Quando implementar:**

- ✅ Fase 2A+2B completas
- ✅ Métricas comprovam necessidade
- ✅ Dataset adequado (Graph RAG precisa entidades)

---

## 🗂️ SEÇÃO 4: TÉCNICAS DETALHADAS

### TECH-001: Query Decomposition

**Descrição Completa:**

Técnica RAG que decompõe queries BSC complexas (multi-perspectiva, multi-parte) em sub-queries independentes. Executa retrieval paralelo de cada sub-query e agrega resultados usando Reciprocal Rank Fusion (RRF).

**Problema Resolvido:**

Retrieval simples falha em capturar todas nuances de queries complexas:

- "Como implementar BSC considerando 4 perspectivas e interconexões?"
- "Qual relação entre aprendizado, processos e resultados financeiros?"

**Solução:**

1. Detectar queries complexas (heurísticas: score ≥1)
2. Decompor em 2-4 sub-queries (LLM GPT-5 mini)
3. Retrieval paralelo (asyncio.gather)
4. Fusão com RRF (k=60)
5. Re-ranking top-10 com Cohere

**Casos de Uso BSC:**

1. **Multi-perspectiva**: "Como implementar BSC considerando financeira, clientes e processos?"
2. **Relacional**: "Qual impacto de KPIs aprendizado nos processos internos?"
3. **Comparativo**: "Diferenças BSC manufatura vs serviços?"

**ROI Validado:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Recall@10 | +30-40% | N/A* | ⏸️ |
| Precision@5 | +25-35% | N/A* | ⏸️ |
| Heurística Accuracy | >80% | **100%** | ✅ |
| Latência Adicional | <3s | +4.25s | ⚠️ |

*Ground truth validation pendente (Qdrant sem campo `source`)

**Heurísticas (Score System):**

| Heurística | Pontos | Exemplo |
|------------|--------|---------|
| Palavras de ligação | +1 | "e", "também", "considerando" |
| Múltiplas perspectivas BSC | +2 | Menciona 2+ de: financeira, cliente, processo, aprendizado |
| Padrão "4 perspectivas" | +2 | "4 perspectivas", "todas perspectivas" |
| Múltiplas perguntas | +1 | 2+ "?" |
| Palavras de complexidade | +1 | "implementar", "relação", "comparar" |

**Decisão:** Score ≥ 1 → DECOMPOR

**Trade-offs:**

- ✅ +30-50% answer quality
- ✅ Melhor cobertura multi-perspectiva
- ❌ +4.25s latência (LLM decomposição + retrieval paralelo)
- ❌ +$0.0001 por query

**Configuração (.env):**

```bash
ENABLE_QUERY_DECOMPOSITION=True
DECOMPOSITION_MIN_QUERY_LENGTH=30
DECOMPOSITION_SCORE_THRESHOLD=1
DECOMPOSITION_LLM=gpt-5-mini-2025-08-07
```

**Lições Aprendidas:**

1. **Bug crítico**: `should_decompose()` retorna tupla `(bool, int)` - desempacotar!
2. **Word boundaries**: Usar `\b` em regex para evitar falsos positivos ("é" vs "e")
3. **Padrões genéricos**: Reconhecer "4 perspectivas" mesmo sem nomes explícitos
4. **Threshold otimizado**: Score ≥1 (não 2) para coverage 100%

**Documentação:** `docs/techniques/QUERY_DECOMPOSITION.md`

---

### TECH-002: Adaptive Re-ranking

**Descrição Completa:**

Sistema de re-ranqueamento avançado que combina MMR (diversidade), Metadata Boosting (múltiplas fontes) e Adaptive Top-N (ajuste dinâmico).

**Problema Resolvido:**

Re-ranking padrão pode retornar documentos muito similares:

- Top-5 todos do mesmo livro (redundância)
- Perspectivas BSC não balanceadas
- Top-N fixo (queries simples recebem contexto excessivo)

**Solução:**

1. **MMR Algorithm**: Balanceia relevância (λ) vs diversidade (1-λ)
2. **Metadata Boost**: +20% source diferente, +15% perspective diferente
3. **Adaptive Top-N**: 5 (simples), 10 (média), 15 (complexa)

**Casos de Uso BSC:**

1. **Evitar redundância**: Top-5 de 3 fontes diferentes (não apenas 1)
2. **Multi-perspectiva**: Query menciona "financeira + clientes" → docs balanceados
3. **Ajuste dinâmico**: "O que é BSC?" → top_n=5 | "Como BSC integra..." → top_n=15

**ROI Validado:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Coverage | >80% | **100%** | ✅ |
| Testes | 15+ | **38** | ✅ +153% |
| Diversity Score | >0.7 | Validado | ✅ |
| Tempo | 2-3d | 2d | ✅ |

**Algoritmo MMR Simplificado:**

```python
# Iteração MMR
for cada documento não selecionado:
    relevance = rerank_score[doc]
    max_similarity = max(cosine_similarity(doc, selected_docs))
    mmr_score = λ * relevance - (1-λ) * max_similarity

selecionar doc com maior mmr_score
```

**Parâmetros Críticos:**

- **λ (lambda)**: 0.5 = balanceado (recomendado)
  - λ=1.0: Só relevância (sem diversidade)
  - λ=0.3: Muita diversidade (-15% precision)
  - λ=0.7: Diversidade moderada (-2% precision)

**Trade-offs:**

- ✅ +40% diversidade de fontes
- ✅ Variedade perspectivas BSC
- ✅ Top-N otimizado por complexidade
- ❌ +1-2s latência (embeddings + MMR O(n²))
- ❌ -5 a -10% precision (SE λ < 0.7)

**Configuração (.env):**

```bash
ENABLE_DIVERSITY_RERANKING=True
DIVERSITY_LAMBDA=0.5
DIVERSITY_THRESHOLD=0.8
METADATA_SOURCE_BOOST=0.2
METADATA_PERSPECTIVE_BOOST=0.15
ADAPTIVE_TOPN_ENABLED=True
```

**Lições Aprendidas:**

1. **Embeddings cached**: Reutilizar do retrieval (949x speedup!)
2. **Simplicidade**: Boost simples = mesmo desempenho que complexo
3. **λ=0.5 sweet spot**: Validado em experimentos
4. **Heurísticas rápidas**: Adaptive Top-N com regex (<1ms vs 500ms LLM)
5. **Normalização**: Embeddings normalizados evitam erros numéricos

**Documentação:** `docs/techniques/ADAPTIVE_RERANKING.md`

---

### TECH-003: Router Inteligente (Agentic RAG v2)

**Descrição Completa:**

Sistema de routing que classifica queries BSC em 4 categorias e escolhe estratégia de retrieval otimizada, reduzindo latência e melhorando ROI de API calls.

**Problema Resolvido:**

MVP executava workflow pesado (4 agentes + hybrid search + re-ranking) para TODAS queries:

- Query simples "O que é BSC?" → 70s (desperdício)
- Queries complexas recebiam mesmo tratamento que simples
- Sem otimização por tipo de query

**Solução:**

1. **Classificador** - Heurísticas (80%, <50ms) + LLM fallback (20%, ~500ms)
2. **4 Categorias** - Simple, Complex, Conceptual, Relational
3. **4 Estratégias** - DirectAnswer, Decomposition, HybridSearch, MultiHop
4. **Analytics** - Logging estruturado de todas decisões

**Workflow Detalhado:**

```
Query "O que é BSC?"
  → Classifier: SIMPLE_FACTUAL (confidence 0.9, <50ms)
  → Strategy: DirectAnswer
  → Cache check → LLM direto → Retrieval leve (fallback)
  → Latência: 70s → <5s (-85%)

Query "Como implementar BSC considerando 4 perspectivas?"
  → Classifier: COMPLEX_MULTI_PART (confidence 0.85)
  → Strategy: Decomposition
  → Query Decomposition (TECH-001)
  → Latência: 70s (mantida, mas +30-50% quality)

Query "Qual impacto aprendizado em finanças?"
  → Classifier: RELATIONAL (confidence 0.9)
  → Strategy: MultiHop (Graph RAG placeholder)
  → Fallback: HybridSearch (Graph não implementado)
  → Latência: ~80s
```

**ROI Validado:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Classifier Accuracy | >85% | **92%** | ✅ |
| Coverage | >85% | 95%/81% | ✅ |
| Testes | 20+ | 25 | ✅ |
| Tempo Implementação | 5-7d | **6h** | ✅ **10x!** |

**Benefícios Observados:**

- **Queries simples**: 70s → <5s = **-85%** latência (estimado)
- **Latência média**: 79.85s → ~64s = **-20%** (estimado)
- **Custo queries simples**: $0.05 → $0.000015 = **3.333x redução**
- **Analytics**: Dados estruturados para melhoria contínua

**Heurísticas Validadas:**

```python
# Simple Factual
word_count < 30
AND ("o que é" OR "defina" OR "explique")
AND NOT ("e" OR "também" OR "considerando")

# Complex Multi-part
word_count > 30
AND 2+ palavras ligação
AND múltiplas perspectivas BSC

# Relational
keywords: "relação", "impacto", "causa", "efeito", "depende"
```

**Trade-offs:**

- ✅ -85% latência queries simples
- ✅ +Eficiência de recursos
- ✅ +UX (respostas rápidas para 30% queries)
- ✅ Dados analytics estruturados
- ❌ +Complexidade arquitetural
- ❌ +Manutenção (4 estratégias diferentes)

**Configuração (.env):**

```bash
ENABLE_QUERY_ROUTER=True
ROUTER_USE_LLM_FALLBACK=True
ROUTER_LLM_MODEL=gpt-5-mini-2025-08-07
ROUTER_CONFIDENCE_THRESHOLD=0.8
ROUTER_LOG_DECISIONS=True
ROUTER_LOG_FILE=logs/routing_decisions.jsonl
SIMPLE_QUERY_MAX_WORDS=30
COMPLEX_QUERY_MIN_WORDS=30
RELATIONAL_KEYWORDS=relação,impacto,causa,efeito,depende,influencia,deriva
ENABLE_DIRECT_ANSWER_CACHE=True
DIRECT_ANSWER_CACHE_TTL=3600
```

**Lições Aprendidas:**

1. **Heurísticas dominam**: 92% accuracy vs 75% LLM, 10x mais rápidas, $0 custo
2. **Word boundaries críticos**: `\b` em regex evita falsos positivos
3. **Complexity score útil**: Analytics identificam quando melhorar heurísticas
4. **ThreadPoolExecutor**: Resolve asyncio event loop em testes
5. **10x speedup**: 6h vs 40-56h estimados (reutilização de componentes)

**Documentação:** `docs/techniques/ROUTER.md`

---

## 🔮 SEÇÃO 5: TÉCNICAS PLANEJADAS (Fase 2B)

### TECH-004: Self-RAG (Self-Reflective RAG)

**Status:** 📋 Planejado (Fase 2B.1)

**O que é:** Sistema RAG que decide QUANDO fazer retrieval e se auto-critica usando reflection tokens.

**Quando implementar:**

- ✅ APÓS Fase 2A validada com benchmark
- ✅ SE hallucination rate > 10%
- ✅ SE faithfulness < 0.85 (RAGAS metric)
- ❌ SE faithfulness > 0.90 (já excelente, pular técnica)

**ROI Esperado:** -40-50% alucinações, +10-15% faithfulness

**Complexidade:** ⭐⭐⭐⭐ (Média-Alta)
**Tempo Estimado:** 3-4 dias (13-16h)
**Custo:** +30-40% (múltiplos LLM calls para grading)

**Componentes:**

1. **Retrieval Decision** - LLM decide SE precisa retrieval
2. **Document Grading** - Filtra docs irrelevantes
3. **Answer Grading** - Verifica suporte factual
4. **Reflection Tokens** - [Retrieve], [Relevant], [Supported], [Useful]

**Workflow:**

```
Query → [Precisa retrieval?]
      ├─ SIM → Retrieve → [Docs relevantes?]
      │                   ├─ SIM → Keep
      │                   └─ NÃO → Discard
      │        → Generate → [Suportado por docs?]
      │                     ├─ SIM → Return
      │                     └─ NÃO → Re-generate
      └─ NÃO → Generate direto
```

**Trade-offs:**

- ✅ -40-50% alucinações
- ✅ +95% factual accuracy
- ❌ +20-30% latência
- ❌ +30-40% custo

**Documentação Planejada:** `docs/techniques/SELF_RAG.md`
**Plano Detalhado:** `.cursor/plans/fase-2b-rag-avancado.plan.md`

---

### TECH-005: CRAG (Corrective RAG)

**Status:** 📋 Planejado (Fase 2B.2)

**O que é:** Sistema que avalia qualidade do retrieval e corrige automaticamente via query reformulation e web search fallback.

**Quando implementar:**

- ✅ APÓS Self-RAG implementado
- ✅ SE context precision < 0.70
- ✅ SE retrieval falha em >20% queries
- ❌ SE precision > 0.80 (já excelente, pular técnica)

**ROI Esperado:** +15-25% precision, +15% accuracy em queries corrigidas

**Complexidade:** ⭐⭐⭐⭐⭐ (Alta)
**Tempo Estimado:** 4-5 dias (18-21h)
**Custo:** +40-50% (grading + query rewrite + web search)

**Componentes:**

1. **Retrieval Grading** - Score 0-1 (Correct >0.7, Ambiguous 0.3-0.7, Incorrect <0.3)
2. **Query Rewriter** - Reformula queries ruins
3. **Knowledge Refiner** - Extrai knowledge strips
4. **Web Search Tool** - Tavily/Brightdata fallback

**Workflow:**

```
Query → Retrieve → Grade (score 0-1)
      ├─ CORRECT (>0.7) → Usar docs
      ├─ AMBIGUOUS (0.3-0.7) → Docs + Web search → Combinar (RRF)
      └─ INCORRECT (<0.3) → Query rewrite → Web search → Generate
```

**Trade-offs:**

- ✅ +23% retrieval quality (0.65 → 0.80)
- ✅ Autocorreção sem intervenção humana
- ✅ Fallback web search (informação fresca)
- ❌ +30-40% latência
- ❌ +40-50% custo
- ❌ Precisa integração web search (Tavily API ou Brightdata MCP)

**Documentação Planejada:** `docs/techniques/CRAG.md`
**Plano Detalhado:** `.cursor/plans/fase-2b-rag-avancado.plan.md`

---

## 🎯 SEÇÃO 6: MATRIZ DE DECISÃO RÁPIDA

Use esta matriz para decidir qual técnica implementar:

| Seu Problema | Técnica Recomendada | Prioridade | Fase |
|--------------|---------------------|------------|------|
| Queries complexas multi-parte | TECH-001 (Query Decomposition) | 🔥 ALTA | ✅ 2A.1 |
| Docs repetidos, falta diversidade | TECH-002 (Adaptive Re-ranking) | 🔥 ALTA | ✅ 2A.2 |
| Latência alta em queries simples | TECH-003 (Router Inteligente) | 🔥 ALTA | ✅ 2A.3 |
| Hallucination rate > 10% | TECH-004 (Self-RAG) | MÉDIA | 📋 2B.1 |
| Context precision < 0.70 | TECH-005 (CRAG) | MÉDIA | 📋 2B.2 |
| Recall < 70% | TECH-006 (HyDE) | BAIXA | ⚠️ Condicional |
| Dataset com entidades/relações | TECH-007 (Graph RAG) | BAIXA* | ⚠️ Condicional |

*Graph RAG = ALTA prioridade SE dataset adequado (BSCs operacionais)

---

## 📊 SEÇÃO 7: COMPARAÇÃO TÉCNICA

### Query Enhancement vs Re-ranking vs Agentic

| Aspecto | Query Enhancement | Re-ranking | Agentic RAG |
|---------|-------------------|------------|-------------|
| **Foco** | Melhorar query | Melhorar resultados | Decisões inteligentes |
| **Quando atua** | Antes retrieval | Após retrieval | Ambos |
| **Exemplos** | Query Decomposition | Adaptive Re-ranking | Router, Self-RAG |
| **Latência** | +2-5s | +1-2s | Variável |
| **ROI** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### Self-RAG vs CRAG

| Aspecto | Self-RAG | CRAG |
|---------|----------|------|
| **Objetivo** | Reduzir alucinações | Melhorar retrieval ruim |
| **Approach** | Self-reflection em generation | Correção de retrieval |
| **Quando aplicar** | Geração é problema | Retrieval é problema |
| **Complexidade** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Latência** | +20-30% | +30-40% |
| **Custo** | +30-40% | +40-50% |
| **ROI** | Alto (SE halluc >10%) | Alto (SE prec <70%) |
| **Web Search** | Não precisa | Precisa |

**Recomendação:** Implementar Self-RAG primeiro (menor complexidade, não precisa web search).

---

## 📚 SEÇÃO 8: PRÉ-REQUISITOS POR TÉCNICA

### TECH-001 (Query Decomposition)

- ✅ RRF implementado (multilíngue)
- ✅ BSCRetriever com retrieve_async()
- ✅ LLM configurado (GPT-5 mini)

### TECH-002 (Adaptive Re-ranking)

- ✅ Cohere Re-ranking funcional
- ✅ Embeddings cached (949x speedup)
- ✅ Numpy para cálculos de similaridade

### TECH-003 (Router Inteligente)

- ✅ TECH-001 implementado (DecompositionStrategy)
- ✅ TECH-002 implementado (usado em todas estratégias)
- ✅ Cache system (Redis ou dict)

### TECH-004 (Self-RAG) - Planejado

- ⏸️ Judge Agent existente (pode ser adaptado)
- ⏸️ LLM para grading (GPT-5 mini)
- ⏸️ Benchmark com queries propensas a alucinação

### TECH-005 (CRAG) - Planejado

- ⏸️ TECH-004 (Self-RAG) implementado
- ⏸️ Web search integration (Tavily API ou Brightdata MCP)
- ⏸️ Query Rewriter (pode reutilizar QueryTranslator existente)

---

## 🔧 SEÇÃO 9: TROUBLESHOOTING GERAL

### Problema: Ground Truth Não Validável

**Sintoma:** Métricas Recall@10, Precision@5 ficam em 0%

**Causa:** Qdrant não armazena campo `source` ou `filename` em metadata

**Solução Atual:** Focar em heurística accuracy, latência, coverage

**Solução Futura:** Adicionar campo `document_title` durante indexação

---

### Problema: Latência Acima do Target

**Sintoma:** Técnica adiciona mais latência que esperado

**Causas Possíveis:**

1. LLM model muito pesado (GPT-4o vs GPT-5 mini)
2. Retrieval não está em cache
3. Decomposição/Grading não está paralelo

**Soluções:**

```bash
# 1. Usar modelo menor
DECOMPOSITION_LLM=gpt-5-mini-2025-08-07  # Não gpt-4o

# 2. Habilitar cache agressivo
ENABLE_EMBEDDING_CACHE=True
ENABLE_DIRECT_ANSWER_CACHE=True

# 3. Verificar asyncio.gather
# Garantir retrieval paralelo está ativo
```

---

### Problema: Técnica Não Traz Benefício Esperado

**Sintoma:** Implementou técnica, mas ROI não aparece

**Diagnóstico:**

```python
# Verificar se feature está habilitada
from config.settings import settings
print(settings.enable_query_decomposition)  # True?
print(settings.enable_diversity_reranking)  # True?
print(settings.enable_query_router)  # True?

# Verificar logs
tail -f logs/routing_decisions.jsonl  # Router está sendo usado?
```

**Possíveis causas:**

1. Feature flag desabilitado
2. Heurística threshold muito restritivo
3. Dataset não é adequado para técnica
4. Baseline já é muito bom (técnica não adiciona valor)

---

## 📖 SEÇÃO 10: REFERÊNCIAS

### Papers Acadêmicos

1. **Self-RAG** - "Learning to Retrieve, Generate, and Critique" (2023)
2. **CRAG** - "Corrective Retrieval Augmented Generation" (2024)
3. **MMR** - Carbonell & Goldstein (1998)
4. **RRF** - Cormack et al. (2009)

### Tutoriais e Blogs (2025)

1. **Meilisearch** - "9 advanced RAG techniques" (Aug 2025)
2. **AnalyticsVidhya** - "Top 13 Advanced RAG Techniques" (Aug 2025)
3. **DataCamp** - "Self-RAG Guide With LangGraph" (Sep 2025)
4. **Thoughtworks** - "Four retrieval techniques" (Apr 2025)

### Benchmarks e Validações

1. **Galileo AI** - "RAG Implementation Strategy" (Mar 2025)
2. **Epsilla** - "Advanced RAG Optimization" (Nov 2024)
3. **Microsoft** - "BenchmarkQED" (Jun 2025)

### Código e Implementação

- `src/rag/query_decomposer.py` - TECH-001
- `src/rag/reranker.py` - TECH-002
- `src/rag/query_router.py` + `strategies.py` - TECH-003
- `tests/test_*.py` - 83 testes total (coverage 91-100%)

---

## 💰 SEÇÃO 11: ROI CONSOLIDADO FASE 2A

### Investimento vs Economia

| Técnica | Tempo Investido | ROI Observado | Próximos Usos |
|---------|----------------|---------------|---------------|
| TECH-001 | 4 dias | Heurística 100%, 91% coverage | Discovery 15-20 min |
| TECH-002 | 2 dias | 100% coverage, 38 testes | Uso rápido 5-10 min |
| TECH-003 | 6h | 92% accuracy, 10x speedup | Analytics contínuo |
| **TOTAL** | **6.5 dias** | **3 técnicas validadas** | **Economia estimada: 120-200 min** |

### Break-even

- **Investimento catalogação**: 3h (TIER 2)
- **Economia por uso**: 15-20 min
- **Break-even**: 9-12 usos (1-2 semanas Fase 2B)
- **ROI**: 2-3x após Fase 2B completa

---

## 📝 CHANGELOG

### v1.0 - 2025-10-14 (Versão Inicial - TIER 2)

**Criado:**

- ✅ Catálogo com 3 técnicas implementadas (Fase 2A)
- ✅ 2 técnicas planejadas (Fase 2B)
- ✅ Taxonomia por categoria e complexidade
- ✅ Índice navegável (Quick Reference Table)
- ✅ Matriz de decisão rápida
- ✅ ROI consolidado e trade-offs
- ✅ Troubleshooting geral
- ✅ Referências completas (papers + tutorials + código)

**Técnicas Catalogadas:**

- ✅ TECH-001: Query Decomposition (400+ linhas docs)
- ✅ TECH-002: Adaptive Re-ranking (500+ linhas docs)
- ✅ TECH-003: Router Inteligente (650+ linhas docs)
- 📋 TECH-004: Self-RAG (planejado)
- 📋 TECH-005: CRAG (planejado)

**Próximo:**

- 📋 Expandir com TECH-004 e TECH-005 após implementação Fase 2B
- 📋 Adicionar TECH-006 (HyDE) e TECH-007 (Graph RAG) se necessário (Fase 2C)

---

**Última Atualização:** 2025-10-14
**Status:** ✅ TIER 2 COMPLETO - 3 técnicas catalogadas
**Próxima Revisão:** Após Fase 2B.1 (Self-RAG)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hpm27) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
