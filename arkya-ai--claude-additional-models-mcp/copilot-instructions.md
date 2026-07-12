## claude-additional-models-mcp

> This project implements a Model Context Protocol (MCP) server that integrates Google Gemini 3.x capabilities directly into the Claude Desktop environment. The primary objective is to create a high-efficiency hybrid intelligence system where Claude acts as the primary orchestrator and Gemini serves as the high-volume execution engine.

# CLAUDE.md - Gemini MCP Integration Project Memory

## Project Overview
This project implements a Model Context Protocol (MCP) server that integrates Google Gemini 3.x capabilities directly into the Claude Desktop environment. The primary objective is to create a high-efficiency hybrid intelligence system where Claude acts as the primary orchestrator and Gemini serves as the high-volume execution engine.

### Core Value Proposition
- **10x Reduction in Claude Consumption:** By offloading heavy lifting (long-form writing, large-scale analysis, code generation) to Gemini, Claude's token usage is minimized.
- **90%+ Opus Savings:** High-cost Claude Opus cycles are reserved strictly for complex orchestration and final synthesis, while Gemini handles the bulk of the data processing.
- **Continuous Availability:** Leverages Gemini's high rate limits and cost-effective tiers to ensure the agentic workflow remains responsive even during high-load periods.

### What This Is Not
- **Not a Standalone Chatbot:** This server is designed to be used *by* Claude, not as a replacement for the Claude interface.
- **Not a Bypass for Safety Filters:** All Gemini safety settings are active; this tool does not circumvent provider-level content restrictions.
- **Not a Persistent Database:** While it processes large amounts of data, it does not provide long-term vector storage or RAG memory outside of the current session context.
- **Not a Direct File System Access Tool:** Gemini tools return data to Claude; Claude remains the only entity authorized to write to the local filesystem.

### Quick Start
1. **Prerequisites:** Node.js v18+, Google AI Studio API Key.
2. **Installation:** Clone the repository and run `npm install`.
3. **Build:** Run `npm run build` to generate the distribution files.
4. **Configuration:** Add the server to your `claude_desktop_config.json` (see Configuration section).
5. **Verification:** Open Claude Desktop and look for the Gemini MCP tools. Test with: "Use Gemini to summarize the latest news on MCP."

### Key Metrics
- **Target Token Reduction:** 10:1 ratio (Gemini tokens to Claude tokens).
- **Efficiency Goal:** 90% reduction in Claude-native processing for tasks exceeding 500 words or 100 lines of code.
- **Reliability:** 99% tool call success rate through robust error handling and fallback logic.

---

## Architecture
The integration utilizes a dual-path communication strategy to maximize feature availability while maintaining compatibility.

### Data Flow
1. **User Input:** User provides a prompt to Claude Desktop.
2. **Orchestration:** Claude determines if the task requires high-volume processing or external data.
3. **Tool Call:** Claude sends a JSON-RPC request to the MCP Server.
4. **API Routing:** The MCP Server routes the request to either the OpenAI SDK path or the Native Gemini API path.
5. **Execution:** Gemini processes the request (optionally using Google Search grounding).
6. **Return Path:** Gemini Response -> MCP Server -> Claude Desktop.
7. **Final Synthesis:** Claude reviews the Gemini output and presents it to the user or performs file operations.

### Hybrid Communication Approach
1. **OpenAI SDK Path:** Used for standard chat-based tools (`ask_gemini`, `ask_gemini_pro`). This provides a stable, standardized interface for text-to-text operations and ensures compatibility with standard streaming protocols.
2. **Native Gemini API Path:** Used for advanced features including `google_search` grounding, `url_context`, and multi-modal document processing. This path bypasses the OpenAI compatibility layer to access Gemini-specific parameters like `dynamic_retrieval_config`.

### OpenAI SDK vs. Native API Tradeoffs
- **OpenAI SDK:**
    - *Pros:* Simpler error handling, standardized response format, easier to swap models.
    - *Cons:* No access to Google Search grounding, limited multi-modal support.
- **Native API:**
    - *Pros:* Full access to Gemini 3.x features, grounding, system instructions, and safety settings.
    - *Cons:* More complex request/response structure, requires specific SDK management.

