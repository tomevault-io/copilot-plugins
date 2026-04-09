## sturdy-barnacle

> Bot de trading automatizado para Binance Futures escrito em Python (~5276 linhas em um unico arquivo). Opera scalping em pares USDT com multiplas camadas de analise tecnica, sentimento, microestrutura e aprendizado estatistico.

# CLAUDE.md — Analise Forense Completa do Cruzada Bot v3.0

## Visao Geral

Bot de trading automatizado para Binance Futures escrito em Python (~5276 linhas em um unico arquivo). Opera scalping em pares USDT com multiplas camadas de analise tecnica, sentimento, microestrutura e aprendizado estatistico.

**Arquivo principal:** `teste_binance23_ia2_adapt (1) (1).py`

---

## Arquitetura

### Threads (5 + main loop)
| Thread | Funcao | Problema |
|--------|--------|----------|
| `Precos` | Busca klines 1m/5m/15m de 200 pares | Gargalo: 600 API calls por ciclo |
| `Pares` | Atualiza lista de pares USDT a cada 5min | OK |
| `Trailing` | Monitora posicoes abertas (loop 200ms) | OK, bem implementado |
| `Memoria` | Salva estado em disco a cada 30s | OK |
| `MT-Score` | Recalcula score multi-temporalidade | Redundante com calculo no main |

### Pipeline de Decisao (em ordem)
```
1. calcular_indicadores_e_prever()     → RSI, MACD, BB, ADX, CCI, ATR em 1m/5m/15m
2. camada_decisao_extraterrestre()     → Tendencia multi-janela, reversao, momentum
3. construir_micro_modelo()            → Book L2 depth
4. [FAST LANE se forca >= 0.90]        → Execucao imediata sem esperar ranking
5. cerebro_decisao()                   → Order flow, funding rate, open interest
6. camada_invisivel()                  → Microestrutura (jerks, wicks, absorcao)
7. detectar_armadilha_baleia()         → Spoofing, wash trading, stop hunting
8. confirmar_entrada()                 → Regime, cooldown, exposicao, R:R
9. verificar_padrao_perda()            → Anti-fragilidade
10. validacao_final_ia()               → 10 testes + fingerprint + limiar adaptativo
11. preparar_e_executar()              → Ordem MARKET na Binance
```

### Sistema de "Aprendizado" (ia_memoria.json)
```
Fingerprint = 3 dimensoes × 3 categorias = 27 combinacoes
  - micro_trend: BOM / NEUTRO / RUIM
  - velocidade:  BOM / NEUTRO / RUIM
  - book:        BOM / NEUTRO / RUIM

Armazenamento:
  - _ia_historico: par → ultimas 50 entradas com resultado
  - _ia_padroes:   fingerprint → {wins, losses, total}
  - _ia_stats:     metricas globais (wins, losses, streak, limiar)
  - _ia_comportamento: fingerprint → curvas de preco pos-entrada
```

---

## DIAGNOSTICO: A "IA" E REALMENTE INTELIGENTE?

### O que ela FAZ (pontos positivos)
1. **Fingerprint simplificado** (linha 1662): 3 testes = repete rapido, aprende em ~20 trades
2. **Limiar adaptativo** (linha 1722): Ajusta exigencia baseado em win rate + streak
3. **Blacklist por par** (linha 1784): 3 losses seguidos = bloqueia 30min
4. **Aprendizado por horario** (linha 1818): Bloqueia horarios com <20% acerto
5. **Score por par** (linha 2500): Penaliza pares historicamente ruins
6. **Monitoramento 15s** (linha 3004): Verifica se preco subiu/caiu apos entrada
7. **Curvas pos-entrada** (linha 2437): Salva comportamento do preco apos trade

### O que ela NAO FAZ (pontos criticos)
1. **Nao e machine learning** — e contagem estatistica com regras fixas
2. **Nao faz backtesting** — impossivel saber se a estrategia funciona antes de perder dinheiro
3. **Nao tem protecao contra overfitting** — 27 combinacoes e pouco para generalizar
4. **Nao aprende feature importance** — nao sabe QUAIS indicadores importam mais
5. **Nao tem treinamento/validacao** — nao separa dados de treino e teste
6. **Nao aprende com contexto** — mesmo fingerprint em mercado calmo e extremo

---

## DIAGNOSTICO: OS FILTROS ESTAO CORRETOS?

### PROBLEMA CRITICO: EXCESSO DE FILTROS (Over-filtering)

O sinal passa por **~30 verificacoes** antes de virar trade. Cada filtro independente que permite ~70% dos sinais resulta em:

