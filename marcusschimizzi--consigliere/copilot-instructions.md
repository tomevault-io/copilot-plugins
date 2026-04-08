## consigliere

> This guide will walk you through setting up a multi-agent MCP server that provides specialized AI agents (planner, reviewer, architect) to work with Claude Code.

# Multi-Agent Gemini MCP Server Setup Guide

This guide will walk you through setting up a multi-agent MCP server that provides specialized AI agents (planner, reviewer, architect) to work with Claude Code.

## Prerequisites

- Node.js 18+ installed
- Claude Code desktop application
- Google AI Studio API key
- Git (for cloning repositories)

## Step 1: Get Your Gemini API Key

1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Sign in with your Google account
3. Navigate to "API keys" in the left menu
4. Click "Create API key"
5. Copy and save the key securely

## Step 2: Set Up Consigliere

### Create the project directory
```bash
mkdir consigliere
cd consigliere
```

### Create the files
1. Save the server code as `server.js`
2. Save the package.json configuration
3. Install dependencies:
```bash
npm install
```

### Test the server
```bash
GEMINI_API_KEY=your_api_key_here npm start
```

## Step 3: Configure Claude Desktop

### Find your Claude config file:
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

### Add the MCP server configuration:
```json
{
  "mcpServers": {
    "consigliere": {
      "command": [
        "node",
        "/absolute/path/to/your/consigliere/server.js"
      ],
      "env": {
        "GEMINI_API_KEY": "your_actual_api_key_here"
      }
    }
  }
}
```

**Important**: Replace `/absolute/path/to/your/consigliere/server.js` with the actual full path to your server.js file.

### Restart Claude Desktop
Completely quit and restart the Claude desktop application for changes to take effect.

## Step 4: Verify Installation

In Claude Code, try one of these commands:

```
plan_project: "Create a REST API for a todo list application"
```

```
review_code: Check this function for issues:
def calculate_total(items):
    total = 0
    for item in items:
        total += item.price
    return total
```

```
design_architecture: "Design a system for real-time chat with 10k concurrent users"
```

## Available Tools

### 1. `plan_project`
**Purpose**: Get project planning and task breakdown
**Usage**: 
```
plan_project: "Build a social media dashboard with React and Node.js"
```

### 2. `review_code` 
**Purpose**: Comprehensive code review
**Usage**:
```
review_code: [paste your code here]
```

### 3. `design_architecture`
**Purpose**: System architecture guidance
**Usage**:
```
design_architecture: "E-commerce platform handling 1M+ products"
```

### 4. `consult_agent`
**Purpose**: Direct consultation with specific agents
**Usage**:
```
consult_agent: agent=planner, prompt="How should I structure a microservices project?"
```

## Workflow Examples

### Starting a New Project
1. **Plan**: `plan_project: "Build a task management app with team collaboration"`
2. **Architect**: `design_architecture: "Task management with real-time updates, 1000 users"`
3. **Code**: Use Claude Code to implement
4. **Review**: `review_code: [your implementation]`

### Code Review Workflow
1. Write your code in Claude Code
2. Use `review_code` to get comprehensive feedback
3. Address the suggested improvements
4. Re-review if needed for complex changes

### Architecture Decisions
1. Use `design_architecture` for system design questions
2. Use `consult_agent: agent=architect` for specific technical decisions
3. Use `plan_project` to break down implementation into phases

## Troubleshooting

### Server won't start
- Check that Node.js 18+ is installed: `node --version`
- Verify your API key is valid
- Check the console for error messages

### Claude can't find the server
- Verify the absolute path in your config is correct
- Make sure you restarted Claude Desktop after config changes
- Check that the server.js file is executable

### API errors
- Ensure your Gemini API key is valid and has quota
- Check your internet connection
- Verify the API key is correctly set in the environment

## Advanced Usage

### Custom Agent Prompts
You can modify the system prompts in the `AGENTS` configuration to customize each agent's behavior and expertise areas.

### Adding New Agents
To add new specialized agents:
1. Add a new entry to the `AGENTS` object
2. Add corresponding tools in the `setupToolHandlers` method
3. Implement the handler methods

### Integration with Claude Code Workflows
- Use the planner at project start
- Consult the reviewer before commits
- Check with the architect for complex design decisions
- Use `consult_agent` for ad-hoc expert advice

## Next Steps

Once you have Consigliere working:
1. Experiment with different types of projects
2. Customize the agent prompts for your specific needs
3. Consider adding domain-specific advisors (DevOps, Security, etc.)
4. Build automated workflows that chain multiple advisors together

The goal is to create a collaborative AI development environment where each advisor contributes their specialized expertise to your coding workflow!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcusschimizzi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcusschimizzi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
