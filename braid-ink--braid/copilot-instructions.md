## braid

> This file provides essential guidance for Claude when working on Braid projects, focusing on creating custom LangGraph agents using our built-in toolkit and integrations.

# Claude Assistant Guide for Braid Development

This file provides essential guidance for Claude when working on Braid projects, focusing on creating custom LangGraph agents using our built-in toolkit and integrations.

## 🎯 **STRUCTURED AGENT DEVELOPMENT PROCESS**

**When users request agent development, follow the [Agent Development Roadmap](CLAUDE_AGENT_DEVELOPMENT_ROADMAP.md) for consistent, high-quality results.**

### **Trigger Commands**
- **"Please prepare to create a LangGraph agent by reviewing the docs and examples"** → Execute Phase 1: Discovery & Planning
- **"Optional Pro Pack and production deployment"** → Execute Phase 2: Production Enhancement

These commands ensure systematic review of all critical documentation, proper template selection, and adherence to production best practices.

## 🚀 Creating New Agents with Direct API Integrations

**Primary Approach**: Use direct API integrations from `core/integrations/` for all new agents. This provides simple, reliable access to external services.

When a user requests a new agent, follow this recommended approach:

### 1. Use Direct API Integrations

**Primary Approach**: Use direct API integrations from `core/integrations/` for all new agents.

```bash
# Available direct integrations
core/integrations/
├── xero/tools.py         # Direct Xero API integration
├── notion/tools.py       # Direct Notion API integration  
├── perplexity/tools.py   # Direct Perplexity API integration
├── gworkspace/tools.py   # Google Workspace integration
└── slack/tools.py        # Slack integration
```

### 2. Standard Direct Integration Process

1. **Use Existing Integrations**: Import tools from `core/integrations/`
2. **Configure Environment**: Set API keys in `.env` file
3. **Import and Use**: Import tools directly in your agent code
4. **Test Integration**: Test API calls with direct tool invocation
5. **Handle Errors**: Add graceful fallbacks for API failures

### 3. Available Direct Integrations

| Integration | Tools | Use Cases |
|-------------|-------|-----------|
| **Xero** | `get_xero_profit_and_loss`, `get_xero_balance_sheet`, `get_xero_trial_balance` | Financial data, accounting reports |
| **Notion** | `create_notion_page`, `get_notion_page`, `update_notion_page` | Documentation, report generation |
| **Perplexity** | `perplexity_search`, `perplexity_market_research`, `perplexity_ask` | Real-time web research, market insights |
| **Google Workspace** | Gmail, Calendar, Sheets tools | Email, scheduling, data management |
| **Slack** | Message, channel, user tools | Team communication, notifications |

### 4. Example Direct Integration Usage

```python
# Import direct API tools
from core.integrations.xero.tools import get_xero_profit_and_loss
from core.integrations.notion.tools import create_notion_page
from core.integrations.perplexity.tools import perplexity_market_research

# Use tools directly in your agent
@tool
def get_financial_data(report_type: str = "profit_and_loss") -> str:
    """Get financial data from Xero using direct API."""
    try:
        if report_type == "profit_and_loss":
            result = get_xero_profit_and_loss.invoke({})
        # Handle result and return formatted data
        return result
    except Exception as e:
        return f"Error: {e}"
```

## 📋 Agent Development Checklist

When creating any new agent, verify:

- [ ] Use direct API integrations from `core/integrations/`
- [ ] Configure required environment variables (API keys)
- [ ] Test direct API calls before coding
- [ ] Follow existing patterns in agent examples
- [ ] Include error handling and graceful fallbacks
- [ ] Add appropriate logging
- [ ] Create tests for your agent
- [ ] Document any new direct integrations

## 🔧 Direct Integration Patterns

### Standard Direct API Setup
```python
# Import direct integrations
from core.integrations.xero.tools import get_xero_profit_and_loss
from core.integrations.notion.tools import create_notion_page
from core.integrations.perplexity.tools import perplexity_market_research

# Use in your agent tools
tools = [
    get_xero_profit_and_loss,
    create_notion_page,
    perplexity_market_research
]
```

### Adding New Direct Integrations
```python
# Create new integration file: core/integrations/newservice/tools.py
@tool("new_service_action", args_schema=NewServiceInput)
def new_service_action(param: str) -> str:
    """Tool description for LLM."""
    api_key = os.environ.get("NEW_SERVICE_API_KEY")
    if not api_key:
        return "Error: NEW_SERVICE_API_KEY not set"
    
    # Direct API call
    response = requests.post(
        "https://api.newservice.com/action",
        headers={"Authorization": f"Bearer {api_key}"},
        json={"param": param}
    )
    return response.json()
    )
```

## 📁 File Structure Reference

### Core Integration Files
```
braid/
├── core/integrations/              # Direct API integrations (USE THIS)
│   ├── xero/tools.py              # Xero financial data tools
│   ├── notion/tools.py            # Notion documentation tools
│   ├── perplexity/tools.py        # Perplexity research tools
│   ├── gworkspace/tools.py        # Google Workspace tools
│   └── slack/tools.py             # Slack communication tools
├── agents/                        # Example agent implementations
├── templates/                     # Agent templates and starter code
└── CLAUDE.md                      # This file
```

### When Creating Financial Agents
- Use existing agent examples in `agents/` as reference
- Import tools from `core/integrations/xero/tools.py`
- Reference direct Xero API integration examples

### When Creating Research Agents  
- Use `core/integrations/perplexity/tools.py` for web research
- Use `core/integrations/notion/tools.py` for report generation

## 🚨 Important Notes

### Always Use Direct Integrations
```python
# Use direct integrations:
from core.integrations.xero.tools import get_xero_profit_and_loss
from core.integrations.notion.tools import create_notion_page
from core.integrations.perplexity.tools import perplexity_search
```

### Direct Integration Dependencies
- **Python only** - no Node.js required
- **Environment variables** for API keys
- **Direct HTTP calls** to service APIs

### Testing Requirements
- Test direct API calls with `.invoke({})` method
- Use actual API keys for integration testing
- Test error handling and fallbacks

### Deployment Considerations
- Simple Python environment requirements
- Standard LangGraph deployment patterns
- Follow security best practices for API keys

## 🔄 Workflow Summary

1. **Import** → Direct integrations from `core/integrations/`
2. **Configure** → API keys in environment variables  
3. **Test** → Direct API calls work properly
4. **Develop** → Follow existing agent patterns in `agents/` and `templates/`
5. **Deploy** → Standard Python deployment

## 📞 Quick Reference Commands

```bash
# Test direct integrations
python -c "from core.integrations.xero.tools import get_xero_profit_and_loss; print(get_xero_profit_and_loss.invoke({}))"

# Test specific agent
python agents/your-agent/agent.py

# Create new agent from template
braid create my-new-agent --template react-agent
```

## 🚀 Getting Started

### Quick Agent Creation
```bash
# Create a new agent with financial capabilities
braid create my-financial-agent --template production-financial-agent

# Create a research agent
braid create my-research-agent --template react-agent

# Create a memory-enabled agent
braid create my-memory-agent --template memory-agent
```

### Available Templates
- `react-agent` - Basic ReAct pattern agent
- `memory-agent` - Agent with conversation memory
- `production-financial-agent` - Full financial analysis capabilities
- `data-enrichment` - Data processing and enrichment
- `retrieval-agent-template` - RAG-enabled agent

---

**Remember**: Use direct API integrations from `core/integrations/` for all new agents. Focus on simple, reliable LangGraph patterns with our built-in toolkit.

---
> Source: [braid-ink/braid](https://github.com/braid-ink/braid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
