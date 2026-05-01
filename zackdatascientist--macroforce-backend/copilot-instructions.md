## macroforce-backend

> O **Macro Core Service** é o cérebro quantitativo do ecossistema MacroForce. Ele opera como uma API "Headless" (sem interface própria) que consome dados de mercado, processa algoritmos matemáticos (Scoring Inteligente, Volatilidade) e fornece um **Dashboard Unificado (JSON)** para o Front-end React.

# Product Requirements Document (PRD) - Macro Core Service v4.6

## 1. Visão do Produto
O **Macro Core Service** é o cérebro quantitativo do ecossistema MacroForce. Ele opera como uma API "Headless" (sem interface própria) que consome dados de mercado, processa algoritmos matemáticos (Scoring Inteligente, Volatilidade) e fornece um **Dashboard Unificado (JSON)** para o Front-end React.

**Arquitetura de Motores Independentes:**
1.  **Macro-D Engine (Smart Scoring):** Consome dados em tempo real do *Ingester* (33 ativos brutos) para medir o fluxo com ponderação inteligente.
2.  **Volatility Engine (Grid):** Consome dados históricos da *AwesomeAPI* (USD-BRL) para calcular níveis de preço.
3.  **TrendEngine (Context):** Calcula o desvio estatístico (Z-Score) para validar a qualidade do fluxo.

---

## 2. Requisitos Funcionais (RFs)

### [RF-01] Classificação e Scoring (Motor Macro-D)
O sistema deve processar **todos os 33 ativos** recebidos do Ingester. A variação destes ativos compõe o cálculo de fluxo (Risco vs Segurança).

#### **1. Ativos de Risco (Risk ON)**
*(Regra de Correlação: Alta destes ativos = Pressão de BAIXA no Dólar)*

* **Commodities (Indicadores de Atividade Econômica):**
    * `CL` (Petróleo WTI)
    * `GC` (Ouro)
    * `HG` (Cobre)
    * `ZS` (Soja)
    * `I0.DCE` (Minério de Ferro - Dalian)
* **Índices e Futuros Globais:**
    * `.GDOW` (Global Dow)
    * `YMH26` (Dow Jones Futuro - Contrato Março/26)
    * `.OSEAX` (Oslo Benchmark - Proxy de Energia)
    * `.BSESN` (SENSEX Índia)
* **ETFs e Setores (S&P Spiders):**
    * `XLF` (Financeiro)
    * `XLP` (Consumo Básico)
    * `XLE` (Energia)
    * `XME` (Metais e Mineração)
    * `SOXX.O` (Semiconductores - Tech Risk)
    * `EEM` (Mercados Emergentes)
* **Ativos Brasileiros (ADRs/ETFs):**
    * `VALE.K` (ADR Vale)
    * `PBR` (ADR Petrobras)
    * `EWZ` (ETF MSCI Brazil)
* **Bloco China (Agregador Asiático):**
    * *Componentes:* `HSIQF6`, `CHINA50`, `.DJSH`, `.SZI`, `.SSEC`.
    * *Tratamento:* Média aritmética dos 5 ativos conta como **1 único ativo sintético**.

#### **2. Ativos de Segurança (Risk OFF)**
*(Regra de Correlação: Alta destes ativos = Pressão de ALTA no Dólar)*

* **Volatilidade e Dólar Global:**
    * `VX` (VIX - Índice do Medo)
    * `DX` (DXY - Dólar Index)
* **Pares de Moedas (Forex - USD vs Mundo):**
    * `USD/MXN` (Peso Mexicano)
    * `USD/NOK` (Coroa Norueguesa)
    * `USD/NZD` (Dólar Neozelandês)
    * `USD/AUD` (Dólar Australiano)
    * `USD/KRW` (Won Sul-Coreano)
    * `USD/CNY` (Yuan Chinês Offshore)
    * `EUR/BRL` (Euro vs Real)

---

### [RF-02] Regras de Pontuação (Thresholds Imutáveis)
Se a variação do ativo romper o limite, soma 1 ponto base para a direção correspondente. Caso contrário, é considerado Neutro (0). **Esta regra define a entrada do ativo no cálculo, mas não seu peso final.**

