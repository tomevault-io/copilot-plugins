## wfam

> Scope: This folder only.

# Project Rules: WFAM (WinFormsAutoMCP)

Scope: This folder only.

## Language Policy (CRITICAL)

**Internal Processing**: MUST use English for analysis/reasoning.

**Output (MANDATORY)**:
- User responses: Japanese only
- Code comments: Japanese only
- Design docs (DESIGN.md): Japanese only (MANDATORY)
- README files: Japanese only (MANDATORY)
- Documentation: Japanese only
- Explanations: Japanese only
- Technical terms (API/function names): English OK, but explain in Japanese

**Enforcement**: If output is in English, user says "日本語で" to correct.

## Work Process (MANDATORY)

### 1. Pre-Work
- Read `./DESIGN.md` fully before starting
- Verify task is within this project scope
- Never start without reading DESIGN.md
- Check `../DESIGN.md` if it exists (for parent project context)

### 2. During Work
- Follow all methods/procedures in DESIGN.md
- Minimal changes only
- Keep existing code style
- For cross-project changes, ask user first

### 3. Document Management (CRITICAL)

**Never create new documents without explicit user request:**
- Document = Any file for human reading (README, TODO, notes, guides, proposals, etc.)
- Code files (source, config, tests) are NOT documents

**Before creating any document:**
1. **Check all existing files** in project using file listing/search
2. **Use existing files** when possible:
   - Add to DESIGN.md for design/architecture decisions
   - Add to README for user-facing documentation
   - Add to existing notes/docs if applicable
3. **Only if absolutely necessary** and better for project management:
   - **Ask user first** before creating new document
   - Explain why existing files are insufficient
   - Get explicit approval

**This applies to ALL documents including:**
- TODO lists, task tracking files
- Additional README files
- Documentation files (*.md, *.txt)
- Design proposals, RFCs
- Meeting notes, decision logs

### 4. Design Sync (CRITICAL)
- Update `./DESIGN.md` when changing implementation/structure/procedures
- Report design-code inconsistencies before proceeding
- For ambiguous specs, propose DESIGN.md updates first

### 5. Completion Verification
Run these if applicable (per DESIGN.md instructions):
- **Build**: Execute build command defined in DESIGN.md, verify success, report warnings
- **Test**: Run all tests defined in DESIGN.md, verify pass, report results (pass/fail count)
- **Review**: DESIGN.md updated, change rationale documented

**Resource Cleanup**:
- Stop all processes (servers, daemons, etc.)
- Release ports, file locks, handles
- Follow DESIGN.md for proper shutdown method

### 6. Breaking Changes
- Show impact analysis first
- Get user confirmation before proceeding
- Update DESIGN.md with migration notes if needed

## Development Flow (MANDATORY)

If DESIGN.md defines a development flow, follow it strictly.
If not defined, use standard development practices with these minimums:
- Write tests for new functionality
- Verify build succeeds before commit
- Update documentation for significant changes

## Python Rules (MANDATORY)

- Use `uv` for all package management, virtual environment, and execution
- Manage dependencies in `pyproject.toml` (NOT requirements.txt)
- Use pytest for testing
- Follow PEP 8 style guide
- Use type hints for all functions (Python 3.10+)
- Use `async/await` for MCP ツールハンドラ
- `pywinauto` のブロッキング操作は必ず `asyncio.to_thread()` で実行

**uv コマンド体系**:
- 仮想環境作成: `uv venv`
- パッケージインストール: `uv pip install -e ".[dev]"`
- スクリプト実行: `uv run wfam` または `uv run python -m winforms_auto_mcp.server`
- テスト実行: `uv run pytest`

**Dependency version management**:
1. Install without version: `uv pip install package-name`
2. Check installed version: `uv pip show package-name`
3. Record exact version in pyproject.toml dependencies: `"package-name>=X.Y.Z"`
4. Example:
   ```bash
   uv pip install pywinauto
   uv pip show pywinauto  # Shows: Version: 0.6.8
   # Add to pyproject.toml: "pywinauto>=0.6.8"
   ```

## WFAM-Specific Rules (MANDATORY)

### Windows 専用プロジェクト
- 対象OS: Windows のみ（pywinauto は Windows API に依存）
- Unix/macOS 互換性は不要

### セッション管理
- アプリケーション操作は必ず `SessionManager` 経由で行う
- セッションID は `launch_app` または `attach_app` で取得
- グローバル変数で Application オブジェクトを直接管理しない

### pywinauto 操作の非同期化
- `pywinauto` の全操作はブロッキング（同期）処理
- MCP のイベントループを阻害しないよう、`asyncio.to_thread()` で別スレッドに逃がす
- `@mcp.tool()` デコレータ内では必ず `await asyncio.to_thread(sync_func)` パターンを使用

### MCP 通信
- 通信プロトコル: JSON-RPC over Standard I/O
- ローカル実行のみ（ネットワーク越しのコマンドは受け付けない）

### セキュリティ
- 信頼できる実行ファイルのみを対象とする
- README にセキュリティ警告を明記する
- 任意のEXEパスを受け付けるリスクを認識し、将来的にホワイトリスト機能を検討

## License Management (MANDATORY)

When adding dependencies/libraries:
- Track all dependency licenses
- Record license info in DESIGN.md or dedicated license file
- Check license compatibility with project license (MIT)

If application displays license info (e.g., "About" dialog, OSS notice):
- Include mechanism to collect and display dependency licenses
- Keep license display updated when dependencies change
- Follow each license's attribution requirements

## OS-Aware Command Execution (CRITICAL)

本プロジェクトは Windows 専用だが、開発環境として PowerShell / cmd の両方を考慮する。

- **Path separators**: `\` for Windows
- **Null device**: `NUL` for Windows
- **Environment variables**: `%VAR%` (cmd), `$env:VAR` (PowerShell)

**WARNING**: Redirecting to `nul` on Windows in some contexts creates a file named "nul" instead of discarding output. Use proper null device handling.

## File Safety
- Never create: `nul`, `con`, `prn`, `aux`, `com1-9`, `lpt1-9`
- Delete if accidentally created
- Avoid OS reserved names

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/satorunnlg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
