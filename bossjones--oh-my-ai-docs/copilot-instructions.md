## oh-my-ai-docs

> This rule governs testing patterns for FastMCP context functionality. It should be applied whenever: (1) Writing tests for tools that use context injection, (2) Testing context-related features like logging or resource access, (3) Testing async context behavior, or (4) Testing context lifecycle and error handling. The rule ensures consistent and thorough testing of context functionality across FastMCP implementations.


#  tests/**/*test*.py
# FastMCP Context Testing Patterns

## Critical Rules

- ALWAYS use `@pytest.mark.anyio` for all context-related tests
- ALWAYS test both sync and async context injection scenarios
- ALWAYS verify context attributes (request_id, etc.) when injected
- ALWAYS test context logging methods (debug, info, warning, error)
- ALWAYS test context resource access when relevant
- ALWAYS test error handling for context-related operations
- ALWAYS test optional context scenarios (tools without context)
- ALWAYS test context lifecycle (lifespan) when implementing server features
- NEVER skip testing error cases for context operations
- NEVER mix sync and async code incorrectly in context tests

## Examples

<example>
```python
# Basic Context Detection Test
@pytest.mark.anyio
async def test_context_detection():
    mcp = FastMCP()

    def tool_with_context(x: int, ctx: Context) -> str:
        return f"Request {ctx.request_id}: {x}"

    tool = mcp.add_tool(tool_with_context)
    assert tool.context_kwarg == "ctx"

# Context Injection Test
@pytest.mark.anyio
async def test_context_injection():
    mcp = FastMCP()

    def tool_with_context(x: int, ctx: Context) -> str:
        assert ctx.request_id is not None
        return f"Request {ctx.request_id}: {x}"

    mcp.add_tool(tool_with_context)
    async with client_session(mcp._mcp_server) as client:
        result = await client.call_tool("tool_with_context", {"x": 42})
        assert len(result.content) == 1
        assert "Request" in result.content[0].text
        assert "42" in result.content[0].text

# Async Context Test
@pytest.mark.anyio
async def test_async_context():
    mcp = FastMCP()

    async def async_tool(x: int, ctx: Context) -> str:
        assert ctx.request_id is not None
        return f"Async request {ctx.request_id}: {x}"

    mcp.add_tool(async_tool)
    async with client_session(mcp._mcp_server) as client:
        result = await client.call_tool("async_tool", {"x": 42})
        assert "Async request" in result.content[0].text

# Context Logging Test
@pytest.mark.anyio
async def test_context_logging():
    mcp = FastMCP()

    async def logging_tool(msg: str, ctx: Context) -> str:
        await ctx.debug("Debug message")
        await ctx.info("Info message")
        await ctx.warning("Warning message")
        await ctx.error("Error message")
        return f"Logged messages for {msg}"

    with patch("mcp.server.session.ServerSession.send_log_message") as mock_log:
        async with client_session(mcp._mcp_server) as client:
            result = await client.call_tool("logging_tool", {"msg": "test"})
            assert mock_log.call_count == 4
            mock_log.assert_any_call(level="debug", data="Debug message", logger=None)

# Resource Access Test
@pytest.mark.anyio
async def test_context_resource_access():
    mcp = FastMCP()

    @mcp.resource("test://data")
    def test_resource() -> str:
        return "resource data"

    @mcp.tool()
    async def tool_with_resource(ctx: Context) -> str:
        r_iter = await ctx.read_resource("test://data")
        r_list = list(r_iter)
        assert len(r_list) == 1
        return f"Read resource: {r_list[0].content}"

    async with client_session(mcp._mcp_server) as client:
        result = await client.call_tool("tool_with_resource", {})
        assert "resource data" in result.content[0].text

# Optional Context Test
@pytest.mark.anyio
async def test_optional_context():
    mcp = FastMCP()

    def no_context(x: int) -> int:
        return x * 2

    mcp.add_tool(no_context)
    async with client_session(mcp._mcp_server) as client:
        result = await client.call_tool("no_context", {"x": 21})
        assert result.content[0].text == "42"
```
</example>

<example type="invalid">
```python
# ❌ Bad practices to avoid

# Missing anyio marker
def test_context_bad():
    mcp = FastMCP()
    def tool(ctx: Context):
        pass

# Incorrect async/sync mixing
@pytest.mark.anyio
async def test_context_bad_async():
    mcp = FastMCP()
    result = mcp.do_something()  # Missing await
    assert result

# Missing context verification
@pytest.mark.anyio
async def test_context_no_verify():
    mcp = FastMCP()
    def tool(ctx: Context):
        return "ok"  # Not verifying context attributes

# Incomplete error handling
@pytest.mark.anyio
async def test_context_incomplete_error():
    mcp = FastMCP()
    try:
        result = await mcp.do_something()
    except:  # Too broad exception handling
        pass  # No assertions on error

# Missing cleanup
@pytest.mark.anyio
async def test_context_no_cleanup():
    mcp = FastMCP()
    client = await mcp._mcp_server.connect()  # No context manager
    result = await client.call_tool("tool", {})
    # Missing client cleanup

# Incorrect context type hints
@pytest.mark.anyio
async def test_context_bad_types():
    mcp = FastMCP()
    def tool(ctx) -> str:  # Missing type hint
        return "test"
```
</example>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bossjones) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
