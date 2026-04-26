## the-trading-floor-v2

> A multi-agent system where specialized AI agents collaborate and debate on market analysis in real-time, simulating a hedge fund research team decision-making process.

# The Trading Floor — Multi-Agent Market Analysis Council

A multi-agent system where specialized AI agents collaborate and debate on market analysis in real-time, simulating a hedge fund research team decision-making process.

## Project Overview

This project creates a visual council of AI agents, each with distinct expertise, that analyze financial instruments and produce consensus-based trading recommendations. Agents engage in structured debate, challenge each other's assumptions, and synthesize diverse perspectives into actionable insights.

## Agent Framework

This project uses **CrewAI** as the primary agent orchestration framework.

### Why CrewAI

- Native support for role-based agent specialization with distinct backstories and goals
- Built-in task delegation and agent collaboration patterns
- Hierarchical process mode enables a "manager" agent to coordinate specialist agents
- Sequential and consensual process modes for structured debate workflows
- Strong integration with LangChain tools for extensibility
- Active community with extensive documentation available via Context7

### Alternative Consideration

LangGraph could be used if more granular control over agent state transitions and cyclic workflows is needed. However, CrewAI's higher-level abstractions better match the council/debate paradigm of this project.

## Technology Stack

### Backend (Python)

- **CrewAI** — Agent orchestration and multi-agent workflows
- **yfinance** — Market data retrieval (prices, fundamentals, historical data)
- **FastAPI** — REST API and WebSocket server for real-time streaming
- **Pydantic** — Data validation and structured output schemas
- **LangChain** — Tool creation and LLM integrations
- **Anthropic SDK** — Claude as the underlying LLM for all agents

### Frontend (Next.js)

- **Next.js 14+** — App router with server components
- **TypeScript** — Strict mode enabled throughout
- **Tailwind CSS** — Styling and responsive design
- **Framer Motion** — Agent conversation animations and transitions
- **Server-Sent Events (SSE)** — Real-time streaming of agent deliberations

### Infrastructure

- **Python 3.11+** — Backend runtime
- **Bun** — Frontend package management and runtime

## Project Structure

```
trading-floor/
├── backend/
│   ├── agents/           # Individual agent definitions
│   ├── crews/            # Crew configurations and workflows
│   ├── tools/            # Custom tools for market data and analysis
│   ├── schemas/          # Pydantic models for inputs/outputs
│   ├── api/              # FastAPI routes and WebSocket handlers
│   └── main.py           # Application entry point
├── frontend/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components for agent UI
│   ├── hooks/            # Custom hooks for SSE and state
│   ├── lib/              # Utility functions and API clients
│   └── types/            # TypeScript type definitions
├── shared/
│   └── types/            # Shared type definitions between frontend/backend
├── pyproject.toml
├── package.json
└── README.md
```

## Agent Definitions

### The Council Members

Each agent must be defined with a clear role, goal, and backstory in CrewAI. The backstory shapes the agent's perspective and analytical bias.

#### 1. Quant Analyst

- **Focus**: Technical analysis, price patterns, statistical indicators
- **Data Sources**: Historical prices, volume, moving averages, RSI, MACD, Bollinger Bands
- **Perspective**: Data-driven, skeptical of narratives, focused on quantifiable signals
- **Output**: Technical score with key indicator readings and pattern identification

#### 2. Sentiment Scout

- **Focus**: Market sentiment, news flow, social metrics
- **Data Sources**: News headlines (via web search tools), fear/greed proxy indicators, volume anomalies
- **Perspective**: Contrarian-aware, understands crowd psychology, tracks narrative shifts
- **Output**: Sentiment score with supporting evidence and narrative summary

#### 3. Macro Strategist

- **Focus**: Macroeconomic context, sector dynamics, intermarket relationships
- **Data Sources**: Index performance, sector ETFs, yield data, currency movements
- **Perspective**: Top-down thinker, connects micro to macro, identifies regime changes
- **Output**: Macro alignment score with sector and economic context

#### 4. Risk Manager

- **Focus**: Position sizing, correlation risks, downside scenarios
- **Data Sources**: Volatility metrics, historical drawdowns, correlation matrices
- **Perspective**: Conservative, focused on capital preservation, stress-tests assumptions
- **Output**: Risk assessment with position size recommendation and key risks

#### 5. Portfolio Chief

- **Focus**: Synthesis and final decision-making
- **Data Sources**: All other agent outputs
- **Perspective**: Balanced, resolves conflicts, weighs evidence quality
- **Output**: Final recommendation with confidence level and dissenting opinions noted

## Workflow Design

### Primary Workflow: Consensus Analysis

1. **Input Reception**: User submits a ticker symbol and optional analysis context
2. **Data Gathering Phase**: All specialist agents independently gather relevant data using their assigned tools
3. **Initial Analysis Phase**: Each specialist produces their individual assessment
4. **Debate Phase**: Agents review each other's analyses and raise objections or supporting points
5. **Synthesis Phase**: Portfolio Chief integrates all perspectives and objections
6. **Output Generation**: Structured final report with individual scores, consensus view, and key disagreements

