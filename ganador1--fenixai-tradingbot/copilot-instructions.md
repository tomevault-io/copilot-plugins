## fenixai-tradingbot

> FenixAI employs a **multi-agent architecture** where specialized AI agents collaborate to analyze cryptocurrency markets. Each agent is an expert in a specific domain, and their outputs are combined through a weighted consensus mechanism.

# 🤖 FenixAI v2.0 - Agent System

## Overview

FenixAI employs a **multi-agent architecture** where specialized AI agents collaborate to analyze cryptocurrency markets. Each agent is an expert in a specific domain, and their outputs are combined through a weighted consensus mechanism.

## Agent Hierarchy

```
                    ┌─────────────────────────────┐
                    │     LangGraph Orchestrator   │
                    │      (State Machine)         │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   Technical   │       │    Visual     │       │   Sentiment   │
│    Agent      │       │    Agent      │       │    Agent      │
│  (30% weight) │       │  (25% weight) │       │  (15% weight) │
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │     QABBA Agent       │
                    │    (30% weight)       │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    Decision Agent     │
                    │  (Weighted Consensus) │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    Risk Manager       │
                    │   (Final Approval)    │
                    └───────────────────────┘
```

---

## 1. Technical Analyst Agent

**File:** `src/agents/enhanced_technical_analyst.py`

### Purpose

Analyzes market structure, technical indicators, and price patterns to generate trading signals.

### Capabilities

- **Indicator Analysis**: RSI, MACD, ADX, ATR, Bollinger Bands
- **Market Structure**: Trend detection, support/resistance levels
- **Multi-timeframe**: Analyzes multiple timeframes for confluence
- **Validation**: Checks indicator consistency before signaling

### Input

```python
{
    "kline_data": {"open": [...], "high": [...], "low": [...], "close": [...], "volume": [...]},
    "indicators": {"rsi": 45.2, "macd": {...}, "adx": 28.5, ...},
    "mtf_context": {"15m": {...}, "1h": {...}, "4h": {...}}
}
```

### Output (EnhancedTechnicalOutput)

```python
{
    "signal": "BUY" | "SELL" | "HOLD",
    "confidence": 0.75,  # 0.0 - 1.0
    "reasoning": "RSI showing oversold conditions with bullish divergence...",
    "confluence_score": 0.68,
    "entry_price": 42150.0,
    "stop_loss": 41800.0,
    "take_profit": 43200.0,
    "risk_reward_ratio": 2.5
}
```

### Configuration

```yaml
# config/fenix.yaml
agents:
  technical_weight: 0.30
  enable_technical: true
```

---

## 2. Visual Analyst Agent

**File:** `src/agents/visual_analyst_enhanced.py`

### Purpose

Analyzes TradingView chart screenshots to identify visual patterns that may not be captured by numerical indicators.

### Capabilities

- **Pattern Recognition**: Head & Shoulders, triangles, flags, wedges
- **Candlestick Analysis**: Doji, engulfing, hammer, shooting star
- **Trend Analysis**: Visual trend lines, channels
- **Indicator Reading**: Interprets SuperTrend, SAR points from chart
- **Security**: Validates chart paths to prevent stale data analysis

### Input

```python
{
    "chart_path": "/path/to/BTCUSDT_15m_chart.png",
    "symbol": "BTCUSDT",
    "timeframe": "15m"
}
```

### Output (EnhancedVisualChartAnalysisOutput)

```python
{
    "action": "BUY" | "SELL" | "HOLD",
    "confidence": 0.72,
    "reason": "Bullish engulfing pattern at key support level...",
    "key_candlestick_patterns": ["bullish_engulfing", "morning_star"],
    "chart_patterns": ["ascending_triangle"],
    "trend_analysis": {"direction": "BULLISH", "strength": 0.7},
    "support_resistance_levels": {"support": [41500, 41000], "resistance": [43000, 44500]},
    "next_candle_prediction": {"direction": "UP", "probability": 0.65}
}
```

### Requirements

- Vision-capable LLM (qwen3-vl:8b or similar)
- TradingView chart screenshots via Playwright

---

## 3. Sentiment Analyst Agent

**File:** `src/agents/sentiment_enhanced.py`

### Purpose

Analyzes news articles, social media, and market sentiment to gauge market psychology.

### Capabilities

- **News Analysis**: Crypto news from multiple sources
- **Social Sentiment**: Twitter/X, Reddit analysis
- **Fear & Greed**: Interprets market sentiment indices
- **Event Detection**: Identifies market-moving events

### Input

```python
{
    "news_data": [
        {"title": "Bitcoin ETF sees record inflows", "source": "coindesk", "sentiment": "positive"},
        ...
    ],
    "social_data": {"twitter": {...}, "reddit": {...}},
    "fear_greed_index": 65
}
```

### Output

```python
{
    "sentiment": "POSITIVE" | "NEGATIVE" | "NEUTRAL",
    "confidence": 0.68,
    "reasoning": "Multiple positive news about ETF inflows...",
    "key_events": ["ETF inflows record", "whale accumulation"],
    "social_sentiment_score": 0.72
}
```

---

## 4. QABBA Agent (Quantitative Analysis Bollinger Bands Agent)

**File:** `src/agents/enhanced_qabba_agent.py`

### Purpose

Specialized quantitative agent focusing on Bollinger Bands, volatility analysis, and squeeze detection.

### Capabilities

- **Bollinger Band Analysis**: Position, width, squeeze
- **Volatility Assessment**: ATR-based volatility scoring
- **Breakout Probability**: Predicts potential breakouts
- **Quality Assurance**: Validates other agents' outputs

### Output