```
Probabilidade final = 0.7^30 = 0.02%
```

**Praticamente nenhum trade passa.**

### Mapa de filtros (redundancias marcadas com *)

```
PRE-FILTRO 1: Blacklist par (3 losses → 30min)
PRE-FILTRO 2: Par score < 0.25
PRE-FILTRO 3: Horario < 20% win rate
PRE-FILTRO 4: BTC moveu > 0.3% em 15min  *duplicado com teste 8
PRE-FILTRO 5: Volume < $500k/dia
PRE-FILTRO 5.5: ATR < threshold
PRE-FILTRO 6: Preco moveu > 0.5% desde sinal
PRE-FILTRO 7: Candles 5min contra
PRE-FILTRO 8: Momentum ultra-curto contra

TESTE 1: Micro-tendencia
TESTE 2: Velocidade + aceleracao
TESTE 3: Book L2 desequilibrio  *calculado 3x em camadas diferentes
TESTE 4: Spread
TESTE 5: Divergencia volume
TESTE 6: Posicao no range
TESTE 7: Candles recentes
TESTE 8: Correlacao BTC  *duplicado com pre-filtro 4
TESTE 9: Tempo do ciclo
TESTE 10: Comportamento pos-entrada

TRAVA DURA 1: Padrao historico < 45%
TRAVA DURA 2: micro_trend CONTRA
TRAVA DURA 3: 3+ testes CONTRA

confirmar_entrada(): regime, cooldown, exposicao, R:R, duplicata, horario, streak
verificar_padrao_perda(): anti-fragilidade
detectar_armadilha_baleia(): spoofing, wash, stop hunt, pump&dump

CONVERGENCIA: precisa 2+ votos de 5 fontes
```

### Filtros que MATAM sinais validos:
- **PRE-FILTRO 6** (preco moveu > 0.5%): No tempo que o bot leva para analisar 200 pares, QUALQUER sinal bom ja moveu 0.5%
- **PRE-FILTRO 5.5** (ATR baixo): Exclui moedas estaveis como BTC/ETH que sao as mais seguras
- **TRAVA DURA 2** (micro_trend CONTRA): Scalping contrarian PRECISA de micro_trend contra para pegar reversao

---

## PROBLEMAS GRAVES IDENTIFICADOS

### 1. RECALCULO MULTIPLO DO MESMO SINAL
O indicador de forca e recalculado em 5 camadas, cada uma sobrescrevendo a anterior:

```python
# Camada 1: calcular_indicadores_e_prever (linha 750)
indicador_forca = 0.0
indicador_forca += forca_5m  # direcao 5m

# Camada 2: camada_decisao_extraterrestre (linha 928)
indicador_forca = 0.0        # ← RESETA TUDO e recalcula
indicador_forca += 0.6       # tendencia multi-janela

# Camada 3: construir_micro_modelo (linha 1151)
indicador_forca = mm.get('indicador_forca', 0.0)  # pega da camada 2
indicador_forca += 0.08 * desequilibrio            # book ajusta

# Camada 4: cerebro_decisao (linha 3912)
ajuste_forca = 0.15 * direcao  # funding/flow ajustam

# Camada 5: camada_invisivel (linha 4080)
sinais.append(+0.7)  # microestrutura ajusta
```

**Problema:** RSI e verificado em 3 lugares, MACD em 3, book depth em 3. Os sinais sao double-counted.

### 2. DADOS OBSOLETOS NO MOMENTO DA EXECUCAO

```
Thread Precos: 200 pares × 3 timeframes × rate_limit(8/s) = 75 segundos por ciclo
Main loop: 200 pares × calcular + construir = +200 segundos
Cerebro: N pares × 3 API calls cada = +60 segundos
Validacao IA: 5 API calls por par = +30 segundos

TOTAL: ~6 minutos entre coleta e execucao
```

Para scalping de 5 minutos, dados de 6 minutos atras sao INUTEIS.

### 3. API CALLS EXCESSIVAS

```
validacao_final_ia() faz POR PAR:
  - book_ticker()      (linha 1859)
  - klines 1m limit=10 (linha 1870)
  - depth limit=20     (linha 1886)
  - klines 5m limit=3  (linha 1961)
  - klines BTCUSDT 1m  (linha 2159)

= 5 API calls × N pares candidatos
```

Com rate limit de 8/s, 10 candidatos = 6 segundos so de API na validacao.

### 4. FORCAR_1_TRADE_TESTE = True (linha 51)