### Process Mode

Use CrewAI's **hierarchical process** with the Portfolio Chief as the manager agent. This enables:

- Specialist agents to be delegated specific analysis tasks
- The manager to request clarification or additional analysis
- Natural conflict resolution through the manager's synthesis role

## Tool Definitions

### Market Data Tools

Create custom CrewAI tools wrapping yfinance functionality:

- **Price History Tool**: Retrieves OHLCV data for specified timeframes
- **Fundamentals Tool**: Fetches key financial metrics and ratios
- **Technical Indicators Tool**: Calculates common technical indicators
- **Sector Performance Tool**: Compares performance across sectors and indices

### Analysis Tools

- **Correlation Calculator**: Computes correlation between assets
- **Volatility Analyzer**: Calculates historical and implied volatility metrics
- **Drawdown Analyzer**: Identifies historical drawdown periods and recovery times

### External Tools

- **Web Search Tool**: Use Exa/Firecrawl MCP for news and sentiment research
- **Documentation Lookup**: Use Context7 MCP for library documentation when needed

## Output Schemas

### Individual Agent Output

Each specialist agent must return a structured output containing:

- Numerical score (0-100 scale)
- Confidence level (low/medium/high)
- Key supporting evidence (list of specific data points)
- Primary concerns or caveats
- Timestamp of analysis

### Consensus Report Output

The final report must contain:

- Ticker and analysis timestamp
- Overall recommendation (strong buy/buy/hold/sell/strong sell)
- Consensus confidence level
- Individual agent scores with brief rationale
- Key agreements across agents
- Notable disagreements and how they were resolved
- Suggested position size as percentage of portfolio
- Key risk factors and invalidation criteria

## Frontend Requirements

### Main Dashboard View

- Ticker input with autocomplete
- Real-time streaming display of agent deliberations
- Visual representation of each agent with their current status
- Animated message bubbles showing agent thoughts and debates

### Agent Conversation Panel

- Chronological display of agent messages
- Visual differentiation between agent types (color coding, avatars)
- Highlight disagreements and debates with distinct styling
- Show when agents are "thinking" with loading indicators

### Results Panel

- Summary card with final recommendation
- Individual agent score cards with expandable details
- Visual confidence meter
- Historical analysis log for the session

### Streaming Implementation

- Use Server-Sent Events from FastAPI backend
- Frontend subscribes to analysis stream on submission
- Each agent message is a discrete SSE event with agent ID and content
- Final report is a terminal event that closes the stream

## API Design

### REST Endpoints

- `POST /analyze` — Submit ticker for analysis, returns stream URL
- `GET /history` — Retrieve past analyses
- `GET /agents` — List available agents and their descriptions

### WebSocket/SSE Endpoints

- `GET /stream/{analysis_id}` — SSE stream for real-time agent deliberations

### Request/Response Contracts

All API contracts should be defined using Pydantic models in the backend and corresponding TypeScript types in the frontend. Ensure strict typing throughout.

## Development Guidelines

### Backend Development

- Define each agent in its own module under `backend/agents/`
- Keep tool definitions separate from agent definitions
- Use Pydantic for all data validation and serialization
- Implement proper error handling for yfinance API failures
- Add retry logic for LLM calls with exponential backoff
- Log all agent interactions for debugging and analysis

### Frontend Development

- Use React Server Components where possible
- Client components only for interactive elements and streaming
- Implement proper loading and error states for all async operations
- Use TypeScript strict mode with no implicit any
- Create reusable components for agent message display

### Agent Prompt Engineering

- Each agent's system prompt should strongly emphasize their specific expertise and perspective
- Include examples of the type of analysis expected in the backstory
- Define clear boundaries for what each agent should and should not comment on
- Instruct agents to be concise and cite specific data points

## Testing Strategy

- Unit tests for individual tools (yfinance wrappers, calculations)
- Integration tests for agent workflows with mocked LLM responses
- End-to-end tests for the full analysis pipeline
- Frontend component tests with React Testing Library

## Deployment Considerations

- Backend and frontend can be deployed separately
- Use environment variables for API keys and configuration
- Implement rate limiting on the analyze endpoint
- Consider caching yfinance responses to reduce API calls
- Monitor token usage across agent interactions

## Future Enhancements

- Add memory persistence so agents can reference past analyses
- Implement agent specialization fine-tuning based on accuracy tracking
- Add support for portfolio-level analysis (multiple tickers)
- Create alerting system for significant market condition changes
- Add backtesting mode to evaluate historical recommendation accuracy

---
> Source: [CloudAI-X/the-trading-floor-v2](https://github.com/CloudAI-X/the-trading-floor-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
