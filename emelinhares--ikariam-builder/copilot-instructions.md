## ikariam-builder

> Este arquivo é o **entrypoint de contexto** para agentes que vão programar no projeto.

# AGENTS.md — Índice único para agentes de IA

Este arquivo é o **entrypoint de contexto** para agentes que vão programar no projeto.
Objetivo: reduzir leitura desnecessária e guiar rapidamente para os arquivos corretos.

---

## 1) Resumo rápido do projeto

- Nome: `ikariam-builder`
- Tipo: extensão Chrome MV3 + automação/scraper do Ikariam
- Linguagem principal: JavaScript ESM
- Testes: Vitest (unitários em `tests/unit`)

Arquivos-base:
- [`package.json`](package.json)
- [`manifest.json`](manifest.json)
- [`README.md`](README.md)

---

## 2) Estratégia de leitura recomendada (token-efficient)

1. Ler este arquivo [`AGENTS.md`](AGENTS.md).
2. Ir direto para o documento-alvo conforme a tarefa (seção 3).
3. Ler apenas os módulos impactados (seções 4 e 5).
4. Validar com testes focados em [`tests/unit`](tests/unit).

Evitar leitura completa de toda a documentação quando a tarefa for local.

---

## 3) Roteador de documentação por tipo de tarefa

- Regra de negócio / priorização / ROI / papéis (CFO/COO/HR/CTO/CSO/MnA):
  - [`BUSINESS.md`](BUSINESS.md)
- Implementação técnica, camadas, wiring e eventos:
  - [`ARCHITECTURE.md`](ARCHITECTURE.md)
- Contratos HTTP, parâmetros e ações do jogo:
  - [`ENDPOINTS.md`](ENDPOINTS.md)
- Modelo do jogo (`ikariam.*`, `bgViewData`):
  - [`GAME_MODEL.md`](GAME_MODEL.md)
- UI/painel e contratos de estado para interface:
  - [`UI.md`](UI.md)
- Coleta via Playwright/scraper:
  - [`SCRAPER.md`](SCRAPER.md)

---

## 4) Mapa rápido de código (onde mexer)

### Núcleo de execução
- Eventos (pub/sub): [`modules/Events.js`](modules/Events.js)
- Estado global: [`modules/StateManager.js`](modules/StateManager.js)
- Fila e execução de tarefas: [`modules/TaskQueue.js`](modules/TaskQueue.js)
- Cliente de requests do jogo: [`modules/GameClient.js`](modules/GameClient.js)
- Coleta/interceptação de dados: [`modules/DataCollector.js`](modules/DataCollector.js)

### Módulos de negócio
- Financeiro/build: [`modules/CFO.js`](modules/CFO.js)
- Logística/transporte: [`modules/COO.js`](modules/COO.js)
- Vinho/população: [`modules/HR.js`](modules/HR.js)
- Pesquisa: [`modules/CTO.js`](modules/CTO.js)
- Segurança/ruído: [`modules/CSO.js`](modules/CSO.js)
- Expansão/novas cidades: [`modules/MnA.js`](modules/MnA.js)

### Infra/config/auditoria
- Persistência: [`modules/Storage.js`](modules/Storage.js)
- Configuração: [`modules/Config.js`](modules/Config.js)
- Auditoria/logs: [`modules/Audit.js`](modules/Audit.js)
- Utilitários: [`modules/utils.js`](modules/utils.js)

### UI e integração com página
- Injeção no contexto da página: [`inject/inject.js`](inject/inject.js)
- Content script bridge: [`content/content.js`](content/content.js)
- Worker/background: [`background/background.js`](background/background.js)
- Bridge para UI: [`modules/UIBridge.js`](modules/UIBridge.js)
- Painel: [`ui/panel.html`](ui/panel.html), [`ui/panel.js`](ui/panel.js), [`ui/panel.css`](ui/panel.css)

