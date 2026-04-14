## director-ai

> During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

# Instructions

During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

You should also use the `.cursorrules` file as a scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## Screenshot Verification
The screenshot verification workflow allows you to capture screenshots of web pages and verify their appearance using LLMs. The following tools are available:

1. Screenshot Capture:
```bash
venv/bin/python tools/screenshot_utils.py URL [--output OUTPUT] [--width WIDTH] [--height HEIGHT]
```

2. LLM Verification with Images:
```bash
venv/bin/python tools/llm_api.py --prompt "Your verification question" --provider {openai|anthropic} --image path/to/screenshot.png
```

Example workflow:
```python
from screenshot_utils import take_screenshot_sync
from llm_api import query_llm

# Take a screenshot
screenshot_path = take_screenshot_sync('https://example.com', 'screenshot.png')

# Verify with LLM
response = query_llm(
    "What is the background color and title of this webpage?",
    provider="openai",  # or "anthropic"
    image_path=screenshot_path
)
print(response)
```

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```
venv/bin/python ./tools/llm_api.py --prompt "What is the capital of France?" --provider "anthropic"
```

The LLM API supports multiple providers:
- OpenAI (default, model: gpt-4o)
- Azure OpenAI (model: configured via AZURE_OPENAI_MODEL_DEPLOYMENT in .env file, defaults to gpt-4o-ms)
- DeepSeek (model: deepseek-chat)
- Anthropic (model: claude-3-sonnet-20240229)
- Gemini (model: gemini-pro)
- Local LLM (model: Qwen/Qwen2.5-32B-Instruct-AWQ)

But usually it's a better idea to check the content of the file and use the APIs in the `tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the `tools/web_scraper.py` file to scrape the web.
```
venv/bin/python ./tools/web_scraper.py --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the `tools/search_engine.py` file to search the web.
```
venv/bin/python ./tools/search_engine.py "your search keywords"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python venv in ./venv. Use it.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Due to Cursor's limit, when you use `git` and `gh` and need to submit a multiline commit message, first write the message in a file, and then use `git commit -F <filename>` or similar command to commit. And then remove the file. Include "[Cursor] " in the commit message and PR title.

## Cursor learned

- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes
- Use 'gpt-4o' as the model name for OpenAI's GPT-4 with vision capabilities
- Always check regex match() results for null before accessing array indices - use optional chaining or explicit null checks
- Use 'let' instead of 'const' for variables that need reassignment, especially in conditional blocks
- When extracting URLs, use comprehensive regex patterns and provide fallback mechanisms for edge cases
- Successfully patched @modelcontextprotocol/sdk to extend DEFAULT_REQUEST_TIMEOUT_MSEC from 60000ms to 6000000ms using patch-package
- patch-package with postinstall scripts is the reliable method for maintaining npm package patches across installs

## Agent Creation Process (agent-server/)

### Understanding Agent Registration System
The agent-server uses a factory pattern with registry for managing agents:

1. **Base Agent Structure**: All agents extend `BaseAgent` class from `src/types/agent.ts`
2. **Agent Factory**: Registry system in `src/core/agent-factory.ts` manages agent creation
3. **Registration**: Agents are registered in `src/index.ts` in `initializeAgents()` function

### Agent Creation Steps

#### Step 1: Create Agent Class
Create new agent in `src/agents/main/[agent-name].ts`:
```typescript
import type { AgentConfig, AgentRequest, AgentResponse, AgentExecutionContext } from "../../types/agent.js";
import { BaseAgent } from "../../types/agent.js";

export class YourAgentClass extends BaseAgent {
  constructor() {
    const config: AgentConfig = {
      id: "your_agent_id", // Must be unique
      name: "Your Agent Name",
      description: "Agent description",
      version: "1.0.0",
      capabilities: ["capability1", "capability2"],
      metadata: { /* agent-specific config */ },
    };
    super(config);
  }

  async execute(request: AgentRequest, context: AgentExecutionContext): Promise<AgentResponse> {
    // Implementation logic
  }

  async healthCheck(): Promise<boolean> {
    // Health check logic
  }
}

export const createYourAgent = (): BaseAgent => new YourAgentClass();
```

#### Step 2: Register Agent in Main Index
Update `src/index.ts`:
1. Import the factory function
2. Register in `initializeAgents()` function
3. Create default instance

#### Step 3: Reusable Utilities
- Use `src/utils/gemini-util.ts` for Gemini AI interactions
- Create shared utilities in `src/utils/` for common functionality
- Follow singleton pattern for stateless utilities

### Key Patterns
- **Payload Handling**: Agents receive payload as string (like web scraper)
- **Error Handling**: Always return structured AgentResponse with success/error
- **Metadata**: Include processing time, content length, request IDs
- **Health Checks**: Implement meaningful health checks for external dependencies
- **Factory Pattern**: Export factory function for registry registration