```python
FORCAR_1_TRADE_TESTE = True  # ← PERIGOSO em producao
```

Forca um trade mesmo sem capital alocado. Em producao com dinheiro real, isso e irresponsavel.

### 5. API_KEY e API_SECRET VAZIAS (linhas 69-70) mas REAL_TRADES = True

```python
API_KEY = ""
API_SECRET = ""
REAL_TRADES = True  # ← vai tentar trades reais sem credenciais
```

### 6. CODIGO MORTO

```python
def _monitorar_posicoes_DESATIVADA():  # linha 2701 - 2947
    # 246 linhas de codigo que NUNCA executam
```

### 7. RACE CONDITIONS

```python
lock_pares = lock_precos  # linha 114 — MESMO lock para 2 propositos
```

Variaveis globais `_ia_stats`, `_ia_padroes`, `_ia_historico` sao modificadas sem lock em `validacao_final_ia()` e `ia_registrar_resultado()`, ambas podem rodar de threads diferentes.

### 8. DEFINICAO DE "WIN" E ENVIESADA

```python
# linha 2912 e 3347
direcao_certa = lucro_pct > -0.001  # tolerancia de 0.1%
```

Um trade que PERDEU 0.09% e contado como WIN para aprendizado. Isso infla artificialmente a taxa de acerto e faz o bot achar que esta aprendendo quando na verdade esta perdendo dinheiro com taxas.

### 9. SCALPING vs TIMEFRAME MISMATCH

O bot diz fazer scalping (5 min) mas:
- Usa indicadores de **15 minutos** para direcao (muito lento)
- Timeout padrao de **600 segundos** (10 minutos, nao 5)
- Stop loss de **1.5-2.5%** (largo demais para scalping, normal para swing)
- Trailing triggers em **0.3-2%** (swing trade, nao scalp)

### 10. CORRELACAO O(n^2)

```python
def calcular_correlacoes():  # linha 1447
    for i, p1 in enumerate(pares_snapshot):     # 200 pares
        for j, p2 in enumerate(pares_snapshot): # × 200 pares
            # = 20.000 comparacoes com numpy
```

Roda a cada 5 ciclos mas trava o main loop por dezenas de segundos.

### 11. SENTIMENTO RSS E FRACO

```python
def obter_sentimento_rss(par):  # linha 539
    # Analisa TITULO de noticias RSS com VADER
    score = analyzer_vader.polarity_scores(title)['compound']
```

- VADER foi treinado para ingles generico, nao crypto
- "Bitcoin drops 5%" → sentimento negativo, mas pode ser oportunidade de compra
- Titulo nao contem informacao suficiente para decisao de trade

### 12. CAPITAL FLOOR DE $30 ANULA GESTAO DE RISCO

```python
capital_base = max(capital_base, CAPITAL_MIN_ABSOLUTO)  # $30 SEMPRE
# ...depois...
cap = max(cap, 30.0)  # A3 FIX: Floor ABSOLUTO
```

Se todos os ajustes dizem "nao invista" (0.7 × 0.5 × 0.6 × 0.5 = 10.5%), o bot ainda coloca $30. Isso contradiz a logica de gestao de risco.

---

## O QUE FALTA PARA TRADES REALMENTE LUCRATIVOS

### Prioridade 1: BACKTESTING
Sem backtesting, e impossivel saber se a estrategia funciona. O bot precisa de:
- Dados historicos de klines
- Simulador que roda o pipeline completo offline
- Metricas: Sharpe ratio, max drawdown, win rate REAL, profit factor

### Prioridade 2: REDUZIR PIPELINE
Menos e mais. O pipeline atual tem tantas camadas que:
- Perde tempo (dados ficam obsoletos)
- Double-counts sinais
- Over-filtra (nenhum trade passa)

Pipeline ideal para scalping:
```
1. Indicadores 1m + 5m (RSI, MACD, BB)
2. Book L2 (desequilibrio + spread)
3. Confirmacao BTC
4. EXECUTA
```

4 etapas, nao 11.

### Prioridade 3: MACHINE LEARNING REAL
Substituir o sistema de fingerprints por um modelo real:
- Random Forest ou XGBoost com features numericas
- Features: RSI, MACD, ADX, volume_ratio, spread, book_imbalance, BTC_corr, hora
- Target: preco subiu/caiu > 0.1% em 5 minutos
- Treinamento com rolling window (ultimas 2 semanas)
- Validacao out-of-sample

