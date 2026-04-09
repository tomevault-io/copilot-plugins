## agent-walkthrough

> │   ├── 01_pure_python_walkthrough.ipynb

# Projectplan: Agentic AI Demo met CBS API & MCP

## Projectstructuur
```
agentic-ai-demo/
├── notebooks/
│   ├── 01_pure_python_walkthrough.ipynb
│   ├── 02_pydantic_ai_walkthrough.ipynb
│   └── 03_langchain_walkthrough.ipynb
├── mcp_server/
│   ├── tools_server.py
│   ├── requirements.txt
│   └── tools/
│       ├── btw_tool.py
│       ├── discount_tool.py
│       └── cbs_api_tool.py
├── shared_tools.py
├── .env
├── requirements.txt
└── data/
    └── conversation_history.json
```

## Walkthrough Structuur (per notebook)

### Stap 1: Simpele Model Call
- **Pure Python**: Direct Anthropic API call met claude-3-haiku-20240307
- **Pydantic AI**: Basis chat zonder tools
- **LangChain**: Simpele LLM call
- **Demo**: Basis conversatie en tekst generatie

### Stap 2: Tool Definition & Function Calling
- **Pure Python**: Anthropic function calling
- **Pydantic AI**: Tool definition met Pydantic models
- **LangChain**: Custom tool met @tool decorator
- **Demo**: BTW en korting berekeningen

### Stap 3: Alle Tools Integration
- **BTW Tool**: BTW berekeningen met verschillende tarieven
- **Korting Tool**: Kortingsberekeningen
- **CBS API Tool**: Haal inwonersaantallen Nederlandse gemeenten (1/1/2025)
- **API endpoint**: CBS Open Data API
- **Output**: JSON met gemeente naam, code, inwoners
- **Implementatie**: HTTP requests naar CBS StatLine API

### Stap 4: Conversatie Geheugen
- **Pure Python**: Simpele lijst met chat history
- **Pydantic AI**: Built-in conversation history
- **LangChain**: ConversationBufferMemory
- **Persistentie**: JSON file voor session continuïteit

### Stap 5: MCP Client Integration
- **MCP Server**: Standalone tools server met alle 3 tools (BTW, korting, CBS API)
- **Client**: Verbinding vanuit elke notebook
- **Transport**: Server-Sent Events voor real-time communicatie
- **Tools**: Alle tools via MCP protocol (BTW, korting, CBS API)

## MCP Server Specificatie
```python
# mcp_server/tools_server.py
- SSE transport layer
- 3 tools: BTW berekening, korting berekening, CBS API
- JSON-RPC interface voor alle tools
- Lokale server op port 8000
- Tools georganiseerd in aparte modules:
  - tools/btw_tool.py
  - tools/discount_tool.py  
  - tools/cbs_api_tool.py
```

## Dependencies
```
# Notebooks
anthropic
openai  
pydantic-ai
langchain
langchain-anthropic
requests
pandas

# MCP Server
mcp
fastapi
uvicorn
httpx
```

## Demo Use Case
AI agent die:
1. Basis conversatie kan voeren (claude-3-haiku-20240307)
2. Tools kan gebruiken voor berekeningen (BTW, korting)
3. CBS data kan ophalen over Nederlandse gemeenten
4. Conversatie historie bijhoudt
5. Via MCP protocol kan communiceren met alle externe tools

## MCP Tools Overzicht
De MCP server biedt alle 3 tools aan:
1. **BTW Tool**: Bereken BTW over bedragen met verschillende tarieven
2. **Korting Tool**: Bereken kortingen en eindprijzen
3. **CBS API Tool**: Haal Nederlandse gemeentedata op via CBS StatLine API

Dit maakt het mogelijk om via MCP protocol dezelfde functionaliteit te benaderen als in de notebooks, maar dan als externe services.

## Model Configuratie
- **Model**: claude-3-haiku-20240307 (goedkoop en snel)
- **API**: Anthropic Claude API
- **Environment**: Lokale development setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elbrink)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/elbrink)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
