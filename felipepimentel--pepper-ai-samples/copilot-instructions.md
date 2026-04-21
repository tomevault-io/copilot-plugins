## pepper-ai-samples

> This example demonstrates the integration of A2A (Agent-to-Agent) protocol and MCP (Model Context Protocol) to create a web search and summarization application.

# Web Search Summarizer Example

This example demonstrates the integration of A2A (Agent-to-Agent) protocol and MCP (Model Context Protocol) to create a web search and summarization application.

## Core Components

1. **MCP Server**: Provides web search capabilities
   - Uses the `web_search` tool to query the web
   - Returns structured search results

2. **A2A Server**: Provides summarization capabilities
   - Uses the `summarize` capability to process search results
   - Generates structured summaries with main topics and key points

3. **A2A-MCP Bridge**: Connects both servers
   - Allows the A2A server to use MCP web search tool
   - Enables orchestration of the complete workflow

## Implementation Files

- [server.py](mdc:a2a-mcp-examples/02-web-search-summarizer/server.py): Main implementation with server setup and business logic
- [a2a_mcp_bridge.py](mdc:a2a-mcp-examples/02-web-search-summarizer/a2a_mcp_bridge.py): Bridge implementation that connects A2A and MCP
- [templates/index.html](mdc:a2a-mcp-examples/02-web-search-summarizer/templates/index.html): Frontend user interface for search and summarization

## Key Functions

### Web Search

```python
@mcp_server.tool()
async def web_search(query: str) -> Dict[str, Any]:
    """Perform web search and return results"""
    # Uses MCP web search tool or falls back to simulated search
    # Returns structured search results
```

### Summarization

```python
@a2a_server.capability("summarize")
async def summarize(data: Dict[str, Any]) -> Dict[str, Any]:
    """Summarize search results"""
    # Extracts snippets and titles from search results
    # Identifies main topics and key points
    # Returns formatted summary with sections
```

### Helper Functions

```python
def extract_main_topics(query: str, titles: List[str], snippets: List[str]) -> List[str]:
    """Extract main topics from search results"""

def extract_key_points(snippets: List[str]) -> List[str]:
    """Extract key points from search result snippets"""

def format_bullet_points(items: List[str]) -> str:
    """Format a list of items as bullet points"""
```

## Usage Flow

1. User submits a search query via the web interface
2. The application calls the MCP web search tool
3. Search results are passed to the A2A summarization capability
4. The generated summary is returned alongside search results
5. The web interface displays both the summary and search results

## HTTP Endpoints

- `POST /api/search`: Performs web search only
- `POST /api/search-and-summarize`: Performs search and generates summary

## Templates

The application uses a responsive HTML template with JavaScript for the frontend interface:
```html
<div class="search-container">
    <input type="text" id="search-input" placeholder="Enter your search query">
    <div class="button-container">
        <button id="search-button">Search Only</button>
        <button id="summarize-button">Search & Summarize</button>
    </div>
</div>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felipepimentel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
