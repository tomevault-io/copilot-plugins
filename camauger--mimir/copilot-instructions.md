## mimir

> **Mimir** is a VS Code/Cursor extension that converts DOCX documents to JSON and HTML formats. It's specifically designed to process pedagogical components, making it ideal for educational content management and documentation workflows.

# CLAUDE.md - Mimir Extension Documentation

## Project Overview

**Mimir** is a VS Code/Cursor extension that converts DOCX documents to JSON and HTML formats. It's specifically designed to process pedagogical components, making it ideal for educational content management and documentation workflows.

### Key Features
- Convert DOCX files to HTML with pedagogical component support
- Convert DOCX files to JSON for downstream processing
- Preview DOCX files as HTML directly in the editor
- Flexible image handling (save to disk or base64 encoding)

---

## Architecture

### Technology Stack
- **Runtime**: Node.js (VS Code Extension Host)
- **Language**: JavaScript (extension.js)
- **Backend**: Python script (external process)
- **VS Code API**: ^1.96.0

### Extension Structure

```
mimir/
├── extension.js         # Main extension logic
├── package.json         # Extension manifest and configuration
├── README.md           # User documentation (French)
├── LICENCE.md          # MIT License
├── CLAUDE.md           # This file (AI assistant documentation)
└── mimir-0.1.0.vsix    # Packaged extension
```

### Command Registration

The extension registers three commands:

1. **mimir.convertToHtml** - Convert DOCX to HTML
2. **mimir.convertToJson** - Convert DOCX to JSON
3. **mimir.previewHtml** - Preview DOCX as HTML in webview

All commands are available via right-click context menu on `.docx` files in the explorer.

---

## How It Works

### Conversion Flow

```
User Action → VSCode Command → exec() Python Script → Output File → User Notification
```

1. **File Selection**: User right-clicks a DOCX file or triggers command without selection
2. **Configuration Loading**: Extension reads user settings for Python path, script path, and image handling
3. **Command Execution**: Spawns Python process with appropriate flags
4. **Progress Notification**: Shows VS Code progress notification during conversion
5. **Result Handling**:
   - Success: Shows notification with "Open" button
   - Error: Shows error message with details

### Python Script Integration

The extension calls an external Python script with the following pattern:

```bash
python <scriptPath> <docxPath> --output-dir <dir> [--save-images|--base64-images] [--html|--json]
```

**Expected Script Interface:**
- Accepts DOCX file path as first argument
- `--output-dir`: Specifies output directory
- `--save-images`: Saves images to disk
- `--base64-images`: Encodes images as base64
- `--html`: Outputs HTML format
- `--json`: Outputs JSON format

---

## Configuration

### User Settings

Three configuration options are exposed via `package.json`:

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `mimir.pythonPath` | string | `"python"` | Path to Python executable |
| `mimir.scriptPath` | string | `""` | Path to conversion script |
| `mimir.saveImagesToDisk` | boolean | `true` | Save images vs base64 encoding |

### Setting Locations
- User Settings: `~/.config/Code/User/settings.json`
- Workspace Settings: `.vscode/settings.json`

Example configuration:
```json
{
  "mimir.pythonPath": "/usr/bin/python3",
  "mimir.scriptPath": "/path/to/converter/script.py",
  "mimir.saveImagesToDisk": true
}
```

---

## Development

### Prerequisites
- Node.js (v14 or higher)
- VS Code (v1.96.0 or higher)
- Python 3.x with required conversion libraries

### Setup

1. Clone the repository:
```bash
git clone https://github.com/camauger/mimir.git
cd mimir
```

2. Install dependencies:
```bash
npm install
```

3. Open in VS Code:
```bash
code .
```

4. Press F5 to launch Extension Development Host

### Testing

The extension includes ESLint for code quality:

```bash
npm run lint          # Run linter
npm run pretest       # Lint before testing
npm test              # Run tests
```

### Building

To package the extension:

```bash
vsce package
```

This creates a `.vsix` file that can be installed manually or published to the marketplace.

---

## Code Structure

### extension.js Analysis

The main extension file contains three similar command handlers:

1. **File Selection Logic** (lines 12-32, 85-105, 157-178)
   - Checks if file URI is provided
   - If not, searches workspace for all `.docx` files
   - Shows quick pick dialog for user selection

2. **Configuration Retrieval** (lines 41-44, 114-117, 193-195)
   - Loads user/workspace settings
   - Retrieves Python path and script path
   - Determines image handling mode

3. **Command Execution** (lines 48-80, 121-153, 198-242)
   - Constructs shell command with proper escaping
   - Shows progress notification
   - Executes Python script via `child_process.exec`
   - Handles success/error cases

### Key Implementation Details

**Progress Feedback**: Uses `vscode.window.withProgress` for non-blocking notifications

**Error Handling**: Catches execution errors and displays via `vscode.window.showErrorMessage`

**Preview Mode**:
- Creates temporary `.preview` directory
- Always uses base64 encoding to avoid path issues
- Renders HTML in webview panel (column 2)