1.  **Regra Específica - Minério de Ferro (`I0.DCE`):**
    * Threshold: **3.00%**
    * *Pontuação Base:* ±1 ponto.

2.  **Regra Específica - VIX (`VX`):**
    * Threshold: **5.00%**
    * *Pontuação Base:* ±1 ponto.

3.  **Regra Padrão (Demais 31 ativos):**
    * Threshold: **0.30%**
    * *Pontuação Base:* ±1 ponto.

---

### [RF-03] Sanitização de Tickers
Os dados brutos do Ingester podem conter descrições (ex: `USD/MXN - Dólar Americano...`).
* **Regra:** O sistema deve cortar a string no primeiro separador " - ", utilizando apenas o código do ativo para o match com as regras.

---

### [RF-04] Pré-Mercado: Cálculo de Variação Estendida
**Descrição:** Para ativos com pré-mercado (ADRs e ETFs), calcula a variação % usando preço estendido se disponível.
**Ativos Elegíveis:** `VALE.K`, `PBR`, `EWZ`, `XLF`, `XLP`, `XLE`, `XME`, `EEM`, `SOXX.O`.

**Fórmula:**
Se `precoEstendido` != null: `((PrecoEstendido - PrecoAnterior) / PrecoAnterior) * 100`.
Caso contrário: considera neutro ou variação padrão.

---

### [RF-05] Agregador China
Tratamento de bloco para os índices asiáticos para evitar peso excessivo da região.
* **Input:** `HSIQF6`, `CHINA50`, `.DJSH`, `.SZI`, `.SSEC`.
* **Lógica:** Média aritmética das variações percentuais.
* **Peso:** O resultado consolidado conta como a variação de **1 único ativo de Risco**.
* **Integridade:** Se um componente for nulo, o bloco retorna neutro (0).

**Fórmula:** `varChina % = (varHangSeng + varChinaA50 + varDJShanghai + varSZSE + varShanghaiComp) / 5`

---

### [RF-06] Indicadores de Fluxo Ponderados (Smart Momentum)
O sistema deve implementar uma arquitetura de "Separation of Concerns" (Separação de Responsabilidades), distinguindo claramente a **Contabilidade de Tendência** (Saldo) da **Física de Movimento** (Oscilador), utilizando uma frequência de atualização híbrida.

* **Frequência de Execução (Cron):** A cada **1 minuto** (Cron Trigger: `0 */1 * * * *`).

#### **Estratégia Híbrida de Atualização:**
Para garantir fluidez visual no gráfico sem gerar ruído na tendência acumulada, o sistema deve operar com duas frequências lógicas:
1.  **Oscilador (Física):** Calculado e emitido em **todo ciclo (1 min)**.
2.  **Saldo (Contabilidade):** Atualizado e persistido apenas em **marcos de 5 minutos** (ex: 09:00, 09:05, 09:10...). Nos minutos intermediários, o sistema repete o último saldo consolidado.

#### **1. Saldo Acumulado (Tendência Estrutural)**
Este indicador representa o "fundo" do mercado e deve ser resiliente a ruídos.
* **Trigger de Atualização:** Apenas quando `(currentMinute % 5 == 0)`.
* **Lógica:** Contabilidade Simples e Rígida.
* **Regra:** Utiliza estritamente os pontos base definidos no **[RF-02]** (+1 ou -1).
* **Fórmula:** `Saldo[t] = Saldo[t-5min] + Soma(Pontos_Base_RF02)`.

#### **2. Oscilador de Fluxo (Smart Momentum)**
Este indicador representa a "força" e "agressividade" do movimento atual, servindo como gatilho de entrada.
* **Trigger de Atualização:** Todo minuto (t-1min).

**Camadas de Ponderação:**

* **Camada A: Qualidade Individual (Por Ativo)**
    * **Magnitude:** Se `abs(var%) > 1.00%`, aplica multiplicador **1.5x**. Se `> 0.60%`, aplica **1.2x**.
    * **Aceleração (Velocity):** Compara a variação atual com o snapshot de **1 minuto atrás**.
        * Se `(VarAtual - Var_1min_Atras) > 0.15%`: Multiplicador **1.2x** (Impulso Rápido).
        * Se Delta < 0.05%: Multiplicador **0.8x** (Lentidão).