### Model Specifications
- **gemini-3-flash-preview:**
    - **Role:** General purpose, high-speed execution.
    - **Default Output Tokens:** 8,192.
    - **Use Case:** Summarization, basic research, drafting, and sub-agent tasks.
- **gemini-3-pro-preview:**
    - **Role:** Complex reasoning and heavy code generation.
    - **Default Output Tokens:** 16,384.
    - **Use Case:** Architectural design, complex debugging, deep technical analysis.

### Core Toolset
1. `ask_gemini`: General text generation and reasoning via Flash.
2. `ask_gemini_pro`: High-complexity reasoning and code generation via Pro.
3. `web_search`: Real-time information retrieval using Google Search grounding.
4. `web_reader`: High-fidelity content extraction from specific URLs.
5. `parse_document`: Advanced OCR and structured data extraction from PDFs and images.

### Configuration
To enable the server, add the following to your `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "gemini": {
      "command": "node",
      "args": ["/path/to/gemini-mcp/dist/index.js"],
      "env": {
        "GOOGLE_API_KEY": "your-api-key-here"
      }
    }
  }
}
```
Environment variables:
- `GOOGLE_API_KEY` (required): Your Google AI Studio API key. Generate one at https://aistudio.google.com/apikey.
- `GEMINI_MODEL_FLASH` (optional): Override the Flash model identifier. Defaults to `gemini-3-flash-preview`.
- `GEMINI_MODEL_PRO` (optional): Override the Pro model identifier. Defaults to `gemini-3-pro-preview`.

---

## Gemini Delegation Gate (MANDATORY)
Claude must evaluate every task against this gate before performing the work itself. This applies to Claude (parent) AND to any sub-agents spawned during task execution.

### STOP Checklist
1. **Code Volume:** Will the output exceed 50 lines of code? -> **MUST** use `ask_gemini_pro`.
2. **Analysis Volume:** Will the analysis exceed 300 words? -> **MUST** use `ask_gemini`.
3. **Sub-Agent Requirement:** If the task requires multiple steps, sub-agents **MUST** be instructed to use Gemini tools for their internal processing.
4. **Pattern:** Gemini generates the content -> Claude reviews and writes to the local filesystem.

### Execution Priority
1. **FIRST PRIORITY:** Spawn parallel sub-agents. Each sub-agent is assigned a specific segment of the task and must use Gemini tools for execution. This is the fastest path and must always be preferred when work can be parallelized.
2. **SECOND PRIORITY:** Direct delegation. If the task is a single unit of work (e.g., "Write this function"), call Gemini directly.
3. **LAST RESORT:** Claude performs the task itself. This is reserved ONLY for orchestration, final assembly, file I/O, or tasks under 100 words where Gemini overhead is not justified.

### Violation Examples
- **Violation:** Claude writes a 200-line React component directly in the chat.
    - *Why it fails:* This consumes approximately 1,000 Claude output tokens that should be Gemini tokens.
    - *Correction:* Claude should call `ask_gemini_pro` with the component requirements, receive the code, and write it to disk.
- **Violation:** Claude summarizes a 50-page PDF using its own context window.
    - *Why it fails:* Claude reads and processes 50,000+ input tokens internally, then generates a 2,000-word summary.
    - *Correction:* Claude should use `parse_document` to extract text, then pass the extracted text to `ask_gemini` for summarization.
- **Violation:** A sub-agent generates a 500-word analysis without using Gemini.
    - *Why it fails:* Sub-agents have access to the same MCP tools as the parent. They must use them.
    - *Correction:* When spawning the sub-agent, include explicit instructions: "Use `ask_gemini` for any analysis over 300 words."
- **Violation:** Claude performs 5 consecutive web lookups using its native tools and then synthesizes the findings.
    - *Why it fails:* The research + synthesis can be handled entirely by `web_search` + `web_reader` + `ask_gemini`.
    - *Correction:* Use `web_search` for discovery, `web_reader` for extraction, `ask_gemini` for synthesis.