### Dados estáticos
- Constantes: [`data/const.js`](data/const.js)
- Custos de edifícios: [`data/buildings.js`](data/buildings.js)
- Efeitos: [`data/effects.js`](data/effects.js)
- Vinho: [`data/wine.js`](data/wine.js)
- Pesquisa: [`data/research.js`](data/research.js)

---

## 5) “Se a tarefa for X, abra Y”

- “Bug em consumo/cache de recurso” → [`modules/ResourceCache.js`](modules/ResourceCache.js), [`modules/ResourceBalance.js`](modules/ResourceBalance.js), [`tests/unit/resourcecache.test.js`](tests/unit/resourcecache.test.js)
- “Build não entra/falha/duplica” → [`modules/CFO.js`](modules/CFO.js), [`modules/TaskQueue.js`](modules/TaskQueue.js), [`modules/GameClient.js`](modules/GameClient.js)
- “Transporte atrasado ou payload errado” → [`modules/COO.js`](modules/COO.js), [`modules/GameClient.js`](modules/GameClient.js), [`ENDPOINTS.md`](ENDPOINTS.md)
- “Problema de UI” → [`modules/UIBridge.js`](modules/UIBridge.js), [`ui/panel.js`](ui/panel.js), [`UI.md`](UI.md)
- “Falha de captura de dados do jogo” → [`modules/DataCollector.js`](modules/DataCollector.js), [`inject/inject.js`](inject/inject.js), [`GAME_MODEL.md`](GAME_MODEL.md)
- “Problema de storage/config” → [`modules/Storage.js`](modules/Storage.js), [`modules/Config.js`](modules/Config.js), [`content/content.js`](content/content.js)

---

## 6) Fluxos rápidos para agentes

### Fluxo A — corrigir bug
1. Identificar módulo dono (seção 5).
2. Ler teste existente correspondente em [`tests/unit`](tests/unit).
3. Corrigir no módulo alvo.
4. Rodar teste focado.
5. Rodar suíte completa.

### Fluxo B — implementar feature pequena
1. Confirmar regra em [`BUSINESS.md`](BUSINESS.md) e contrato técnico em [`ARCHITECTURE.md`](ARCHITECTURE.md).
2. Alterar módulo principal + dependências mínimas.
3. Adicionar/ajustar teste unitário.
4. Validar sem expandir escopo.

### Fluxo C — ajustar contrato HTTP/jogo
1. Validar parâmetros em [`ENDPOINTS.md`](ENDPOINTS.md).
2. Ajustar chamadas em [`modules/GameClient.js`](modules/GameClient.js).
3. Confirmar impactos em [`modules/TaskQueue.js`](modules/TaskQueue.js) e [`modules/DataCollector.js`](modules/DataCollector.js).

---

## 7) Comandos de validação

Scripts disponíveis em [`package.json`](package.json):

- `npm test` → executa testes uma vez
- `npm run test:watch` → modo watch
- `npm run test:coverage` → cobertura

---

## 8) Regras práticas para agentes (objetivas)

- Preferir mudanças pequenas e localizadas.
- Não refatorar módulos não relacionados à tarefa.
- Preservar nomes e contratos já usados por outros módulos.
- Sempre que possível, provar mudança com teste unitário.
- Em caso de dúvida de regra de negócio, priorizar [`BUSINESS.md`](BUSINESS.md); de implementação, priorizar [`ARCHITECTURE.md`](ARCHITECTURE.md).

---

## 9) Ordem de prioridade de fontes (desempate)

1. Código atual em `modules/`, `data/`, `inject/`, `content/`, `ui/`
2. [`ARCHITECTURE.md`](ARCHITECTURE.md)
3. [`BUSINESS.md`](BUSINESS.md)
4. Demais docs especializadas ([`ENDPOINTS.md`](ENDPOINTS.md), [`GAME_MODEL.md`](GAME_MODEL.md), [`UI.md`](UI.md), [`SCRAPER.md`](SCRAPER.md))

Quando houver divergência, reportar explicitamente no PR/commit o ponto de conflito e a decisão tomada.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emelinhares)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emelinhares)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
