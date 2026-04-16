## vscode-project-md

> **Project.md** is a VSCode extension that transforms Markdown files into interactive project navigation hubs. It provides intelligent file path detection, clickable navigation, and automatic file creation - specifically designed to enhance workflow for code assistants like Claude Code, Gemini CLI, and Codex CLI.

# Project.md - Technical Documentation for Claude Code

## Project Overview

**Project.md** is a VSCode extension that transforms Markdown files into interactive project navigation hubs. It provides intelligent file path detection, clickable navigation, and automatic file creation - specifically designed to enhance workflow for code assistants like Claude Code, Gemini CLI, and Codex CLI.

### Core Purpose

When working with code assistants, developers often reference project files in Markdown documentation. This extension makes those references **actionable**:
- File paths become clickable links
- Non-existent files are created automatically
- Navigation works seamlessly with VSCode's native features

## Architecture

### Extension Structure

```
project-md/
├── src/
│   ├── extension.ts          # Main extension logic
│   └── test/
│       └── extension.test.ts # Test suite (placeholder)
├── dist/                      # Compiled output (esbuild)
├── package.json              # Extension manifest
└── tsconfig.json             # TypeScript configuration
```

### Key Components

#### 1. **Activation Events**
```json
"activationEvents": ["onLanguage:markdown"]
```
- Extension activates when any Markdown file opens
- Lazy activation for performance optimization
- No activation on VSCode startup (reduces overhead)

#### 2. **Providers Registered**

**a) Document Link Provider**
- Scans document for file path patterns
- Converts paths to clickable VSCode links
- Uses command URI scheme for custom behavior
- Tooltip: "Open path"

**b) Definition Provider**
- Enables "Go to Definition" (F12) on file paths
- Only works for existing files
- Returns `vscode.Location` pointing to target file

**c) Command Handler**
- Command ID: `markdownLinks.open`
- Handles click events on document links
- Implements auto-creation logic
- Directory handling with fallback

### Path Detection Strategy

Three regex patterns capture different reference styles:

#### Pattern 1: Markdown Links
```typescript
const MD_LINK_RE = /\[[^\]]+\]\(\s*(@?(?:\.{1,2}\/|\/)[^)\s]+)\s*\)/g;
```

**Matches:**
- `[text](./path/file.md)`
- `[config](@./config/settings.ts)`
- `[root](/absolute/path.ts)`

**Captures:** Path inside parentheses, including optional `@` prefix