### Prioridade 4: VELOCIDADE
Para scalping, cada segundo conta:
- WebSocket em vez de REST polling (elimina 90% das API calls)
- Processar apenas top 20 pares por volume, nao 200
- Pipeline paralelo (indicadores + book ao mesmo tempo)
- Cache inteligente com TTL por tipo de dado

### Prioridade 5: GESTAO DE RISCO CORRETA
- Remover floor de $30 (se o sinal e ruim, NAO ENTRA)
- Position sizing via Kelly criterion PURO (sem floor)
- Stop loss baseado em ATR, nao percentual fixo
- Max drawdown diario (para de operar se perdeu X% no dia)

---

## ESTRUTURA DO CODIGO

### Configuracao (linhas 1-170)
- Imports, API keys, constantes, rate limiter
- `FORCAR_1_TRADE_TESTE = True` ← remover em producao
- `REAL_TRADES = True` ← perigoso sem API keys

### Memoria e Cache (linhas 172-451)
- Salva/carrega estado em `memoria.json`
- Sincroniza posicoes com Binance (fonte da verdade)
- Cache de exchange info e filtros (TTL 300s)

### Coleta de Dados (linhas 454-560)
- Thread de atualizacao de pares USDT
- Thread de precos (1m/5m/15m klines)
- Sentimento via RSS + VADER

### Indicadores Tecnicos (linhas 562-855)
- Multi-temporalidade (1m, 5m, 15m)
- RSI, MACD, Bollinger, ADX, CCI, ATR
- Decisao por 5m com timing por 1m

### Camadas de Decisao (linhas 858-1249)
- `camada_decisao_extraterrestre`: tendencia multi-janela
- `construir_micro_modelo`: Book L2

### Cerebro IA (linhas 1575-2560)
- Sistema de fingerprints e aprendizado
- Validacao final com 10 testes
- Registro de resultados e atualizacao de padroes

### Execucao (linhas 2562-2691)
- `preparar_e_executar`: calcula quantidade, verifica filtros Binance, envia ordem

### Trailing Stop (linhas 2693-3384)
- Thread independente com client proprio
- Velocimetro (velocidade + aceleracao do preco)
- Classificacao de personalidade da moeda (spike/volatil/calma/normal)
- 10 regras de saida (stop loss, early exit, trailing, profit lock, etc)

### Regime e Adaptacao (linhas 3387-3647)
- Detecta regime via BTC (bull/bear/chop/neutro)
- Adapta parametros: stop_loss, timeout, trailing, risco, max_posicoes

### Sinais Preditivos (linhas 3649-4037)
- Order flow delta
- Funding rate
- Open interest
- Cerebro que integra tudo

### Camada Invisivel + Baleias (linhas 4039-4471)
- Derivadas (velocidade, aceleracao, jerk)
- Divergencia volume-preco
- Vacuo de liquidez
- Padrao de wicks
- Absorcao silenciosa
- Deteccao de spoofing, wash trading, stop hunting, pump&dump

### Main Loop (linhas 4674-5276)
- Pipeline de 2 fases (scan rapido + analise profunda)
- Fast lane para sinais muito fortes
- Ranking por convergencia
- Execucao com confirmacao multipla
- Painel de monitoramento

---

## CONVENCOES DO CODIGO

- Linguagem: Python 3
- Dependencias: numpy, pandas, ta, vaderSentiment, python-binance, feedparser, colorama
- Estilo: snake_case para funcoes e variaveis
- Locks: threading.Lock/RLock para estado compartilhado
- Logging: modulo logging + print colorido
- Persistencia: JSON files (memoria.json, ia_memoria.json)
- Sem testes automatizados
- Sem type hints
- Sem docstrings consistentes (algumas funcoes tem, maioria nao)

---

## RESUMO EXECUTIVO

| Aspecto | Status | Nota |
|---------|--------|------|
| A IA aprende? | Parcialmente | Conta wins/losses por padrao, mas nao e ML real |
| Filtros estao certos? | Excessivos | ~30 filtros = quase nenhum trade passa |
| Entra em trades lucrativos? | Improvavel | Dados obsoletos + over-filtering |
| Gestao de risco? | Contraditoria | Floor de $30 anula ajustes de risco |
| Velocidade? | Lenta demais | 6+ min de pipeline para scalping de 5min |
| Codigo limpo? | Nao | 5276 linhas em 1 arquivo, codigo morto, race conditions |
| Backtesting? | Inexistente | Sem forma de validar estrategia |
| Pronto para producao? | Nao | API keys vazias, FORCAR_TRADE=True, race conditions |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/industriacatarine-debug)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/industriacatarine-debug)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