### Self-Audit After Each Task
After completing any significant task, Claude must verify:
- [ ] Did I use more than 1,000 of my own output tokens for content generation? (If yes, delegation failed.)
- [ ] Did I perform a task that `ask_gemini` or `ask_gemini_pro` could have handled?
- [ ] Did I read web content natively instead of using `web_reader`?
- [ ] Did I process a PDF natively instead of using `parse_document`?
- [ ] Did my sub-agents use Gemini for their heavy work?
- [ ] Is the final output written to disk (not just in conversation)?

---

## Tool Delegation Guidelines

### ask_gemini (Flash)

**Role:** High-speed, low-cost execution for standard analytical and writing tasks.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `prompt` | string | (required) | The task description and all necessary context. |
| `system_prompt` | string | (optional) | Persona or behavioral instruction for the model. |
| `temperature` | number | 0.7 | Controls randomness. Use 0.1-0.3 for factual tasks, 0.7-0.9 for creative tasks. |
| `max_tokens` | number | 8192 | Maximum output tokens. Increase to 16384 for longer outputs. |

**When to Use:**
- Drafting documents, emails, or summaries exceeding 300 words.
- Synthesizing research from multiple sources into a report.
- Generating boilerplate content or templates.
- Performing initial analysis before deeper Pro-level investigation.

**Good Example:** "Summarize these 10 meeting transcripts into a bulleted list of action items, grouped by responsible party."
**Bad Example:** "Write a complex recursive algorithm with memoization for graph traversal." (Use Pro instead.)

**Edge Cases:**
- If Flash returns a shallow or incomplete analysis, retry with a more specific prompt before upgrading to Pro.
- For very long inputs (>100k tokens), ensure the prompt is well-structured with clear section markers.

### ask_gemini_pro (Pro)

**Role:** Deep reasoning, complex coding, and multi-step architectural planning.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `prompt` | string | (required) | Detailed technical requirements and context. |
| `system_prompt` | string | (optional) | Define the expert persona and constraints. |
| `temperature` | number | 0.7 | Use 0.1-0.3 for deterministic code, 0.5-0.7 for architectural design. |
| `max_tokens` | number | 16384 | Maximum output tokens. Sufficient for most code generation tasks. |

**When to Use:**
- Code generation exceeding 50 lines.
- Complex algorithm implementation (recursion, graph algorithms, state machines).
- Multi-file refactoring with cross-reference awareness.
- Technical architecture documents with code examples.

**Recommended System Prompts:**
- Code generation: "You are a senior software engineer. Write clean, well-documented code with proper error handling and type safety. Include JSDoc or docstring comments."
- Architecture: "You are a principal architect. Design systems for scalability, maintainability, and security. Provide concrete implementation details, not abstract descriptions."
- Refactoring: "You are a code reviewer. Identify anti-patterns, suggest improvements, and provide the refactored code. Preserve existing test interfaces."

**Good Example:** "Refactor this 500-line monolithic function into a modular architecture using the Strategy pattern. Provide the full implementation with tests."
**Bad Example:** "What is the capital of France?" (Waste of Pro resources; use Flash or Claude directly.)

**Error Handling:**
- If Pro returns a 429 (Rate Limit), implement a 30-second backoff, then fall back to Flash with a simplified version of the task.
- If Pro returns incomplete code (truncated output), increase `max_tokens` and retry.

### web_search

**Role:** Real-time information retrieval with Google Search grounding.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `search_query` | string | (required) | The search terms. Be specific and detailed. |
| `count` | integer | 10 | Number of results to return (1-50). Use 30-50 for deep research. |
| `search_recency_filter` | string | "noLimit" | Options: oneDay, oneWeek, oneMonth, oneYear, noLimit. |
| `search_domain_filter` | string | (optional) | Whitelist specific domains (comma-separated). |

**When to Use:**
- Any research task requiring information beyond Claude's knowledge cutoff.
- Competitive intelligence gathering (use count=30-50).
- Verifying claims or checking current data (prices, statistics, events).
- Finding authoritative sources for a specific topic.

**Good Example:** `web_search("enterprise AI adoption trends manufacturing 2025", count=30, search_recency_filter="oneMonth")`
**Bad Example:** `web_search("Python", count=5)` (Too generic; general knowledge does not require search.)