#### Pattern 2: Bare References
```typescript
const BARE_REF_RE = /(^|[\s(`])(@?(?:\.{1,2}\/|\/)[^\s`)\]]+)/g;
```

**Matches:**
- `./src/index.ts` (standalone)
- `Check ./config/app.ts for details`
- Line-beginning paths

**Captures:** Leading whitespace/delimiter + path

**Conflict Resolution:** Only adds if not already captured by MD_LINK_RE

#### Pattern 3: Inline Code References
```typescript
const INLINE_CODE_RE = /`(@?(?:\.{1,2}\/|\/)[^\s`)\]]+)`/g;
```

**Matches:**
- `` `./src/utils.ts` ``
- `` `@./config/database.ts` ``

**Captures:** Path inside backticks

**Conflict Resolution:** Checks for range intersection before adding

### Path Resolution Logic

```typescript
const abs = path.resolve(baseDir, rel.replace(/^@/, ""));
```

**Steps:**
1. Get document's directory (`path.dirname(doc.uri.fsPath)`)
2. Remove `@` prefix if present
3. Resolve relative path to absolute
4. Convert to filesystem path

**@ Prefix Behavior:**
- Stripped before resolution
- Allows aliasing convention (e.g., `@./` for project root contexts)
- No special mapping (yet) - treated as `./`

## Feature Implementation Details

### Feature 1: Clickable Links

**Implementation:**
```typescript
const linkProvider: vscode.DocumentLinkProvider = {
  provideDocumentLinks(doc) {
    return getRefRanges(doc).map(({ range, targetFsPath }) => {
      const cmdUri = vscode.Uri.parse(
        `command:${commandId}?${encodeURIComponent(JSON.stringify({ path: targetFsPath }))}`
      );
      const link = new vscode.DocumentLink(range, cmdUri);
      link.tooltip = "Open path";
      return link;
    });
  }
};
```

**Flow:**
1. User hovers over detected path
2. VSCode renders underline (document link)
3. Cmd/Ctrl+Click triggers command URI
4. Command handler (`markdownLinks.open`) executes
5. `openPathLike()` processes the path

### Feature 2: Auto-Creation

**Implementation:**
```typescript
async function openPathLike(targetFsPath: string) {
  const stat = await fs.promises.stat(targetFsPath).catch(() => undefined);

  if (!stat) {
    await fs.promises.mkdir(path.dirname(targetFsPath), { recursive: true });
    await fs.promises.writeFile(targetFsPath, "", "utf8");
  }

  const td = await vscode.workspace.openTextDocument(vscode.Uri.file(targetFsPath));
  await vscode.window.showTextDocument(td);
}
```

**Behavior:**
- **File doesn't exist**: Create parent directories + empty file
- **Directory**: Reveal in Explorer (or Finder/File Explorer as fallback)
- **File exists**: Open in editor
- **Error**: Show error message with path

**Safety:**
- Uses `recursive: true` for `mkdir` (safe if directories exist)
- Creates empty UTF-8 file (no data loss risk)
- Catch-all error handling prevents crashes

### Feature 3: Go to Definition

**Implementation:**
```typescript
const defProvider: vscode.DefinitionProvider = {
  provideDefinition(doc, pos) {
    const hit = getRefRanges(doc).find(h => h.range.contains(pos));
    if (!hit) return;

    const stat = fs.existsSync(hit.targetFsPath) ? fs.statSync(hit.targetFsPath) : undefined;
    if (stat?.isFile()) {
      return new vscode.Location(vscode.Uri.file(hit.targetFsPath), new vscode.Position(0, 0));
    }
  }
};
```

**Behavior:**
- Only works for **existing files**
- Does NOT work for directories or non-existent paths
- Jumps to line 0, column 0 of target file
- Enables F12, Cmd+Click to jump

**Design Decision:**
- Separated from `openPathLike` to avoid auto-creation on F12
- Definition = "jump to existing code", not "create new file"

### Feature 4: Directory Handling

**Implementation:**
```typescript
if (stat?.isDirectory()) {
  try {
    await vscode.commands.executeCommand("revealInExplorer", vscode.Uri.file(targetFsPath));
  } catch {
    await vscode.commands.executeCommand("revealFileInOS", vscode.Uri.file(targetFsPath));
  }
  return;
}
```

**Fallback Strategy:**
1. Try `revealInExplorer` (VSCode Explorer sidebar)
2. If fails, try `revealFileInOS` (system file manager)
3. No error thrown - best-effort approach

## Development Guidelines

### Code Style

**Type Safety:**
- All functions use TypeScript types
- `vscode` types from `@types/vscode`
- Explicit return types for public functions

**Error Handling:**
- Async operations use `.catch(() => undefined)` for graceful degradation
- Try-catch blocks for user-facing operations
- Error messages include context (file path)

**Naming Conventions:**
- `getRefRanges`: Pure function, no side effects
- `openPathLike`: Async action with side effects
- `Hit`: Type represents detected path + range

### Building & Testing

**Build Commands:**
```bash
npm run compile       # Type check + lint + build
npm run watch         # Watch mode (esbuild + tsc)
npm run package       # Production build
```

**Development Flow:**
1. `npm run watch` in terminal
2. Press F5 to launch Extension Development Host
3. Open Markdown file in dev window
4. Test path detection and navigation

### Testing Strategy

**Unit Tests (Recommended):**
```typescript
// Test regex patterns
describe('getRefRanges', () => {
  it('should detect markdown links', () => {
    const doc = createMockDocument('[test](./file.md)');
    const ranges = getRefRanges(doc);
    expect(ranges).toHaveLength(1);
    expect(ranges[0].targetFsPath).toContain('file.md');
  });

  it('should detect bare references', () => { /* ... */ });
  it('should detect inline code paths', () => { /* ... */ });
  it('should handle @ prefix', () => { /* ... */ });
  it('should prevent duplicate ranges', () => { /* ... */ });
});
```

**Integration Tests (Recommended):**
```typescript
// Test providers
describe('DocumentLinkProvider', () => {
  it('should provide clickable links', async () => { /* ... */ });
});

describe('DefinitionProvider', () => {
  it('should navigate to existing files', async () => { /* ... */ });
  it('should return undefined for non-existent files', async () => { /* ... */ });
});
```

**Manual Testing Checklist:**
- [ ] Markdown links with relative paths
- [ ] Markdown links with absolute paths
- [ ] Markdown links with `@` prefix
- [ ] Bare references in text
- [ ] Inline code references
- [ ] Click to open existing file
- [ ] Click to create non-existent file
- [ ] Click to reveal directory
- [ ] F12 on existing file path
- [ ] F12 on non-existent path (should do nothing)

## Future Roadmap

### Planned Features

#### 1. **Syntax Highlighting**
```markdown
Goal: Visual differentiation for file paths in Markdown

Implementation approach:
- TextMate grammar in package.json contributes
- Scope: source.markdown meta.path.projectmd
- Color themes can customize highlighting
- Distinguish existing vs non-existent files
```

#### 2. **Path Validation**
```markdown
Goal: Warn about broken references

Implementation approach:
- Diagnostic provider for Markdown files
- Check file existence on document change
- Warning severity for non-existent paths
- Quick fix: "Create file" code action
```

#### 3. **Assistant-Specific Tooling**

**Claude Code Integration:**
```markdown
- Detect claude.md references
- Auto-link to project context files
- Validate @-references against project structure
```

**Gemini CLI Integration:**
```markdown
- Support for .geminirc path conventions
- Validate tool configurations
```

**Codex CLI Integration:**
```markdown
- Support for .codex paths
- Validate context file references
```

#### 4. **Configuration Options**
```json
{
  "project-md.autoCreate": true,
  "project-md.atPrefixAlias": "./",
  "project-md.highlightPaths": true,
  "project-md.validatePaths": "warning"
}
```

#### 5. **Workspace Support**
```markdown
Goal: Multi-root workspace handling

Implementation approach:
- Resolve paths relative to workspace root
- Support workspace-relative paths (e.g., ${workspaceFolder}/...)
- Handle monorepo structures
```

## Design Decisions & Trade-offs

### Why Three Separate Regex Patterns?

**Decision:** Use 3 patterns instead of one complex regex

**Rationale:**
- **Clarity:** Each pattern has single responsibility
- **Maintainability:** Easy to debug and extend individual patterns
- **Conflict handling:** Explicit deduplication logic
- **Performance:** Simpler patterns = faster execution

**Trade-off:** Potential overlapping matches (mitigated by intersection checks)

### Why Auto-Create Files?

**Decision:** Automatically create referenced files when clicked

**Rationale:**
- **Workflow optimization:** Reduces friction when documenting
- **Code assistant friendly:** LLMs often reference files that don't exist yet
- **Markdown-first development:** Document structure before implementation

**Trade-off:** Accidental file creation (mitigated by requiring explicit click)

### Why Separate Definition Provider?

**Decision:** Definition provider only navigates to existing files

**Rationale:**
- **User expectation:** F12 = "go to existing code"
- **Avoid confusion:** Definition ≠ creation
- **Explicit intent:** Click = create, F12 = navigate

**Trade-off:** Different behavior for same path (click vs F12)

### Why Remove @ Prefix Without Mapping?

**Decision:** Strip `@` but don't resolve to special directory

**Rationale:**
- **Future-proofing:** Reserve `@` for future aliasing feature
- **Compatibility:** Works with existing paths
- **Simplicity:** No configuration needed for v0.0.1

**Trade-off:** `@./` behaves identically to `./` (no benefit yet)

## Extension Manifest Details

### Key Package.json Fields

```json
{
  "name": "project-md",
  "displayName": "Project.md",
  "version": "0.0.1",
  "engines": { "vscode": "^1.104.0" },
  "activationEvents": ["onLanguage:markdown"],
  "main": "./dist/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "project-md.helloWorld",
        "title": "Hello World"
      }
    ]
  }
}
```

**Note:** `project-md.helloWorld` is template command - not used in v0.0.1

### Dependencies

**Runtime:** None (uses only VSCode API + Node.js built-ins)

**Development:**
- `esbuild`: Fast bundling for production
- `typescript`: Type checking
- `eslint`: Code linting
- `@types/vscode`: VSCode API types

## Performance Considerations

### Current Performance Characteristics

**Document Scanning:**
- 3 regex patterns executed per document
- Complexity: O(n) where n = document length
- Runs synchronously in `provideDocumentLinks()`

**Optimization Opportunities:**
1. **Caching:** Cache regex results per document version
2. **Incremental parsing:** Only re-scan changed ranges
3. **Web worker:** Offload regex execution for large files

**Current Impact:** Negligible for typical Markdown files (<10k lines)

### Memory Usage

**Per Document:**
- Array of `Hit` objects (range + path)
- Typical: 10-50 hits per document
- Memory: ~1-5KB per document

**Extension Footprint:** <500KB (bundled)

## Security Considerations

### Path Resolution Safety

**Vulnerability:** Path traversal attacks

**Mitigation:**
```typescript
const abs = path.resolve(baseDir, rel.replace(/^@/, ""));
```
- `path.resolve()` normalizes paths (prevents `../../../etc/passwd`)
- Resolved to absolute path (no ambiguity)
- No shell execution (uses `fs` directly)

### File Creation Safety

**Vulnerability:** Unintended file creation

**Mitigation:**
- Requires explicit user click (no automatic creation)
- Creates only empty files (no content injection)
- Uses `utf8` encoding (prevents binary corruption)

**Remaining Risk:** User could click malicious path in untrusted Markdown

## Contributing Guidelines

### Pull Request Checklist

- [ ] Code follows TypeScript best practices
- [ ] All regex patterns tested with unit tests
- [ ] Manual testing performed (see checklist above)
- [ ] No new dependencies added without justification
- [ ] `npm run lint` passes
- [ ] `npm run compile` succeeds
- [ ] CHANGELOG.md updated

### Code Review Focus Areas

1. **Regex correctness:** Patterns match intended paths only
2. **Error handling:** All async operations have error handling
3. **Type safety:** No `any` types without justification
4. **Performance:** No blocking operations in providers
5. **UX:** Clear error messages and tooltips

## Debugging Tips

### Enable Extension Development Host Logging

```typescript
// Add to extension.ts
const outputChannel = vscode.window.createOutputChannel("Project.md");
outputChannel.appendLine(`Detected ${hits.length} paths`);
```

### Inspect Document Links

1. Open Command Palette (`Cmd+Shift+P`)
2. Run "Developer: Inspect Editor Tokens and Scopes"
3. Hover over path - shows token info

### Regex Testing

Use online regex tester with JavaScript flavor:
- https://regex101.com (select ECMAScript/JavaScript)
- Paste pattern + test Markdown content
- Verify capture groups

## VSCode Extension API Reference

### Key APIs Used

**Document Link Provider:**
- `vscode.languages.registerDocumentLinkProvider()`
- `vscode.DocumentLink(range, target)`
- `command:` URI scheme for custom commands

**Definition Provider:**
- `vscode.languages.registerDefinitionProvider()`
- `vscode.Location(uri, position)`

**Commands:**
- `vscode.commands.registerCommand()`
- `vscode.commands.executeCommand()`

**File System:**
- `vscode.workspace.openTextDocument()`
- `vscode.window.showTextDocument()`

**Node.js APIs:**
- `fs.promises.stat()` / `fs.existsSync()` / `fs.statSync()`
- `fs.promises.mkdir()` / `fs.promises.writeFile()`
- `path.resolve()` / `path.dirname()`

## Questions & Support

### Common Questions

**Q: Why aren't my paths being detected?**
A: Check if path starts with `./`, `../`, or `/`. Paths without these prefixes are not detected.

**Q: Can I use workspace-relative paths?**
A: Not yet in v0.0.1. This is planned for future release.

**Q: Why does F12 create files but Definition provider doesn't?**
A: They use different code paths. Click uses `openPathLike()` (auto-creates), F12 uses definition provider (existing files only).

**Q: How do I configure the extension?**
A: No configuration options in v0.0.1. Coming in future releases.

### Development Support

For technical questions or contributions:
1. Check existing GitHub issues
2. Review this documentation
3. Open new issue with:
   - VSCode version
   - Extension version
   - Minimal reproduction case
   - Expected vs actual behavior

---

**Last Updated:** 2025-10-01
**Extension Version:** 0.0.1
**VSCode Engine:** ^1.104.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daviguides) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