### SEO Optimization Agent Example
- Created `seo-optimization-agent.ts` following the pattern
- Uses `geminiUtil` for AI-powered SEO analysis
- Takes same payload format as web scraper (string input)
- Provides comprehensive SEO recommendations with structured output

### GitHub Agent Example
- Created `github-agent.ts` following the pattern
- **Simplified to SEO-only agent**: Takes string payload containing SEO context with embedded GitHub repo URL
- **Payload Format**: String input (no JSON parsing) containing SEO context and repository URL
- **Core Functionality**:
  - Extracts repository URL from context using regex patterns
  - Uses Gemini AI to generate complete SEO-optimized HTML from context
  - Creates unique branch names with format: `director-ai-{UUID}`
  - Commits optimized HTML to new branch and creates PR
  - Returns simple "Action completed check on github" message
- **Repository URL Detection**: Supports both `github.com/owner/repo` and direct `owner/repo` patterns
- **AI-Powered HTML Generation**: Creates complete HTML5 documents with comprehensive SEO elements
- Uses Octokit for GitHub API interactions and Gemini for AI-powered HTML generation

### Dependencies
- `@google/genai` for Gemini AI integration
- `@octokit/rest` for GitHub API operations
- `dotenv` for environment variable management
- Ensure GOOGLE_API_KEY and GITHUB_TOKEN are configured in environment

# Scratchpad

## Current Task: Landing Page Agent Cards Optimization ✅ COMPLETED

### Task Description
Optimize the live agents section for better visual presentation and readability:
- Expand container width to better align with content above
- Increase card width to prevent text overflow
- Format agent names by replacing underscores with spaces
- Increase heading text sizes for better hierarchy

### Task Progress
[X] Expand container width with responsive max-widths (max-w-4xl lg:max-w-5xl xl:max-w-6xl)
[X] Optimize grid layout to show fewer columns per row for wider cards
[X] Increase card padding from p-6 to p-7 for better text spacing
[X] Format agent names using name.split('_').join(' ') for readability
[X] Increase "Live Agents" heading from text-2xl to text-3xl
[X] Increase "Live Use Cases" heading from text-sm to text-base

### Key Improvements Made
- **Container Alignment**:
  - Expanded container width: max-w-4xl lg:max-w-5xl xl:max-w-6xl
  - Better alignment with content sections above
  - More horizontal space for wider agent cards
- **Grid Layout Optimization**:
  - Changed from lg:grid-cols-3 to lg:grid-cols-2 xl:grid-cols-3
  - Increased gap from gap-6 to gap-8 for better spacing
  - Cards are now wider on large screens, preventing text overflow
- **Typography Enhancements**:
  - "Live Agents": text-2xl → text-3xl (more prominent)
  - "Live Use Cases": text-sm → text-base (better readability)
  - Agent names formatted: underscores replaced with spaces
- **Card Spacing**:
  - Increased padding from p-6 to p-7 across all cards
  - Better text spacing and reduced overflow risk
  - Applied to both skeleton and actual agent cards

### Results
- ✅ Agent cards now properly aligned with content above
- ✅ Eliminated text overflow issues with wider cards
- ✅ Improved agent name readability (spaces instead of underscores)
- ✅ Enhanced visual hierarchy with larger headings
- ✅ Better overall spacing and professional appearance
- ✅ Maintained responsive behavior across all screen sizes

## Previous Task: Enhanced Live Agents Display ✅ COMPLETED

### Summary of Previous Enhancements
- Successfully displays ALL available agents on landing page
- Smooth, professional expand/collapse functionality
- No overflow issues across mobile, tablet, and desktop
- Enhanced UX with staggered animations and micro-interactions
- Smart button text showing exact count of additional agents
- Maintained consistent dark theme and brand aesthetics
- Improved loading states with detailed skeleton animations

## Previous Task: Landing Page Layout Separation ✅ COMPLETED

### Summary of Previous Layout Changes
- Text content now stays clearly on the left side
- Lottie animation positioned far right with minimal text overlap
- Better responsive behavior across screen sizes
- Clean visual separation while maintaining design aesthetics
- Proper overflow handling for extended animation positioning

## Previous Task: Landing Page UI/UX Improvements ✅ COMPLETED

### Summary of Previous Improvements
- Enhanced typography to modern landing page scales (text-8xl headings)
- Improved button styling with larger padding and hover effects
- Fixed Lottie animation whitespace issues with absolute positioning
- Maintained excellent color scheme throughout

## Previous Task: Dynamic Blockchain Chain Configuration ✅ COMPLETED

**Summary:** Successfully implemented dynamic blockchain chain configuration across all three applications (backend, agent-server, frontend). The system now supports switching between base-sepolia and polygon-amoy networks using environment variables.

**Key Features Implemented:**
- Environment-based chain selection with validation
- Dynamic network configuration for wallet connections
- Proper error handling and fallback to default values
- Consistent chain usage across payment processing, agent handling, and UI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AkhileshManda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