**Edge Cases:**
- For niche topics, try multiple query variations if the first search yields poor results.
- Always pair with `web_reader` for full content extraction from top results.

### web_reader

**Role:** High-fidelity content extraction from specific URLs.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `url` | string | (required) | The target webpage URL. Must be valid http/https. |
| `return_format` | string | "markdown" | Options: markdown (best for LLM), text (plain text). |
| `with_images_summary` | boolean | false | Include summary of images found on the page. |
| `with_links_summary` | boolean | false | Include summary of links found on the page. |

**Standard Workflow:**
1. `web_search` to discover relevant URLs.
2. `web_reader` on the top 3-5 URLs (call in parallel for speed).
3. `ask_gemini` to synthesize the extracted content into a structured analysis.

**Edge Cases:**
- For paywalled sites, `web_reader` may return a login page or cookie consent. Claude must detect this and inform the user or try an alternative source.
- If a URL fails, try using `web_search` to find a cached version or alternative source.
- Some JavaScript-heavy single-page applications may not render content properly. Note this limitation to the user.

### parse_document

**Role:** Advanced OCR and structured data extraction from PDFs and images.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `file_url` | string | (required) | URL of the file to parse (must be publicly accessible). |
| `return_format` | string | "markdown" | Options: markdown (default), text (plain text). |
| `parse_mode` | string | "auto" | Options: auto (automatic detection), ocr (force OCR), layout (preserve layout). |

**When to Use:**
- Extracting text from scanned documents, PDFs with complex tables, or images of text.
- Processing diagrams, flowcharts, or screenshots of code.
- Analyzing multi-page reports or contracts for key clauses.

**Good Example:** "Extract all data from the tables in this quarterly earnings PDF and return as structured markdown."
**Bad Example:** "Read this 1-page plain text file." (Standard file reading is faster and cheaper.)

**Processing Pattern:**
1. Call `parse_document` to extract raw text/structure.
2. Pass the extracted content to `ask_gemini` for analysis, summarization, or comparison.
3. Claude writes the final output to disk.

---

## Model Selection Strategy

### Decision Flowchart
```
Is the task file I/O, planning, or under 100 words?
  YES -> Claude handles it directly.
  NO  -> Does the task require real-time or recent data?
           YES -> Use web_search + web_reader + ask_gemini.
           NO  -> Is the task complex coding or deep reasoning?
                    YES -> Use ask_gemini_pro (temperature 0.1-0.3).
                    NO  -> Use ask_gemini (Flash).
```

### Claude Sonnet (Orchestrator)
- **Primary Role:** File operations, tool calling, quick responses, and high-level planning.
- **Delegation Strategy:** Delegate 100% of "content creation" tasks. Sonnet should act as a Project Manager who assigns work to Gemini workers.
- **Token Budget:** Target under 500 output tokens per task for orchestration overhead.

### Claude Opus (Advanced Orchestrator)
- **Primary Role:** Complex multi-step orchestration where logic flow is highly non-linear.
- **Priority 1:** Spawn parallel sub-agents (each sub-agent uses Gemini for its execution).
- **Priority 2:** Direct delegation to Gemini Pro for single hard problems.
- **Priority 3:** Self-execution. Only if the logic is too meta-cognitive for Gemini or requires real-time iterative judgment.

### Upgrade Guidance (Flash to Pro)
- **Hallucination Check:** If Flash provides code that does not compile or logic that is circular, immediately upgrade the task to Pro.
- **Context Complexity:** If the input context exceeds 30k tokens with complex cross-references, Pro is preferred for its higher attention stability.
- **Instruction Following:** If Flash ignores specific formatting constraints after two attempts, move to Pro with a strict `system_prompt`.

---

## Consumption Optimization Rules

### Rule 1: Never Synthesize in Claude What Gemini Can Do
- **Wrong:** Claude reads 5 competitor websites and writes a 2,000-word analysis.
- **Right:** `web_search` -> `web_reader` -> `ask_gemini` synthesizes -> Claude presents the result.

