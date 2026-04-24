## info-gainme-dev

> graph.py           # KnowledgeGraph, Node, Edge

## Estrutura de pastas (projeto)
```
src/
  graph.py           # KnowledgeGraph, Node, Edge
  entropy.py         # Entropy
  data_types.py      # TurnState, PruningResult, ObservabilityMode
  orchestrator.py    # Orchestrator (single game)
  benchmark.py       # BenchmarkRunner (multi-game)
  benchmark_config.py # BenchmarkConfig
  agents/
    seeker.py        # SeekerAgent
    oracle.py         # OracleAgent
    pruner.py         # PrunerAgent
    llm_adapter.py    # LLMAdapter (OpenAI/vLLM-OpenAI) + history
  prompts/
    seeker_system.md  # System prompt do Seeker
    oracle_system.md  # System prompt do Oracle
    pruner_system.md  # System prompt do Pruner
  utils/
    utils.py          # parse_first_json_object
  domain/
    geo/
      loader.py       # load_geo_graph()
  analysis/
    data_types.py     # GameRun, CityStats, ExperimentResults
    loader.py         # load_experiment_results()
    writer.py         # save_summary(), save_city_variance()

benchmark_runner.py   # Script principal multi-game
demo_single_game.py   # Demo single game
scripts/
  analyze_results.py  # Análise de resultados e geração de JSONs
data/                 # entradas (CSV files)
outputs/              # artefatos (CSV, JSON, PNG, conversas)
logs/                 # logs de execução
```

## Diretrizes LLMAdapter
- Integrar diretamente com a lib OpenAI (API geral ou vLLM-OpenAI).
- Manter apenas estado `history` e utilitários (`append_history`, `reset_history`).
- Evitar criar métodos desnecessários: reutilizar as primitivas da API OpenAI.
- Cada agente possui sua própria instância stateful de `LLMAdapter`.

## Convenções de nomenclatura
- Usar snake_case para métodos, atributos e variáveis.
- Classes mantêm PascalCase (ex: `KnowledgeGraph`, `SeekerAgent`).
- Enums mantêm UPPER_CASE (ex: `FULLY_OBSERVED`).

## Fluxo de execução
1. **Single Game**: `demo_single_game.py` → `Orchestrator.from_target()` → `orchestrator.run()`
2. **Multi-Game**: `benchmark_runner.py` → `BenchmarkRunner` → múltiplos `Orchestrator` → CSV + conversas
3. **Analysis**: `scripts/analyze_results.py` → `load_experiment_results()` → `summary.json` + `variance.json`
4. **Data Flow**: `load_geo_graph()` → `KnowledgeGraph` → `Orchestrator` → results

## Agentes e responsabilidades
- **SeekerAgent**: Gera perguntas estratégicas (FO/PO modes)
- **OracleAgent**: Responde sobre target (JSON com game_over)
- **PrunerAgent**: Decide pruning baseado em Q&A (LLM-driven)
- **Orchestrator**: Coordena turnos e calcula entropia
- **BenchmarkRunner**: Executa múltiplos jogos e salva resultados

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/machadoDaniels) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