* **Camada B: Sincronicidade (Por Grupo)**
    * **Coerência de Risco:** Se >75% dos ativos do grupo [RF-01: Risco] estiverem na mesma direção, aplica multiplicador **1.3x** no score total.
    * **Indecisão:** Se a direção estiver dividida (40%-60%), aplica multiplicador **0.7x**.

* **Camada C: Contexto de Preço (TrendEngine)**
    * Acessa o Z-Score calculado pelo **[RF-12]** para validar o contexto estatístico.
    * **Regra de Pesos:**
        * **Z-Score > +3.0 (Caro):** Peso **0.5x** para Compra / **1.5x** para Venda.
        * **Z-Score < -3.0 (Barato):** Peso **1.5x** para Compra / **0.5x** para Venda.
        * **Z-Score Neutro (-1 a +1):** Peso **1.0x**.

**Fórmula do Oscilador:**
`Oscilador[t] = (Soma(ScoreBase * Magnitude * Aceleração) * Sincronicidade) * MultiplicadorZScore`

---

### [RF-07] Barra de Pressão Tricolor
Visualização da distribuição percentual do mercado em tempo real (Venda / Neutro / Compra).
* **Frequência de Atualização:** Tempo Real (push via WebSocket ou Polling de 10s no front).

**Cores:**
1. 🔴 **Vermelho:** % Risco (Ativos indicando queda do Dólar).
2. ⚪ **Cinza:** % Neutro (Ativos dentro da lateralidade).
3. 🟢 **Verde:** % Segurança (Ativos indicando alta do Dólar).

**Lógica de Contagem:**
* Total de Ativos Considerados: **28** (19 de Risco Consolidado + 9 Segurança).
* **Pré-requisito:** O Bloco China deve ser calculado e agregado em 1 ativo sintético **ANTES** da contagem.

**Regra de Direção:**
* **Ativos de Risco:**
    * Variação Positiva (> Threshold) → 🔴 (Venda de Dólar / Otimismo).
    * Variação Negativa (< -Threshold) → 🟢 (Compra de Dólar / Pessimismo).
* **Ativos de Segurança:**
    * Variação Positiva (> Threshold) → 🟢 (Compra de Dólar / Pessimismo).
    * Variação Negativa (< -Threshold) → 🔴 (Venda de Dólar / Otimismo).

---

### [RF-08] Motor Quantitativo: Volatilidade (Grid Adaptativo P-Rank)
Módulo **totalmente independente** da ingestão de dados em tempo real, responsável por calibrar o tamanho do grid operacional com base na história recente do ativo.

* **Responsável:** `VolatilityEngine`.
* **Fonte de Dados:** Endpoint externo AwesomeAPI (`USD-BRL/360`).
* **Ativo Alvo:** `WDO` (Dólar Futuro).
* **Lógica de Normalização por Percentil (P-Rank):**
    1.  **Lookback:** Analisa os últimos **250 dias** (1 ano letivo) de volatilidade (ATR-30).
    2.  **Ranking:** Calcula em qual percentil a volatilidade *atual* se encontra em relação ao histórico.
        * *Fórmula:* `Percentil = (Dias com ATR menor que hoje) / Total de Dias`.
    3.  **Escalonamento Linear:** Aplica um fator de correção dinâmico baseado no percentil.
        * *Fórmula:* `Fator = 0.75 + (Percentil * 0.50)`.
        * *Range:* O Grid oscila entre **0.75x** (Mínima Histórica) e **1.25x** (Máxima Histórica).
* **Output:** Objeto `quantGrid` com níveis de preço ajustados pela "temperatura" histórica do mercado.

---

### [RF-09] Persistência e Replay de Histórico (Contextual Snapshots)
Endpoint otimizado para entregar blocos de contexto ao invés de grandes volumes de dados brutos que prejudicam a visualização.

