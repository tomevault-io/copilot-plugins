## mother-v7-improvements

> > Este arquivo é lido automaticamente por todas as conversas Antigravity neste workspace.

# MOTHER — Contexto para IA

> Este arquivo é lido automaticamente por todas as conversas Antigravity neste workspace.
> **Revisão**: 2026-03-24 · 58 artefatos consolidados de 6 conversas

## O que é MOTHER

Sistema cognitivo autônomo auto-evolutivo criado por **Everton Garcia** (Wizards Down Under / Intelltech).

- **Objetivo A**: SHMS Brain — monitoramento geotécnico de barragens/minas (IoT → MQTT → TimescaleDB → LSTM → alertas)
- **Objetivo B**: Autonomia Total via Darwin Gödel Machine (arXiv:2505.22954)
- **Stack**: 500+ arquivos TypeScript, 9 camadas de qualidade, Multi-LLM, PostgreSQL + pgvector + TimescaleDB
- **Deploy**: Google Cloud Run (australia-southeast1)

## Documentos de contexto

Todos em `.agents/context/`. **Leia os relevantes antes de trabalhar.**

### 📐 Arquitetura & Pipeline (conversa principal)

| Arquivo | Conteúdo |
|---------|----------|
| `mother_architecture_real.md` | Arquitetura completa — módulos, camadas, diagramas mermaid |
| `mother_complete_interconnection_diagram.md` | Diagrama de interconexão entre todos os módulos |
| `mother_answer_pipeline_disclosure.md` | Pipeline de 35 estágios de processamento de queries |
| `mother_evaluation.md` | Métricas e avaliação de qualidade |
| `mother_visual_identity_sota.md` | Design system, oklch tokens, UX/UI guidelines |
| `mother_info_consolidation.md` | Consolidação de informações do sistema |
| `scientific_evaluation_mother_learning.md` | Avaliação científica do aprendizado |
| `mistral_ft_diagnostic_report.md` | Status do fine-tuning Mistral LoRA |
| `mother_shms_design_manifesto.md` | Manifesto de design SHMS — princípios e diretrizes |

### 🏗️ FOS/LEM & Geotecnia (conversa FOS debugging)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_fos_metodos_estabilidade_completo.md` | Métodos de estabilidade completos (Bishop, Janbu, Spencer, etc.) |
| `conv_fos_lem_architecture_research.md` | Pesquisa de arquitetura LEM |
| `conv_fos_lem_digital_twin_architecture.md` | Arquitetura Digital Twin + LEM |
| `conv_fos_ga_geotechnics_sota.md` | SOTA em geotecnia — Algoritmos Genéticos |
| `conv_fos_sota_research_shms.md` | Pesquisa SOTA — SHMS |
| `conv_fos_dam_3d_models_research.md` | Pesquisa modelos 3D de barragens |
| `conv_fos_digital_twins_research.md` | Pesquisa Digital Twins |
| `conv_fos_dgm_flowchart.md` | Fluxograma DGM completo |
| `conv_fos_design_manual.md` | Manual de design do sistema |
| `conv_fos_visual_identity_manual.md` | Manual de identidade visual |
| `conv_fos_sota_stability_ux_research.md` | Pesquisa UX para estabilidade |
| `conv_fos_sota_ux_bench_consolidation.md` | Consolidação benchmarks UX |
| `conv_fos_implementation_plan.md` | Plano de implementação FOS |
| `conv_fos_walkthrough.md` | Walkthrough das mudanças FOS |
| `lem_debug_knowledge.md` | Base de conhecimento de debugging LEM — erros, soluções, lições |
| `lem_scientific_reference.md` | Referência científica LEM — Bishop, Janbu, Spencer fórmulas |
| `conv_lem_mother_complete_study.md` | Estudo completo LEM em MOTHER — análise detalhada |
| `conv_lem_dt_commercial_guide.md` | Guia comercial LEM + Digital Twin — viabilidade e mercado |

### 📊 LEM Engine Optimization (conversa LEM Benchmarks)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_lem_gap_analysis.md` | Gap analysis completa — 21 funções, 39 módulos, root causes |
| `conv_system_gaps_improvements.md` | 12 gaps em 7 módulos — 4 tiers de prioridade + pipeline unificado |
| `conv_lem_100plus_benchmark_roadmap.md` | Roadmap 22→132+ benchmarks — 6 sprints, 3 fases |
| `conv_lem_22_22_walkthrough.md` | Walkthrough 19/22→22/22 — todas as correções documentadas |

### 📊 SHMS UI & Instrumentação (conversa SHMS UI)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_shms_ui_shms-data-instrumentation-research.md` | Pesquisa instrumentação SHMS |
| `conv_shms_ui_shms-ui-concepts.md` | Conceitos de UI SHMS |
| `conv_shms_ui_shms-ui-research.md` | Pesquisa UI SHMS |
| `conv_shms_ui_shms_evaluation.md` | Avaliação do SHMS |
| `conv_shms_ui_shms_ui_prompt.md` | Prompts de UI SHMS |
| `conv_shms_ui_implementation_plan.md` | Plano de implementação SHMS UI |
| `conv_shms_ui_walkthrough.md` | Walkthrough SHMS UI |
| `conv_shms_ui_sota_design_research.md` | Pesquisa SOTA de design para SHMS UI |
| `shms_instrumentation_catalog.md` | Catálogo completo de instrumentação SHMS — sensores, specs |
| `shms_safety_critical_design_guidelines.md` | Diretrizes de design safety-critical para SHMS |

### 🏗️ 3D Digital Twin & glTF (conversa 3D Metadata)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_gltf_metadata_digital_twin.md` | Metadados glTF, enriquecimento GLB, click-to-data, roadmap |

### 🧠 LSTM SOTA & Geotecnia (conversa LSTM Research)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_lstm_sota_update_plan.md` | 10 propostas SOTA completas: métricas, testes, benchmarks, roadmap 6-sprint, critérios aceitação, SOLID, clean code |
| `conv_lstm_sota_reference_content.md` | Conteúdo indexado das referências — equações, métodos, datasets, resultados quantitativos |
| `conv_lstm_mother_architecture_integration.md` | Integração LSTM v2 na arquitetura MOTHER — 7 sinergias, diagramas, fluxo end-to-end |

### 🎨 UX/UI & CAD Geotécnico (pesquisa cross-conversation)

| Arquivo | Conteúdo |
|---------|----------|
| `sota_ux_ui_cad_geotechnics_289.md` | Pesquisa SOTA massiva UX/UI + CAD geotécnico — 289 referências |
| `cad_architecture_integration.md` | Integração arquitetural CAD com MOTHER — pipeline e módulos |
| `conv_cad_canvas_sota.md` | SOTA CAD canvas — 160+ fontes, 12 categorias, seleção/grip/keyboard/dynamic input/boundaries |

### 🧠 Taxonomy & Routing SOTA (pesquisa Mar/2026)

| Arquivo | Conteúdo |
|---------|----------|
| `conv_sota_taxonomy_routing_research.md` | SOTA routing LLM — 12 papers (2024-2026), 7 gaps, gap analysis 71→91, roadmap 4 sprints |

## Regras

- Siga `/rules` workflow
- Use `/startup` para iniciar o dev server
- Use `/mother-pipeline` para referência do pipeline de 35 estágios
- **ZERO BULLSHIT**: não inventar, não chutar, não mentir
- **Leia os docs relevantes em `.agents/context/` antes de responder**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ehrvi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ehrvi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
