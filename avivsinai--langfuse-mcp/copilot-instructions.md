## langfuse-mcp-general-rules

> - We **ONLY** use `uv`/`uvx` in order to run, build our app, **do not use vanilla Python or scripts**


# How to build, run, test and debug Langfuse MCP project effectively

- We **ONLY** use `uv`/`uvx` in order to run, build our app, **do not use vanilla Python or scripts**
- Our server logs reside in `/tmp/langfuse_mcp.log` - read them in our for properly debugging our server
- For debugging Cursor MCP client, the logs exist in: `~/Library/Application\ Support/Cursor/logs/`
- In order to only find MCP related logs one can use `find ~/Library/Application\ Support/Cursor/logs/ -type f -exec grep -i "mcp" {} \; `
- Since we aren't using SSE and there are many SSE implementations that are used, one can also use `find ~/Library/Application\ Support/Cursor/logs/ -type f -exec grep -i "mcp" {} \; | grep -v "SSE error"`
- Read [README.md](mdc:langfuse-mcp/README.md) for various information, which relates to our project.

## Building and Publishing to PyPI

### Version Numbering and Tagging
- We use semantic versioning with format `v0.x.y` for version tags
- Always create a git tag for each release with the format `v0.x.y`

### Build and Publish Workflow
1. **Commit your changes**:
   ```bash
   git add <modified files>
   git commit -m "Your commit message"
   ```

2. **Tag the release**:
   ```bash
   git tag -a v0.x.y -m "Brief description of the release"
   ```

3. **Build the wheel (skip source distribution)**:
   ```bash
   rm -rf dist/
   uv build --wheel --no-build-logs --index-strategy unsafe-best-match
   ```

4. **Publish to TestPyPI first**:
   ```bash
   uv publish --index testpypi dist/langfuse_mcp-0.x.y-py3-none-any.whl
   ```
   - Use `__token__` as username 
   - Use your TestPyPI API token as password

5. **Push tag and changes to remote**:
   ```bash
   git push origin v0.x.y
   git push
   ```

6. **Publish to PyPI (when ready)**:
   ```bash
   uv publish dist/langfuse_mcp-0.x.y-py3-none-any.whl
   ```
   - Use `__token__` as username
   - Use your PyPI API token as password

---
> Source: [avivsinai/langfuse-mcp](https://github.com/avivsinai/langfuse-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