### Rule 2: Delegate All Multi-Document Analysis
- **Wrong:** Claude analyzes 3 PDF reports and compares them.
- **Right:** `parse_document` x3 (in parallel) -> `ask_gemini` comparison -> Claude formats output.

### Rule 3: Use Gemini for First Drafts, Claude for Integration
- **Wrong:** Claude generates an entire proposal section from scratch.
- **Right:** `ask_gemini` generates the draft -> Claude adapts to project voice -> writes to disk.

### Rule 4: Parallel Tool Usage When Possible
- Parse multiple PDFs simultaneously.
- Read multiple web sources in parallel.
- Then aggregate all results with a single `ask_gemini` call.

### Expected Consumption Patterns

**Scenario 1: Research Task**
```
Before (Claude-only):
  - Research: 3,000 tokens
  - Read sources: 10,000 tokens
  - Analysis: 8,000 tokens
  Total: 21,000 Claude tokens

After (Hybrid):
  - Claude orchestration: 800 tokens
  - Gemini web_search: ~200 tokens (Google)
  - Gemini web_reader: ~1,500 tokens (Google)
  - Gemini ask_gemini: ~6,000 tokens (Google)
  Claude total: 800 tokens (96% reduction)
```

**Scenario 2: Code Generation Task**
```
Before (Claude-only):
  - Requirements analysis: 500 tokens
  - Code generation: 5,000 tokens
  - Integration: 500 tokens
  Total: 6,000 Claude tokens

After (Hybrid):
  - Claude orchestration: 300 tokens
  - Gemini ask_gemini_pro: ~5,000 tokens (Google)
  - Claude file write: 200 tokens
  Claude total: 500 tokens (92% reduction)
```

**Scenario 3: Document Analysis Task**
```
Before (Claude-only):
  - PDF reading: 15,000 tokens (input)
  - Analysis: 3,000 tokens
  Total: 18,000 Claude tokens

After (Hybrid):
  - Claude orchestration: 400 tokens
  - Gemini parse_document: ~12,000 tokens (Google)
  - Gemini ask_gemini: ~3,000 tokens (Google)
  Claude total: 400 tokens (98% reduction)
```

---

## Hybrid Orchestration Model

```text
       +----------------------------------------------+
       |            CLAUDE (Parent/Brain)              |
       |  - Planning and strategy                      |
       |  - File system operations (read/write)        |
       |  - Final quality assurance                    |
       |  - Quick responses under 100 words            |
       +----------------------+------------------------+
                              |
          +-------------------+--------------------+
          |                                        |
          v                                        v
 PRIORITY 1: Parallel Sub-Agents       PRIORITY 2: Direct Delegation
 +----------------------------+        +----------------------------+
 | Sub-Agent A (Gemini Flash) |        |                            |
 +----------------------------+        |   ask_gemini_pro           |
 | Sub-Agent B (Gemini Pro)   |        |   (Single Large Task)      |
 +----------------------------+        |                            |
 | Sub-Agent C (Gemini Flash) |        +----------------------------+
 +----------------------------+
```

### Sub-Agent Spawning Protocol
When Claude spawns a sub-agent, it **MUST** include the following instruction in the sub-agent prompt:

> "You are a specialized sub-agent. You have access to Gemini MCP tools including `ask_gemini`, `ask_gemini_pro`, `web_search`, `web_reader`, and `parse_document`. You MUST use these tools for any task involving more than 50 lines of code or 300 words of analysis. Do not consume Claude's native tokens for bulk generation. Pattern: Gemini generates -> you write to disk."

Sub-agents must return structured output, not free-form prose. Recommended format:
```
## Result
- Status: complete | partial | failed
- Output file: /path/to/output.md
- Key findings: [bulleted list]
- Open questions: [if any]
```

---

## Common Patterns

### Research Pipeline
1. Claude calls `web_search` with a specific query and `count=30`.
2. Claude calls `web_reader` for the top 3-5 URLs (in parallel).
3. Claude passes the extracted text to `ask_gemini` with prompt: "Synthesize a structured report from these sources. Include citations."
4. Claude saves the resulting report to disk.

