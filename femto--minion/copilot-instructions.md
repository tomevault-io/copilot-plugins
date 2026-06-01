## minion

> when creating BaseAgent or CodeAgent,

# Agent Development Guidelines
when creating BaseAgent or CodeAgent,
Please use .create cls method instead of constructor,
# Create CodeAgent with budget-related tools
CodeAgent.create(brain=brain,tools=[GetTeamMembersTool(),GetExpensesTool(),GetBudgetByLevelTool()])

instead of
    agent = CodeAgent(brain=brain)
    agent.add_tool(GetTeamMembersTool())
    agent.add_tool(GetExpensesTool())
    agent.add_tool(GetBudgetByLevelTool())
    await agent.setup()  # Initialize the agent

since .setup() is handled in .create

# Note

---
> Source: [femto/minion](https://github.com/femto/minion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
