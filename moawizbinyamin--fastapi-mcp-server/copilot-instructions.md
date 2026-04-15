## fastapi-mcp-server

> - Virtual environment activated

# FastAPI MCP Server Configuration

## Environment Setup
- Python 3.8+
- Virtual environment activated
- Dependencies installed via requirements.txt

## MCP Server Configuration
The MCP server provides the following tools:
- `echo`: Echo back input text
- `add`: Add two numbers
- `get_time`: Get current server time

## Endpoints
- `/`: Root endpoint with server info
- `/health`: Health check endpoint
- `/tools`: List available tools
- `/tools/call`: Call tools via HTTP
- `/mcp`: WebSocket endpoint for MCP protocol
- `/docs`: FastAPI automatic documentation

## Running the Server
```bash
# Activate virtual environment
source venv/Scripts/activate  # Windows
# or
source venv/bin/activate      # Linux/Mac

# Run the server
python main.py
# or
uvicorn main:app --reload
```

## Testing MCP Tools
You can test the tools via HTTP:
```bash
# List tools
curl http://localhost:8000/tools

# Call echo tool
curl -X POST http://localhost:8000/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name": "echo", "arguments": {"text": "Hello World"}}'

# Call add tool
curl -X POST http://localhost:8000/tools/call \
  -H "Content-Type: application/json" \
  -d '{"name": "add", "arguments": {"a": 5, "b": 3}}'
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moawizbinyamin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