### Code Generation Workflow
1. Claude identifies the need for a new module or feature.
2. Claude calls `ask_gemini_pro` with the architectural requirements, interface definitions, and coding standards.
3. Gemini returns the full implementation.
4. Claude performs a quick review, writes the file to disk, and runs any relevant tests.

### Document Analysis
1. Claude uses `parse_document` on a complex PDF.
2. Claude sends the extracted text to `ask_gemini` with analysis instructions (e.g., "Extract all key terms and create a comparison table").
3. Claude receives the structured output and writes it to disk.

### Multi-Source Competitive Analysis
1. **Search:** Call `web_search` for "Product A vs Product B features comparison" with `count=30`.
2. **Extract:** Call `web_reader` for the top 3 comparison articles (in parallel).
3. **Synthesize:** Call `ask_gemini_pro` with the prompt: "Based on these 3 articles, create a feature parity matrix in Markdown. Include pricing, capabilities, integrations, and limitations."
4. **Finalize:** Claude formats the matrix, adds any project-specific context, and writes to disk.

### Iterative Refinement Loop
1. **Draft:** Call `ask_gemini` to create a rough draft of a document or analysis.
2. **Review:** Claude identifies gaps, inaccuracies, or missing details in the draft.
3. **Refine:** Call `ask_gemini_pro` with the draft plus specific instructions addressing the identified gaps.
4. **Polish:** Claude performs a final pass for consistency and saves the file.
5. **Repeat:** If further refinement is needed, return to step 2 with the updated draft.

### Parallel Execution
For tasks with 3+ independent components:
1. Spawn Sub-Agent A: research component -> uses `web_search` + `ask_gemini`.
2. Spawn Sub-Agent B: code generation component -> uses `ask_gemini_pro`.
3. Spawn Sub-Agent C: documentation component -> uses `ask_gemini`.
4. Claude (parent) waits for all sub-agents to complete.
5. Claude assembles the outputs into the final deliverable.

---

## Development Workflow

### Adding New Tools
1. **Define Schema:** Add the tool definition to the MCP server's tool manifest with name, description, and input schema.
2. **Implement Handler:** Write the request handler in the appropriate API path (OpenAI SDK or Native).
3. **Error Handling:** Implement exponential backoff for `429 Rate Limit` errors. Handle `500 Internal Server Error` with retry logic (max 3 attempts).
4. **Logging:** Log token usage for both Claude and Gemini to track optimization metrics.
5. **Parameter Validation:** Validate all inputs before sending to the Gemini API. Reject malformed requests with clear error messages.

### Code Quality Standards
- **Input Validation:** All tool inputs must be validated against the declared JSON schema before API dispatch.
- **Sanitization:** Remove any potential PII, API keys, or sensitive data from logs and error messages.
- **Documentation:** Every tool must have a clear description and example usage in the MCP manifest.
- **Error Messages:** Return actionable error messages that help Claude (or the user) understand what went wrong and how to fix it.
- **Type Safety:** Use TypeScript for all server code. Define interfaces for all API request/response types.

### Testing New Tools
To test a new tool implementation without launching Claude Desktop:
1. Use the MCP Inspector: `npx @modelcontextprotocol/inspector dist/index.js`
2. Send a JSON-RPC request:
   ```json
   {
     "method": "call_tool",
     "params": {
       "name": "ask_gemini",
       "arguments": { "prompt": "Hello world", "max_tokens": 100 }
     }
   }
   ```
3. Verify the response structure matches the expected output schema.
4. Test edge cases: empty prompt, very long prompt, special characters, rate limit behavior.

### Debugging Failed Delegations
1. **Check Logs:** View the MCP server logs in Claude Desktop (Settings -> Developer -> View Logs).
2. **Verify API Key:** Ensure `GOOGLE_API_KEY` is correctly set and the key has not been revoked or rate-limited.
3. **Inspect Payload:** Look for malformed JSON errors in the tool arguments. Common issue: unescaped quotes in prompts.
4. **Rate Limits:** If you see 429 errors, implement a backoff strategy. Free tier limits: Flash 15 RPM, Pro 5 RPM.
5. **Model Availability:** Check Google AI Studio for any outage notifications or model deprecation notices.

---

## API Reference