**Security Considerations**:
- Uses quoted paths to handle spaces
- Executes external Python script (users must trust the script)
- WebView enables scripts for preview functionality

---

## Known Issues & Limitations

### Current Limitations

1. **No Python Script Included**: Extension requires external Python script
   - Users must provide their own conversion script
   - Script must match expected CLI interface

2. **Limited Error Context**:
   - Error messages show generic failures
   - No detailed validation of Python script output

3. **No Input Validation**:
   - Doesn't verify Python path exists
   - Doesn't check if script path is valid
   - No validation of script output format

4. **Synchronous Execution**:
   - Uses `exec()` instead of `spawn()`
   - May have issues with large files or slow scripts
   - No streaming output

5. **Temporary Directory Handling**:
   - Preview creates `.preview` folder
   - No automatic cleanup mechanism

### Potential Improvements

1. **Add Python Script**:
   - Include default conversion script
   - Bundle with extension or provide download

2. **Better Error Handling**:
   - Validate configuration on activation
   - Parse stderr for specific error messages
   - Provide troubleshooting guidance

3. **Progress Reporting**:
   - Stream Python script output
   - Show detailed progress stages
   - Add cancellation support

4. **Preview Enhancements**:
   - Auto-refresh on file changes
   - Side-by-side comparison
   - Export preview to standalone HTML

5. **Configuration UX**:
   - Auto-detect Python installation
   - Wizard for first-time setup
   - Validate script compatibility

---

## Extension Points

### Adding New Output Formats

To add a new output format:

1. Register new command in `package.json`:
```json
{
  "command": "mimir.convertToXml",
  "title": "DOCX: Convertir en XML"
}
```

2. Add menu item for `.docx` files:
```json
{
  "when": "resourceExtname == .docx",
  "command": "mimir.convertToXml",
  "group": "mimir"
}
```

3. Implement command handler in `extension.js`:
```javascript
let convertToXml = vscode.commands.registerCommand(
  'mimir.convertToXml',
  async (fileUri) => {
    // Similar logic to convertToHtml/convertToJson
    // Add --xml flag to Python command
  }
);
context.subscriptions.push(convertToXml);
```

### Customizing Preview

The preview functionality uses a WebView panel (line 225). To customize:

- Inject CSS: Modify HTML content before setting `panel.webview.html`
- Add interactivity: Set `enableScripts: true` and include JavaScript
- Message passing: Use `webview.postMessage()` and `webview.onDidReceiveMessage()`

---

## Repository Information

- **Author**: Christian Amauger
- **License**: MIT
- **Repository**: https://github.com/camauger/mimir.git
- **Publisher**: christian-amauger
- **Version**: 0.1.0

---

## Contributing

### Development Branch
Currently working on: `claude/create-claude-docs-01VCLprGupY2ryLExHdZPSbX`

### Contribution Guidelines

1. Fork the repository
2. Create a feature branch
3. Make changes with clear commit messages
4. Run `npm run lint` to ensure code quality
5. Test in Extension Development Host (F5)
6. Submit pull request with detailed description

### Code Style
- Use ES6+ JavaScript features
- Follow existing indentation (2 spaces)
- Add JSDoc comments for exported functions
- Keep functions focused and single-purpose

---

## Support & Troubleshooting

### Common Issues

**"No Python path configured"**
- Set `mimir.pythonPath` in settings
- Use absolute path to Python executable

**"Script not found"**
- Set `mimir.scriptPath` to conversion script
- Ensure script has execute permissions

**"Conversion failed"**
- Check Python script is compatible
- Verify script accepts expected CLI arguments
- Check error output in Developer Tools console

### Debug Mode

To see detailed error output:
1. Help → Toggle Developer Tools
2. Check Console tab for stderr output
3. Review execution commands and paths

---

## Future Roadmap

Potential enhancements for future versions:

- [ ] Bundle default Python conversion script
- [ ] Add batch conversion support
- [ ] Implement configuration validation on startup
- [ ] Add output format templates
- [ ] Support for other document formats (ODT, RTF)
- [ ] Integration with file watchers for auto-conversion
- [ ] Localization support (currently French-focused)
- [ ] Unit and integration tests
- [ ] CI/CD pipeline setup
- [ ] Marketplace publishing

---

## Technical Notes for AI Assistants

### Command Name Inconsistency
There's a discrepancy between command names:
- `package.json` uses: `mimir.convertToHtml`, `mimir.convertToJson`, `mimir.previewHtml`
- `extension.js` registers: `docx-json-converter.convertToHtml`, etc.

This may cause the commands to not work properly. The extension.js should be updated to match package.json naming.

### Security Considerations
The extension executes external Python scripts using `child_process.exec()`. This is potentially unsafe if:
- Users run untrusted scripts
- Input paths contain shell metacharacters
- Script path is user-controllable

Consider using `execFile()` instead for better security.

### Performance
Using `exec()` buffers entire output in memory. For large documents, consider:
- Using `spawn()` with streaming
- Implementing progress callbacks
- Adding timeout limits

---

*This documentation was created to assist AI assistants and developers in understanding, maintaining, and extending the Mimir VS Code extension.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/camauger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