#### **1. API de Histórico (`/api/v1/macro/history`)**
O sistema deve entregar "fatias" de tempo (Slices).
* **Inputs:** `date`, `endTime`, `windowSizeMinutes` (Default: 180).
* **Processamento:**
    * Recupera dados brutos do dia inteiro (desde 00:00) para calcular o saldo acumulado correto até o momento do `start` da janela.
    * Retorna apenas os registros dentro do intervalo [`endTime` - `windowSize`, `endTime`].
    * Isso garante que o gráfico inicie com o valor de saldo correto (ex: -900) e desenhe a variação a partir dali, sem carregar o dia todo.

---

### [RF-10] Comportamento Geral da Interface e Sessão
Define as regras globais de visualização e integridade matemática.

1.  **Isolamento de Sessão Diária (No D-1 Carryover):**
    * O sistema não carrega saldos de dias anteriores. Abertura sempre em 0.0.

2.  **Inicialização com Contexto (Auto-Hydration):**
    * Ao carregar (F5), o Frontend deve solicitar imediatamente a janela histórica das **últimas 3 horas** (via RF-09) para preencher a tela instantaneamente, sem esperar o próximo tick.

---

### [RF-11] Gerenciamento de Densidade e Escala Visual
Requisito crítico para garantir legibilidade (evitar gráficos "esmagados" ou "riscos finos").

#### **1. Janela de Visualização Fixa (Fixed Data Window)**
* A densidade máxima de dados na tela é de **180 minutos** (3 horas).
* **Comportamento AO VIVO (Sliding Window):**
    * A janela é dinâmica e "desliza" com o tempo.
    * A cada minuto que passa, um minuto novo entra à direita e um minuto antigo sai à esquerda.
* **Comportamento HISTÓRICO (Static Window):**
    * A janela é estática, focada no horário selecionado pelo usuário.

#### **2. Escala Dinâmica de Eixos (Twin Scales)**
O gráfico deve utilizar escalas independentes para garantir a visualização correta de indicadores com magnitudes diferentes.
* **Eixo Principal (Direita - Saldo):** Deve possuir escala **Simétrica Dinâmica**. Se o saldo atingir `-500`, o eixo deve renderizar de `-600` a `+600` (com margem de segurança) para manter o Zero no centro da tela.
* **Eixo Secundário (Overlay - Oscilador):** Deve possuir escala **Fixa Visual** (Ex: -25 a +25). Isso garante que o Oscilador seja desenhado como "ondas" sobrepostas ao Saldo, sem distorcer a leitura de nenhum dos dois.

---

### [RF-12] TrendEngine (Linear Regression Channel)
Motor estatístico responsável por calcular o contexto de preço e o Z-Score para ponderação do fluxo.

1.  **Definição do Algoritmo:**
    * **Modelo:** Regressão Linear Simples (OLS - Ordinary Least Squares).
    * **Base de Dados:** Velas de **5 minutos**.
    * **Período de Análise (Lookback):** **114 períodos** (aprox. 9h30 de pregão).

2.  **Cálculo do Z-Score:**
    * O sistema deve calcular o desvio padrão (StdDev) dos resíduos da regressão.
    * **Fórmula:** `Z-Score = (PreçoAtual - PreçoRegressãoCentral) / StdDev`.

3.  **Integração com Oscilador:**
    * O TrendEngine deve fornecer o último `Z-Score` calculado para o Motor Macro-D ([RF-06]) em tempo real.
    * Se o mercado estiver fechado ou sem dados suficientes (< 114 velas), o Z-Score assume valor **0.0** (Neutro).

---

## 3. Arquitetura de Dados (Payload Bruto do Ingester)

O `MarketDataSyncService` deve processar a seguinte estrutura JSON (exemplo real):

```json
[
  {
    "id": 11496,
    "ticker": "GC",
    "price": 4496.10,
    "variation": -0.92,
    "extendedPrice": null,
    "extendedVariation": null,
    "timestamp": "2026-01-07T12:00:03Z"
  },
  {
    "id": 11499,
    "ticker": ".OSEAX",
    "price": 1954.56,
    "variation": -0.90,
    "extendedPrice": null,
    "extendedVariation": null,
    "timestamp": "2026-01-07T12:00:03Z"
  }
]

---
> Source: [ZackDataScientist/macroforce-backend](https://github.com/ZackDataScientist/macroforce-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