### Endpoints
- **Native API Base:** `https://generativelanguage.googleapis.com/v1beta`
- **OpenAI-Compatible Base:** `https://generativelanguage.googleapis.com/v1beta/openai/`

### Tool Parameter Reference

| Tool | Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ask_gemini` | `prompt` | string | Yes | -- | The main instruction and context. |
| `ask_gemini` | `system_prompt` | string | No | -- | Persona or behavioral guidance. |
| `ask_gemini` | `temperature` | number | No | 0.7 | Randomness control (0.0-1.0). |
| `ask_gemini` | `max_tokens` | number | No | 8192 | Maximum output tokens. |
| `ask_gemini_pro` | `prompt` | string | Yes | -- | Detailed technical requirements. |
| `ask_gemini_pro` | `system_prompt` | string | No | -- | Expert persona definition. |
| `ask_gemini_pro` | `temperature` | number | No | 0.7 | Randomness control (0.0-1.0). |
| `ask_gemini_pro` | `max_tokens` | number | No | 16384 | Maximum output tokens. |
| `web_search` | `search_query` | string | Yes | -- | The search query string. |
| `web_search` | `count` | integer | No | 10 | Results to return (1-50). |
| `web_search` | `search_recency_filter` | string | No | noLimit | Time filter: oneDay, oneWeek, oneMonth, oneYear, noLimit. |
| `web_search` | `search_domain_filter` | string | No | -- | Whitelist domains (comma-separated). |
| `web_reader` | `url` | string | Yes | -- | Target URL (http/https). |
| `web_reader` | `return_format` | string | No | markdown | Output format: markdown or text. |
| `web_reader` | `with_images_summary` | boolean | No | false | Include image descriptions. |
| `web_reader` | `with_links_summary` | boolean | No | false | Include link inventory. |
| `parse_document` | `file_url` | string | Yes | -- | URL of the file to parse. |
| `parse_document` | `return_format` | string | No | markdown | Output format: markdown or text. |
| `parse_document` | `parse_mode` | string | No | auto | Mode: auto, ocr, or layout. |

### Model Capability Comparison

| Feature | Gemini Flash | Gemini Pro | Claude Sonnet | Claude Opus |
| :--- | :--- | :--- | :--- | :--- |
| Speed | Very High | Medium | High | Medium |
| Context Window (Input) | 1M+ tokens | 2M+ tokens | 200k tokens | 200k tokens |
| Max Output Tokens | 8,192 (default) | 16,384 (default) | 8,192 | 8,192 |
| Coding | Good | Excellent | Excellent | Excellent |
| Search Grounding | Yes (native) | Yes (native) | No | No |
| Document OCR | Yes | Yes | Limited | Limited |
| Cost per Token | Minimal | Moderate | High | Very High |
| Recommended Role | Bulk execution | Complex tasks | Orchestration | Advanced orchestration |

### Rate Limits (Free Tier)
- **Gemini 3 Flash:** 15 Requests Per Minute (RPM), 1 million Tokens Per Minute (TPM).
- **Gemini 3 Pro:** 5 Requests Per Minute (RPM), 32,000 Tokens Per Minute (TPM).

*Note: These limits are subject to change. For higher limits, upgrade to a paid tier in Google AI Studio.*

---

## Troubleshooting

### Tools Not Appearing in Claude Desktop
1. Check `claude_desktop_config.json` for syntax errors (missing commas, incorrect paths).
2. Verify the MCP server path in `args` is correct and the built file exists.
3. Ensure the `GOOGLE_API_KEY` environment variable is set in the config.
4. Restart Claude Desktop completely (quit the application, not just close the window).
5. Check that Node.js v18+ is installed and accessible from the system PATH.

### Empty or Truncated Responses
1. Increase `max_tokens` parameter. Flash defaults to 8,192; try 16,384 for longer outputs.
2. Lower `temperature` for analytical tasks (use 0.1-0.3 instead of 0.7).
3. Check if Gemini safety filters are being triggered. Review the `finish_reason` in API response logs.
4. Verify the prompt is not exceeding the model's input context window.

### Rate Limit Errors (429)
1. **Free tier limits:** Flash 15 RPM, Pro 5 RPM. Wait 60 seconds and retry.
2. Switch from `ask_gemini_pro` to `ask_gemini` (Flash) if Pro limits are hit.
3. For sustained high-volume work, upgrade to a paid tier in Google AI Studio.
4. Implement client-side rate limiting to avoid hitting API limits.

### Model Not Found Errors
1. Verify the model identifier matches Google's current naming convention.
2. Use exact model IDs: `gemini-3-flash-preview` or `gemini-3-pro-preview`.
3. Preview models may be renamed or deprecated. Check https://ai.google.dev for the latest stable model names.
4. If using environment variable overrides, ensure they are spelled correctly.

### Context Window Exceeded
1. The input prompt is too large for the model. Split the content into smaller chunks.
2. Use `parse_document` or `web_reader` to pre-process large documents instead of passing raw content.
3. For very large analyses, use a two-pass approach: first summarize each document individually, then analyze the summaries.

### Malformed Response Handling
1. Gemini occasionally wraps JSON output in markdown code fences (```json ... ```). The MCP server includes sanitization logic to strip these.
2. If Claude receives unparseable output, log the raw response for debugging.
3. Retry the request with a more explicit output format instruction in the prompt.

### Performance Degradation Diagnosis
- **Symptom:** Tool calls take more than 10 seconds.
- **Check 1:** Is `web_search` being used? Grounding adds latency (typically 2-5 seconds).
- **Check 2:** Is the Gemini API experiencing high load? Check the Google Cloud Status dashboard.
- **Check 3:** Is the prompt excessively long? Reduce input size or pre-process with a summarization step.
- **Check 4:** Is the MCP server running on a resource-constrained machine? Monitor CPU and memory usage.

### High Claude Consumption Audit
If Claude consumption remains high despite Gemini integration, check for:
1. Claude "thinking out loud" extensively before calling a tool (excessive planning tokens).
2. Claude summarizing or reformatting Gemini's output instead of passing it through directly.
3. Failure to use parallel sub-agents for multi-part tasks (sequential processing wastes Claude tokens).
4. Sub-agents not using Gemini tools (generating content natively instead of delegating).
5. Claude reading large files into its context instead of using `parse_document`.

**Expected token patterns per task type:**
- Research task: Claude 800-2,000 tokens, Gemini 6,000-10,000 tokens.
- Code generation: Claude 500-1,000 tokens, Gemini 5,000-8,000 tokens.
- Document analysis: Claude 400-800 tokens, Gemini 8,000-15,000 tokens.

If Claude consumption exceeds 3,000 tokens on analytical work, delegation is not optimized.

---

## Error Recovery
In the event of a mid-session failure (MCP server crash, API outage, context degradation):

1. **Immediate State Preservation:** Write current progress to a recovery file with: decisions made, files modified, open questions, exact next action.
2. **Inform the User:** Clearly state what work was completed, what may have been lost, and what needs to be re-done.
3. **Restart Protocol:** After restarting Claude Desktop, read the recovery file to restore context.
4. **Resume:** Use Gemini to regenerate any lost intermediate outputs rather than re-processing from scratch.
5. **If auto-compact fires unexpectedly:** Write state to disk immediately, notify the user of potential context loss, and suggest starting a fresh session with the recovery file as input.

---

## Version History

| Version | Date | Changes |
| :--- | :--- | :--- |
| v1.0.0 | -- | Initial release with `ask_gemini` and `web_search`. |
| v1.1.0 | -- | Added `parse_document` and multi-modal support. |
| v1.2.0 | -- | Implemented OpenAI SDK compatibility layer for chat tools. |
| v1.3.0 | -- | Added `web_reader` for URL content extraction. |
| v2.0.0 | -- | Hybrid architecture: OpenAI SDK for chat, Native API for web tools. |
| v2.1.0 | -- | Added `ask_gemini_pro` with dedicated Pro model support. |
| v3.0.0 | -- | Full delegation gate, sub-agent protocol, consumption optimization rules. |

---
> Source: [Arkya-AI/claude-additional-models-mcp](https://github.com/Arkya-AI/claude-additional-models-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