```python
{
    "qabba_signal": "BUY_QABBA" | "SELL_QABBA" | "HOLD_QABBA" | "NEUTRAL_QABBA",
    "qabba_confidence": 0.78,
    "bb_position": "lower" | "middle" | "upper",
    "squeeze_detection": true,
    "breakout_probability": 0.65,
    "volatility_state": "LOW" | "MEDIUM" | "HIGH"
}
```

---

## 5. Decision Agent

**File:** `src/agents/decision.py`

### Purpose

Aggregates all agent outputs using **dynamic weighting** to make the final trading decision.

### Dynamic Weighting System

Weights are adjusted based on:

1. **Historical Performance**: Agents with higher accuracy get more weight
2. **Market Conditions**: Different weights for trending vs. ranging markets
3. **Confidence Scores**: Higher confidence increases influence

```python
# Default weights
weights = {
    'technical': 0.30,
    'qabba': 0.30,
    'visual': 0.25,
    'sentiment': 0.15
}

# Market condition adjustments
market_adjustments = {
    'high_volatility': {'sentiment': 0.8, 'technical': 1.2, 'visual': 1.1},
    'trending': {'sentiment': 0.9, 'technical': 1.3, 'visual': 1.2},
    'ranging': {'sentiment': 1.1, 'technical': 0.9, 'qabba': 1.2}
}
```

### Output (FinalDecisionOutput)

```python
{
    "final_action": "BUY" | "SELL" | "HOLD",
    "final_confidence": 0.72,
    "weighted_scores": {"BUY": 0.65, "SELL": 0.15, "HOLD": 0.20},
    "agent_contributions": {
        "technical": {"action": "BUY", "weight": 0.30, "confidence": 0.75},
        "visual": {"action": "BUY", "weight": 0.25, "confidence": 0.70},
        ...
    },
    "consensus_reasoning": "Strong bullish signals across multiple agents..."
}
```

---

## LLM Provider Configuration (Groq Example)

FenixAI supports multiple provider profiles configured in `config/llm_providers.yaml`. To use the Groq free profile set the environment variable or the `active_profile` to `groq_free`.

Recommended free Groq models (examples):

- `llama-3.3-70b-versatile` — High-throughput Llama 3 model for text reasoning.
- `meta-llama/llama-4-scout-17b-16e-instruct` — Current Groq multimodal (vision) option.
- `mixtral-8x7b-32768` — Efficient model available in some Groq tiers.

TIP: To use the `groq_free` profile set the `GROQ_API_KEY` environment variable and restart the API.

Example `.env` entries:

```
GROQ_API_KEY=gsk_...
LLM_PROFILE=groq_free
```

Fallback behavior: If a specific provider package (e.g., `langchain_groq`) is missing on the machine, the loader will attempt to fall back to the configured `fallback_provider_type` (e.g., `ollama_local`). If fallback fails too and `LLM_ALLOW_NOOP_STUB=1` (development), the system initializes a no-op stub LLM to keep the graph alive for testing.

---

## 6. Risk Manager

**File:** `src/agents/risk.py`

### Purpose

Final gatekeeper that can **veto** any trade that exceeds risk parameters.

### Risk Controls

| Control | Description |
|---------|-------------|
| **Max Risk per Trade** | Maximum 2% of portfolio per trade |
| **Max Total Exposure** | Maximum 5% total open positions |
| **Circuit Breaker** | Stops trading after X consecutive losses |
| **Daily Loss Limit** | Halts trading if daily loss exceeds threshold |
| **Position Sizing** | Dynamic sizing based on volatility |

### Output

```python
{
    "approved": true | false,
    "risk_score": 0.35,  # 0.0 (safe) - 1.0 (max risk)
    "position_size": 0.01,  # BTC
    "stop_loss": 41800.0,
    "take_profit": 43200.0,
    "rejection_reason": null | "Exceeds max daily loss"
}
```

---

## LLM-as-Judge (ReasoningBank Integration)

**File:** `src/inference/reasoning_judge.py`

After each decision, an LLM evaluates the quality of the reasoning:

```python
{
    "verdict": "SOUND" | "QUESTIONABLE" | "POOR",
    "score": 0.82,
    "confidence": 0.75,
    "critique": "Decision well-supported by technical confluence...",
    "risks": ["High leverage in volatile market"],
    "improvements": ["Consider adding volume confirmation"]
}
```

---

## Agent Configuration

All agents can be enabled/disabled and weighted via `config/fenix.yaml`:

```yaml
agents:
  # Enable/disable agents
  enable_technical: true
  enable_qabba: true
  enable_visual: false  # Requires vision model
  enable_sentiment: false  # Requires news APIs
  
  # Agent weights (must sum to 1.0)
  technical_weight: 0.30
  qabba_weight: 0.30
  visual_weight: 0.25
  sentiment_weight: 0.15
  
  # Decision thresholds
  consensus_threshold: 0.65
  min_confidence_to_trade: MEDIUM
```

---

## Adding Custom Agents

To add a new agent:

1. Create agent file in `src/agents/`:

```python
class CustomAgent(EnhancedBaseLLMAgent, LLMProviderMixin):
    name = "CustomAgent"
    role = "Custom Analysis Specialist"
    
    async def analyze(self, data: dict) -> CustomOutput:
        # Implement analysis logic
        pass
```

1. Register in `src/core/langgraph_orchestrator.py`:

```python
# Add node to graph
graph.add_node("custom_agent", self._custom_agent_node)
graph.add_edge("previous_node", "custom_agent")
```

1. Add configuration in `config/fenix.yaml`:

```yaml
agents:
  enable_custom: true
  custom_weight: 0.20
```

---

**See also:**

- [ARCHITECTURE.md](ARCHITECTURE.md) - System architecture
- [API.md](API.md) - REST API reference

---
> Source: [Ganador1/FenixAI_tradingBot](https://github.com/Ganador1/FenixAI_tradingBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
