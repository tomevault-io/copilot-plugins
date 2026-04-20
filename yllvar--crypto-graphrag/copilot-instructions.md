## crypto-graphrag

> You are building an **Agent-Based Graph RAG system for cryptocurrency market analysis** using:


# Agent-Based Graph RAG Development Rules

## Project Context
You are building an **Agent-Based Graph RAG system for cryptocurrency market analysis** using:
- **Knowledge Graph**: Graphiti (Neo4j-based)
- **LLM/Embeddings**: Together.AI (Llama-3-70B, e5-mistral-7b)
- **Agent Framework**: LangChain + LlamaIndex
- **Data Ingestion**: CCXT (Python)
- **Backend**: FastAPI
- **Real-Time**: Redis Streams
- **Time-Series**: TimescaleDB
- **Deployment**: Railway via GitHub

## Core Architecture Principles
1. **Temporal Context First**: All market data must include timestamps and time relationships
2. **Multi-Agent Collaboration**: Technical, Correlation, Event Impact, and Prediction agents work together
3. **Real-Time Validation**: Always verify graph data with live CCXT calls before responding
4. **Predictive Focus**: Prioritize generating forecasts with confidence intervals over historical analysis

## Code Generation Guidelines
1. **Graph Schema**:
   - Use Neo4j Cypher syntax
   - Include temporal properties: `timestamp`, `time_window`
   - Relationship weights must include confidence scores: `CORRELATED_WITH {weight: 0.8, confidence: 0.9}`

2. **Agent Implementation**:
   - All agents inherit from `BaseAgent` class
   - Include `validate_with_realtime()` method using CCXT
   - Return structured predictions with:
     ```python
     {
         "value": float,
         "confidence": float,  # 0-1
         "horizon": str,       # e.g., "24h", "7d"
         "indicators": list,
         "sources": list
     }
     ```

3. **API Endpoints**:
   - All endpoints accept `symbols: List[str]` and `horizon_days: int`
   - Include `/health` endpoint monitoring all services
   - Implement rate limiting for Together.AI calls

4. **Data Flow**:
   ```mermaid
   CCXT → Redis Streams → TimescaleDB
                 ↓
            Graphiti ← Together.AI Embeddings
                 ↓
             Agents → FastAPI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yllvar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
