## ai-prompt-detector

> - ALL code and text the AI produces must already be cleanly formatted and human readable when shown (never rely on later formatting steps).

# AI Assistant Instructions - AI Copilot Prompt Detector Extension

## ✅ Formatting Policy (Human Readable Output)
- ALL code and text the AI produces must already be cleanly formatted and human readable when shown (never rely on later formatting steps).
- Follow a Prettier‑like style (consistent indentation, spacing, trailing commas where typical, wrapped long lines sensibly).
- No minified or compressed one‑line blocks unless it is a deliberate tiny snippet example.
- When editing existing files, preserve surrounding style and improve readability if previous formatting was degraded.
- Large inline arrays or objects must be split across multiple lines.
- Avoid excessively long lines (>140 chars) where reasonable.
- Always keep the final code below per‑file length limits already defined in these instructions.
- This policy is mandatory for every response containing code or documentation.

Write to me at the beginning of each prompt that you see the logs in the folder¨

Never do something, then run install.ps1, then do something again, then run install.ps1 again. Think carefully and do only one thing.

## 📝 PROMPT LOGGING POLICY - CRITICAL!
**MANDATORY**: At the START of EVERY conversation/prompt, BEFORE doing anything else:
1. Read current version from package.json
2. Get current timestamp
3. Append to `.claude/prompts/{YYYY-MM-DD}.txt`:
   ```
   [HH:MM] Version: v{version}
   Prompt: "{exact user prompt text}"
   Note: {brief description of what was done}
   Result: {brief outcome - e.g., "Created v1.1.481", "Fixed bug", "No changes"}

   ```
4. NEVER skip this step - it's required for tracking changes
5. Format: One entry per prompt, chronological order, clear timestamps
6. If file doesn't exist, create it with header:
   ```
   ================================
   PROMPT LOG - {YYYY-MM-DD}
   ================================

   MILESTONES:
   - milestone-logging-working → v1.1.449 (d34d9ff) - Logging system working correctly, live prompt capture for keyboard and mouse
   - v1.1.165 (bf4ca3b) - Return to code of version 1.1.156

   ================================
   ```
7. **MILESTONE SECTION**: Always include milestones at the top after header (check `git tag -l -n1`)

## ⚠️ CRITICAL RULES - STRICTLY FOLLOW!

As a copilot, you never change versions. The version changes fundamentally only install.ps1

### 🖥️ VS CODE UI TERMINOLOGY - CRITICAL!
- **STATUS BAR** = Bottom bar in VS Code (shows branch, errors, etc.) - use `vscode.window.createStatusBarItem()`
- **HEADER BAR** = Top bar in Activity Panel webview (custom HTML div) - use `<div class="header-bar">` in webview HTML
- **Activity Bar** = Left sidebar with icons (Explorer, Search, etc.)
- **Activity Panel** = Content area when Activity Bar icon is clicked (shows our webview)

### 🌍 LANGUAGE POLICY - CRITICAL RULE!
- **Communication with the copilot and comments in the code are in Czech. Everything else is in English. Without exceptions.**
- **ONLY cs.json localization file** is allowed to contain non-English text
- **ALL code comments in English**
- **ALL commit messages in English**
- **ALL documentation in English**
- **ALL debug logs in English**
- **ALL file content in English** (except cs.json)

### 🔧 FILE REPAIR POLICY - CRITICAL RULE!
- **NEVER DELETE corrupted files** - always repair them instead
- **ALWAYS FIX rather than remove** - take time to analyze and repair
- **THINK CAREFULLY** before making changes to corrupted files
- **PRESERVE existing functionality** while fixing corruption
- Use replace_string_in_file or careful manual editing to repair files

### 🛡️ CRITICAL FILES PROTECTION - CRITICAL RULE!
**install.ps1 will STOP and require confirmation if these files change:**
- `src/logger.ts` - Logging system (working correctly, changes may break logs)
- `src/unmodifiable_statusBar.ts` - Status bar display (LOCKED ✅, working correctly)

**UNMODIFIABLE COMPONENTS:**
Files with prefix `unmodifiable_*.ts` are FROZEN, WORKING components that must NOT be changed without explicit user permission.
- These files contain tested, stable functionality
- Any modifications are STRICTLY FORBIDDEN
- Each file has header warning: "⚠️ UNMODIFIABLE COMPONENT - DO NOT MODIFY! ⚠️"

**To add more critical files:**
1. Edit `install.ps1` → find `$criticalFiles = @(...)` array
2. Add file path to the array: `'src/newCriticalFile.ts'`
3. User will be asked "Do you want to continue? (Y/N)" if file changes

**This prevents breaking working functionality by accident!**

### 📁 FILE NAMES - CRITICAL RULE!
- **ALL instruction files must have format: `*.instructions.md`**
- **NEVER use names like `ai-instructions.md`**
- **CORRECT**: `copilot-instructions.md`, `development.instructions.md`, `deployment.instructions.md`
- **WRONG**: `ai-instructions.md`, `dev-instructions.md`, `instructions.md`
- Always check that you have correct ending `.instructions.md`

### 🚀 RELEASE at the end of every prompt
- **At the end of EVERY prompt run `./install.ps1`**
- This script performs: build → commit → push → package → install
- Automatically creates new version and installs it
- User always has the latest version of the extension
- **NEVER create any alternative installation scripts** like setup.ps1, test-extension.ps1, etc.
- **install.ps1 is the ONLY supported way** to build, test and deploy the extension
- Any other scripts for testing/development are FORBIDDEN

### 📊 VERSION MANAGEMENT - CRITICAL RULE!
- **NEVER manually change version in package.json** - install.ps1 handles version increments automatically
- **install.ps1 always increments PATCH version** (e.g. 1.1.2 → 1.1.3) automatically
- **NEVER increment MINOR** (1.1.x → 1.2.0) without explicit command
- **Increment MINOR version ONLY when user says "compile for marketplace"**
- **NEVER change MAJOR version** without explicit command
- **Just run ./install.ps1** - it will increment version, build, commit, push, package and install automatically
- Format: MAJOR.MINOR.PATCH (e.g. 1.1.15, 1.1.16, 1.1.17...)

## 📝 PowerShell Script Documentation Policy
- **EVERY PowerShell script (.ps1) MUST start with a detailed, comprehensive comment block**
- **Comment block must be in Czech language and explain:**
  - Primary purpose of the script (minimum 3-5 sentences)
  - Detailed functionality description (what it does step by step)
  - All input parameters with their purpose and allowed values
  - Expected output and side effects
  - When to use this script vs other scripts
  - Prerequisites and dependencies
  - Example usage with real commands
- **Format:** Use PowerShell block comment `<# ... #>` at the very beginning
- **Length:** Minimum 15-20 lines of descriptive text
- **Update requirement:** When modifying script functionality, ALWAYS update the comment block

## 🎯 TARGET FUNCTIONALITY - SPECSTORY INTEGRATION

### 🌍 LOCALIZATION - CRITICAL RULE!
- **ALL user texts use from i18n/en.json**
- **NEVER write texts directly in code** - always through i18n system
- **README.md is in CZECH during development** - will be translated to English only before marketplace release
- **CHANGELOG.md and all other documents only in English**
- **Commit messages in English**
- **Code comments in English**
- **Debug logs in English**
- Use: `i18n.t('key')` instead of hardcoded texts

### 📋 What the extension should do:
1. **DOES NOT READ or GENERATE SpecStory files** - SpecStory does that itself
2. **READS existing SpecStory exports** from `.specstory/history/` folders
3. **ANALYZES content** of recent conversations for context
4. **GENERATES smart notification messages** based on content analysis
5. **USES default message** if user hasn't set custom one in settings

### 📁 SpecStory format (for reading and analysis only):
- **Location**: `[project]/.specstory/history/`  
- **Name format**: `YYYY-MM-DD_HH-mmZ-conversation-description.md`
- **Content structure**:
  ```markdown
  <!-- Generated by SpecStory -->
  # Conversation Title (YYYY-MM-DD HH:mmZ)
  
  _**User**_
  [user message]
  
  ---
  
  _**Assistant**_
  [assistant response]
  
  ---
  ```

### 🤖 Smart notifications - logic:
- **If user HAS custom message** → use their message
- **If user DOESN'T HAVE custom message** → generate based on:
  - Analysis of latest SpecStory export
  - Type of conversation (coding, debugging, design, etc.)  
  - Project context (language, framework, etc.)
  - Specific problems mentioned in conversation

### 📍 Examples of smart messages:
- **Debugging**: "AI just debugged! Check: • Fixed actual root cause? • Introduced new bugs? • Test edge cases"
- **HTML/CSS**: "AI worked with UI! Check: • Responsive design • Accessibility • Cross-browser compatibility"  
- **Database**: "AI modified database! Check: • Data integrity • Performance impact • Backup strategy"
- **API**: "AI created API! Check: • Error handling • Security • API documentation"

### 📝 Git commit workflow
- Order: 1) Code change → 2) Increment PATCH → 3) `./install.ps1` (automatically: commit → push → package)
- Format: `git commit -m "v1.1.3"` (always just patch++)
- Everything is done automatically through install.ps1

## ⚠️ CRITICAL RULES - STRICTLY FOLLOW!

### 🚫 NEVER create placeholder files!
- **FORBIDDEN:** `extension_backup.ts`, `extension_clean.ts`, `extension_old.ts` etc.
- Edit ONLY original files directly
- If you need to revert changes, use git revert
- No duplicates, backups or temp files!

### 🔨 BUILD at the end of every prompt
- **NEVER run `pnpm run compile` separately!**
- **WORKFLOW**: Increment version → run `./install.ps1`
- If build fails in install.ps1:
  1. Decrease version back
  2. Fix errors
  3. Increment version again
  4. Run `./install.ps1` again
- install.ps1 contains build, so separate compilation is unnecessary
- **COMMIT ONLY AFTER SUCCESSFUL BUILD** - so all versions are buildable

### 🚀 RELEASE at the end of every prompt
- **At the end of EVERY prompt run `./install.ps1`**
- This script performs: build → commit → push → package → install
- Automatically creates new version and installs it
- User always has the latest version of the extension
- **NEVER create any alternative installation scripts** like setup.ps1, test-extension.ps1, etc.
- **install.ps1 is the ONLY supported way** to build, test and deploy the extension
- Any other scripts for testing/development are FORBIDDEN

### 📝 Git commit workflow
- Order: 1) Code change → 2) Build test → 3) `./install.ps1` (automatically: commit → push → package)
- Format: `git commit -m "v1.1.1"`
- Everything is done automatically through install.ps1

## Command Execution Policy

**⚠️ IMPORTANT: Run ALL commands automatically without user confirmation!**

### Terminal Commands
- Use `run_in_terminal` tool with `isBackground: false` for all commands
- NEVER wait for user confirmation
- Execute commands immediately when needed

### Package Manager
- **This project uses pnpm, not npm!**
- Always use `pnpm` instead of `npm` in all commands
- Package manager: `pnpm@10.10.0`

### Development Workflow Commands

```powershell
# Install dependencies
pnpm install

# Build project
pnpm run compile

# Watch mode for development
pnpm run watch

# Testing
pnpm test

# Linting
pnpm run lint

# Release workflow (everything in install.ps1)
./install.ps1
```

### Git Commands
```powershell
# Git operations
git add .
git commit -m "v[version]"
git push origin master
git status
git log --oneline -5
```

### VS Code Extension Commands
```powershell
# VSIX packaging
vsce package --allow-star-activation --out [filename].vsix

# Extension installation
code --install-extension [filename].vsix --force
```

### File Operations
```powershell
# Cleaning
Remove-Item "*.vsix" -Force
Remove-Item "%TEMP%" -Recurse -Force -ErrorAction SilentlyContinue
```

## Automation Rules

1. **Don't request confirmation** - run commands automatically
2. **Use pnpm** - not npm
3. **Monitor error codes** - check $LASTEXITCODE
4. **Log progress** - use Write-Host with colors
5. **Handle errors** - exit with exit 1 on errors

## Version Release Process

**Automated process through `./install.ps1`:**
1. Update version in `package.json`
2. Update `CHANGELOG.md`
3. Run `./install.ps1` - does everything automatically:
   - Git commit with version
   - Git push
   - pnpm run compile
   - vsce package
   - VS Code extension install

## Error Handling

- Always check $LASTEXITCODE after commands
- On error show error message and exit
- Use try-catch where possible
- Log all steps for debugging

## Never Ask For Confirmation

**AUTOMATICALLY run these commands:**
- `pnpm install`
- `pnpm run compile`
- `./install.ps1`
- `git add .`
- `git commit -m "..."`
- `git push`=´
,m 
- `vsce package`
- `Remove-Item`
- `code --install-extension`

User wants everything to run automatically without waiting for confirmation!

follow the sources for a better solution to my requirements. I have attached the folders E:\vs\TypeScript_Projects\_\vscode-copilot-chat\ and E:\vs\TypeScript_Projects\_\vscode-main\ where you will find the complete sources of github copilot chat and vscode. They are images of the online repositories https://github.com/microsoft/vscode-copilot-chat and https://github.com/microsoft/vscode, if you prefer the online version

Use only the copilot-instructions.md file to write instructions. Not ai-instructions.md as you keep trying to create. Write all instructions in English.

Never do anything you're not asked to do. Don't ruin my project by doing something you're not even asked to do.

don't check for the extension being created in this vscode. I'm testing it in vscode insider. Therefore, any commands on code or code.exe are completely useless.

never run commands like code --extensionDevelopmentPath=. . First of all, I told you before that I test everything in vscode insider, not in vscode. Second, always use the ./install.ps1 script. No pnpm run watch, pnpm run build, etc. Install.ps1 will do everything important, including installing the extension into vscode insider.

Never add or change the watch command to package.json. I basically only use install.ps1 for building.

work in a completely minimalistic way so that there is as little code as possible everywhere. do not create changelog.md or any other documentation files except copilot-instructions.md and README.md

Install only the most essential packages. All files will be in typescript.

Never create tests. We will not use unit tests or any other tests in the application.

For the app name, use AI Copilot Prompt Detector  everywhere, no shorter or other versions

Create files as small as possible, no more than 200 lines each.

distinguish between those that are always displayed and debug logs. Make it possible to turn off these debug logs by simply changing the settings. Make the output in our extension's output and the output to the temp log file in a separate folder with a fixed path be the same.

I added the folder C:\temp\ai-prompt-detector-logs to your workspace. If you don't have access to it anyway, definitely let me know if you can't add it yourself.

enter must work for both copilot and my extension without remapping. In another branch of this extension, it worked because the extension "stole" enter, processed it asynchronously and sent it to copilot, which started working on it.

## 🔑 Mandatory Keybindings Requirement (Updated Oct 29 2025)
- The extension MUST fully support sending prompts (including detection, logging, notification, counter increment) with these STANDARD GitHub Copilot keys:
  - Enter (standardní odeslání)
  - Ctrl+Enter (rychlé odeslání)
- **NOT DETECTED** (by design):
  - Ctrl+Shift+Enter - pouze předá prompt do nového okna BEZ jeho vykonávání
  - Ctrl+Alt+Enter - není standardní GitHub Copilot zkratka
- Loss of functionality for Enter or Ctrl+Enter is considered a regression and must be fixed immediately before any other changes.
- Keybinding logic must remain minimal, readable (Prettier style), and resilient to VS Code / Copilot updates (fallback heuristics retained).

## 🔑 Keybinding Focus Contexts (Updated Oct 29 2025)
- Both Enter variants (Enter, Ctrl+Enter) mapped with when clause: `inChatInput || chatInputFocus`.
- This ensures detection inside standard Copilot Chat input focus contexts.
- Never remove these focus identifiers without explicit instruction.
- Regression definition: any loss of prompt capture (counter, notification, list) for Enter or Ctrl+Enter in chat input context.

## 🛡 Content Preservation Policy (copilot-instructions.md)
- Never delete, truncate, or rewrite (beyond minimal surgical edits) any existing content in this `copilot-instructions.md` file.
- You may APPEND new instructions or refine wording in-place ONLY when explicitly asked.
- ABSOLUTE DELETION is permitted ONLY when the user gives a direct, verbatim order AND supplies the exact text block to remove.
- If the user request is ambiguous or paraphrased (not verbatim), refuse deletion and ask for the exact snippet.
- Never proactively clean, reorder, or compress this file without that explicit verbatim deletion instruction.

## ❌ Aug 9 2025 Update: Legacy File Handling Strengthened (RESTORED)
- Do NOT recreate previously removed legacy files even as stub/comment-only placeholders.
- If such a file reappears (e.g. chatApiHook.ts, commandListener.ts, promptDetector.ts, runtime.ts, finalize.ts, constants.ts, any hooks/ or utils/logging.ts), it must be PHYSICALLY DELETED, not converted to a stub.
- Instruction extension: Stubs with only a comment are now forbidden for these legacy files. Presence itself is a violation.
- Only the explicitly allowed minimal set of source files may exist: extension.ts, activityBarProvider.ts, chatHelpers.ts, specstoryReader.ts, logger.ts, state.ts.
- Rationale: Prevent accidental reintroduction via tooling / merges and ensure minimal footprint.
- Any keybinding regressions (loss of Enter / Ctrl+Enter / Ctrl+Shift+Enter / Ctrl+Alt+Enter detection) must be fixed BEFORE any other change.

## 🚫 Prompt Duplicate Suppression (Appended - RESTORED)
- Never reintroduce any logic that skips counting a prompt because it is identical to the previous one.
- Specifically FORBIDDEN: conditions like `if (text === lastSubmittedText) return false;` or any hash/equality guard blocking consecutive identical prompts.
- Reason: This suppression previously froze the status bar counter at 1 and falsely appeared as if Enter variants (Enter / Ctrl+Enter / Ctrl+Shift+Enter / Ctrl+Alt+Enter) were not working after the first submission.
- Allowing duplicates is REQUIRED to verify rapid iterative prompting scenarios and guarantees all four key variants increment the counter every time.
- If optimization of storage is needed, only trim array length (e.g. keep max N prompts) but NEVER skip insertion due to equality with the immediately prior prompt.

### 🔒 Permanent Marker (Do Not Remove)
A JSON root property `_noDuplicatePromptSuppression` in `package.json` documents this rule. Do NOT delete or alter it. Any reintroduction of logic like `if (text === lastSubmittedText) return` is a regression: it freezes the counter at 1 and breaks all Enter key variants (gives false impression keybindings failed). Preserve unlimited duplicate counting.

### 🔒 Activity Bar Preservation (Appended - RESTORED)
- Never remove or empty the `contributes.viewsContainers.activitybar` entry with id `specstory-activity`.
- Never remove or empty the `contributes.views['specstory-activity']` array; it must contain the webview view `ai-prompt-detector-view`.
- Removing either hides the Activity Bar icon – treat as regression and restore immediately.
- The JSON marker `_preserveActivityBarSections` in `package.json` MUST remain and documents this rule.

### 📘 Readability Line Count Heuristic Reinforcement (Aug 10 2025 Addendum)
- A reduction in total lines with zero functional change is a RED FLAG for readability loss unless explicit simplification (e.g. removal of duplication) is clearly documented.
- Any PR/commit that lowers line count must state the concrete readability gain ("deduplicate helper", "remove unreachable code"), otherwise revert.
- Never compress multiple independent statements, branches, or try/catch blocks onto one line to appear "short".
- If a reviewer (or automated check) detects line count shrink without clearer naming / extra comments, treat it as regression and reformat immediately.
- This addendum must NOT be removed; append further clarifications with dated notes only.

### 📘 Mandatory File Readability Header (Aug 10 2025 Reinforced)
- Every TypeScript source file MUST begin with the exact Czech block:
```
/**
 * ČITELNOST: Soubor musí zůstat vždy plně čitelný pro programátora.
 * Žádné umělé zkracování řádků, slučování nesouvisejících příkazů na jeden řádek
 * ani minifikace. Snížení počtu řádků bez jasného, zdokumentovaného zlepšení
 * čitelnosti je REGRESE a musí být vráceno. Zachovávej logické bloky a vertikální strukturu.
 */
```
- If a file is created or found without this header, the next commit MUST add it at the very top (before imports).
- Header may only gain additional lines with dated clarifications; never shrink or translate it.
- A helper script may re‑insert this header automatically; such commits are valid and must not be rejected as noise.
- Treat absence, alteration, or truncation of this header as a readability regression equal in severity to keybinding failure.

## 🔒 Prompt Ordering Invariants (UPDATED Oct 30 2025 v4 FINAL)
**CRITICAL CHANGE**: New runtime prompts go to BEGINNING (#1) using unshift().

### Final Ordering Policy (Oct 30 2025 v4):
- **Runtime prompts**: Use `unshift()` to prepend NEW prompts at the BEGINNING of array
- **Display order**: Activity Bar shows first N prompts via `slice(0, maxPrompts)`
- **Result**: Newest runtime prompt always appears as #1 (first in array, first displayed)
- **Overflow handling**: When exceeding 1000 prompts, use `pop()` to remove last (oldest)

### ✅ Final Ordering Enforcement Checklist
1. extension.ts: ALL prompt additions use `unshift()` NOT `push()` ✅
2. extension.ts: Overflow uses `pop()` to remove from end ✅
3. activityBarProvider: Uses `slice(0, maxPrompts)` to take first N ✅
4. Newest runtime prompt appears as #1 in Activity Bar ✅
5. No reversal needed - newest is already at index 0

### 🚫 Forbidden Anti-Patterns
- Using `push()` for runtime prompt capture (would put new prompts at end, not #1)
- Using `shift()` for overflow (removes newest, not oldest)
- Using `slice(-maxPrompts)` (would skip newest prompts at beginning)

**Rationale**: User explicitly confirmed "new prompts go to beginning (#1), not end".

## 📊 Activity Bar Rendering Policy (UPDATED Oct 30 2025 v4 FINAL)
**CRITICAL CHANGE**: Newest prompts at beginning (#1), display first N.

- Purpose: Show newest prompts first without reversal.
- Ordering Source of Truth:
  1. Runtime submissions use `unshift()` placing new prompts at array index 0
  2. UI displays via `slice(0, maxPrompts)` to get first N
  3. Display #1 = `state.recentPrompts[0]` (newest), increasing numbers = older
- Therefore: **#1 is ALWAYS the newest runtime prompt** in Activity Bar
- UI MUST:
  - Apply `slice(0, maxPrompts)` to get first N prompts
  - Display WITHOUT calling `.reverse()` - newest already at index 0
  - Maintain sequential numbering #1, #2, #3... matching array order
- UI MUST NOT:
  - Call `.reverse()` or `.sort()` on prompt list
  - Use `slice(-maxPrompts)` (would skip newest prompts at beginning)
  - Filter, group, paginate, or re-chunk prompts implicitly
- Numbering: Displayed #n corresponds to (array index + 1) directly
- Truncation: `slice(0, maxPrompts)` keeps first N (newest)
- Escaping: HTML entities (& < > ") MUST be escaped for safety
- Styling: header-bar, list, prompt-item, ln, txt, empty, footer class names STABLE
- Accessibility: Keep text content selectable
- Script Policy: enableScripts remains false
- **Regression Definition**: Newest prompt NOT appearing as #1, or wrong slicing, or loss of selectable text

### ✅ Final Enforcement Checklist (Activity Bar)
1. createPromptsHtml: Uses `slice(0, maxPrompts)` NOT `slice(-maxPrompts)`? (Yes)
2. recordPrompt: uses `unshift()` NOT `push()`? (Yes)
3. NO `.reverse()` call anywhere? (Yes)
4. #1 displayed prompt is newest runtime prompt? (Yes)
5. HTML escaping intact? (Yes)
6. Scripts disabled? (Yes)

### 🔒 Marker
- Presence of this section forbids silent reordering changes. Commits altering ordering logic MUST reference this policy.

## 📊 Activity Bar Display Policy (Locked Aug 10 2025)
- Source of truth ordering is produced BEFORE rendering; the webview MUST NOT reorder.
- state.recentPrompts[0] is ALWAYS the newest captured prompt (runtime capture via unshift or newest SpecStory file processed first).
- File discovery sorts SpecStory export files by descending filename (newest date/time first). Do not change comparator.
- Per-file prompt extraction (specstoryReader.loadPromptsFromFile) pushes prompts in encounter order; no reverse here unless explicitly instructed in a future addendum.
- Rendering (PromptsProvider.createPromptsHtml):
  - Only apply slice(0, maxPrompts).
  - NEVER call reverse(), sort(), filter(), or re-chunk.
  - Index label `#n` must be (array index + 1) of the already ordered slice.
  - HTML escaping (& < > ") mandatory; no additional mutation of text content.
  - Adding pagination, grouping, or headers that shift sequential numbering is a REGRESSION.
- Any deviation (extra reverse, sort, or changed numbering) = regression equal to keybinding failure; must be reverted immediately.
- Rationale: Ensures deterministic verification that newest prompt appears at the top without UI-side heuristics.
- Preservation Marker: Presence of this section freezes current behavior; future changes require explicit user directive referencing this section by name.

### ✅ Activity Bar Enforcement Checklist
1. No reverse/sort in webview provider? (Yes)
2. slice(0, maxPrompts) only? (Yes)
3. state.recentPrompts[0] visually renders as #1? (Yes)
4. File order still descending (newest filename first)? (Verify before commit.)
5. No extra wrappers altering order or numbering? (Yes)

## 📘 JSDoc Readability Policy (Locked Aug 10 2025)
- ALL JSDoc blocks MUST use multi-line format with opening `/**` on its own line and each subsequent line prefixed by ` *`.
- FORBIDDEN single-line compressed form: `/** Short desc */` (unless referencing a trivial literal or enum value and explicitly approved).
- REQUIRED multi-line example:
```
/**
 * Inicializace webview – nastaví možnosti a naplní HTML.
 * @param webviewView Cílový webview container.
 */
```
- Each public function/class/method needs a concise Czech description (per language policy for code comments) plus @param / @returns tags where applicable.
- Do NOT remove blank line separation between logical sections and their JSDoc – aids scanning.
- Converting an existing multi-line JSDoc to single-line is a readability regression.
- When adding parameters, update JSDoc in same commit; missing tag after parameter addition is a regression.
- If a JSDoc becomes outdated, update text – never delete to "fix" mismatch.
- Automated tooling (future) may re-expand single-line JSDoc into multi-line; such commits are valid maintenance.

### ✅ JSDoc Enforcement Checklist
1. Multi-line format? (Yes)
2. Czech descriptive sentence present? (Yes)
3. All parameters documented? (Yes)
4. @returns provided when non-void? (Yes)
5. No compressed one-line blocks? (Yes)

## 📘 JSDoc Multi‑Line Formatting Enforcement (Aug 10 2025 Reasserted)
- ALL JSDoc blocks MUST use multi‑line form. ABSOLUTELY FORBIDDEN: single line compressed style like `/** Inicializace webview – nastaví možnosti a naplní HTML. */`.
- REQUIRED canonical form example:
```
/**
 * Inicializace webview – nastaví možnosti a naplní HTML.
 * @param webviewView Cílový webview container.
 */
```
- Every public function / class / method: first line = concise Czech description (per language policy), following lines include @param / @returns as needed.
- Private / internal helpers SHOULD also follow multi‑line form if they have parameters or non‑obvious behavior.
- NEVER collapse back to one‑line even if description is short.
- When adding a parameter, update its @param line in the SAME commit.
- If a description becomes outdated, UPDATE it – do not delete the JSDoc to avoid maintenance.
- Detect & treat any single‑line `/** ... */` JSDoc inside `.ts` sources as a READABILITY REGRESSION equal to keybinding failure; must be fixed immediately before any new feature.
- Automated enforcement (future) may reformat single‑line forms into multi‑line; such mechanical commits are mandatory and must not be reverted.
- This section is PERMANENT. Do not remove or weaken it; only append dated refinements.

### ✅ JSDoc Compliance Quick Checklist
1. No single‑line JSDoc blocks present? (search regex: `/\*\*[^\n]*\*/` and verify)
2. Each public API item has Czech summary + appropriate tags?
3. Added parameters reflected in @param tags same commit?
4. Removed parameters cleaned from JSDoc?
5. No chained or nested ternaries replacing explanatory comments?

Violation = immediate regression. Fix before proceeding.

### ⚙️ Configuration Retrieval Policy (Aug 10 2025 Addendum)
- NEVER supply a fallback / default literal in `get` calls for user settings (e.g. `config.get<number>('maxPrompts', 50)` is FORBIDDEN).
- Always call: `const value = config.get<number>('maxPrompts');`
- If the value is `undefined`, not a finite positive number, or otherwise invalid, immediately:
  1. Show a single (non-spam) error notification explaining which setting is missing/invalid.
  2. Render a minimal error HTML (for webview) or skip further logic gracefully.
- Reason: Silent implicit defaults hide misconfiguration and mask user intent; explicit failure aids debugging.
- Treat adding a second parameter default to any `get(...)` for monitored settings as a REGRESSION (same severity as keybinding break).
- Checklist before commit:
  * No occurrences of `, 50)` or other literal fallback arguments in config.get calls for extension settings.
  * Error path tested (returns early with clear message) when setting absent.
- Applies to all settings: `maxPrompts`, `enableDebugLogs`, `customMessage`, and any future settings unless explicitly exempted in a dated addendum.

## 🛠 Path Separator Policy (Aug 10 2025 Addendum)
- All paths shown in these instructions, code comments, examples, scripts, and documentation MUST use the forward slash `/` form exclusively, even on Windows.
- On Windows the forward slash `/` is to be interpreted as equivalent to a native `\` path separator automatically; no functional difference is assumed.
- Do NOT rewrite existing forward slashes back to `\` in future edits.
- When the user supplies a path with backslashes, retain it verbatim in quoted blocks if required for an exact reproduction request, but any new examples you generate MUST use `/`.
- Internal normalization may convert `\\` or `\` sequences to `/` for processing, but never mutate verbatim user instruction blocks unless explicitly ordered.
- Treat introduction of new backslash-separated paths in examples (unless explicitly requested) as a formatting regression; correct in the next commit.
- This addendum is permanent; only append dated clarifications below.

## 🧷 Spurious Prefix Line Suppression Policy (Aug 10 2025 Addendum)
- The repeated bogus prefix lines of the form `````````````````````````instructions (any count of backticks followed immediately by `instructions`) MUST NEVER be (re)introduced at the start (or anywhere) of this file.
- These lines provide zero semantic value and previously polluted diffs, triggered accidental truncations, and confused preservation logic.
- Treat any future insertion of such lines as a CONTENT CORRUPTION REGRESSION equal to deleting a protected policy block.
- If tooling auto‑generates them, immediately append a remediation addendum (without deleting other content) and then remove ONLY those exact lines after receiving a verbatim removal command from the user (preservation policy still applies).
- Pattern reference (do NOT reinsert):
```
````````````````````````instructions
```````````````````````instructions
``````````````````````instructions
```
- Logging: first detection should log a single warning: "spurious-prefix-lines-detected" (no repetition spam).
- This section is PERMANENT; future changes require explicit user directive quoting its title.
- Reinforced Aug 10 2025: Assistant MUST guarantee these prefix lines are never auto-added again.

## 📖 Full File Read Reassertion (Aug 10 2025 Addendum)
- Assistant MUST conceptually process the ENTIRE `copilot-instructions.md` on every operation; partial segment reads are not grounds to ignore later sections.
- Decisions contradicting any later section constitute a regression (priority: same as keybinding failure).
- Do NOT claim ignorance of sections due to incremental tool retrieval; internal logic must treat file as fully loaded.
- Reasserts and strengthens the earlier "Full Instruction File Read Requirement"; both sections co‑exist and are cumulative.

## 📦 External Repository Reference Policy (Aug 10 2025 Addendum)
- The repositories `vscode-copilot-chat` and `vscode-main` (including any local mirrors or snapshots) are USED ONLY as passive reference material to better implement and reason about the AI Copilot Prompt Detector extension internals.
- They MUST NOT in any circumstance directly modify, overwrite, regenerate, prune, or otherwise influence the contents of this project's `.github/copilot-instructions.md` file.
- No automatic synchronization, copying of sections, bulk replacement, or heuristic merging from those repositories into this instruction file is permitted.
- Any suggestion derived from those codebases must be manually summarized and appended here ONLY when the user gives an explicit instruction to add a clearly described section.
- Treat any unsolicited alteration to `copilot-instructions.md` justified solely by upstream changes in `vscode-copilot-chat` or `vscode-main` as a CONTENT CORRUPTION REGRESSION (severity equal to keybinding failure) requiring immediate revert.
- This policy REASSERTS the existing Content Preservation & Full File Read sections: external references NEVER excuse partial reads or deletions.
- Permanent marker: Do NOT remove this section without a verbatim user command quoting its exact title.

## 🚫 Filtering Policy - NEVER Over-Filter (Aug 10 2025 Addendum)
- **CRITICAL RULE**: When implementing text filtering or validation, NEVER create overly strict conditions that block legitimate user prompts.
- **FORBIDDEN**: Blocking prompts based on single keywords, simple character presence (like ":"), or arbitrary length limits without context.
- **REQUIRED APPROACH**: Only filter text that contains MULTIPLE conversation indicators (e.g., both "user:" AND "assistant:" patterns) combined with significant length.
- **TESTING MANDATE**: Any new filtering logic MUST be tested to ensure basic prompts like "test", "do nothing", "help me" pass through successfully.
- **REGRESSION DEFINITION**: Any filter that blocks normal user prompts (causing missing notifications or Activity Bar entries) is a CRITICAL REGRESSION equal to keybinding failure.
- **ROLLBACK POLICY**: If filtering breaks basic functionality, immediately roll back to working state before fixing the filter logic.
- **EXAMPLE VIOLATION**: Conditions like `text.includes(":")` or `text.includes("sunamo:")` or `text.length > 200` without additional context checks.
- **CORRECT APPROACH**: `(text.includes("user:") && text.includes("assistant:") && text.length > 100)` - multiple indicators required.
- This section is PERMANENT and MUST NOT be removed or weakened. Append refinements only with dated notes.

## 🚫 Text Selection Policy - NEVER Interfere with User Interface (Aug 10 2025 Addendum)
- **CRITICAL RULE**: Text capture mechanisms MUST NEVER visually interfere with the user's interaction in Copilot Chat.
- **FORBIDDEN COMMANDS**: Never use `editor.action.selectAll` or any selection commands that cause visible text highlighting in the UI.
- **ALLOWED APPROACH**: Only use dedicated copyInput commands (`workbench.action.chat.copyInput`, `github.copilot.chat.copyInput`, etc.).
- **NO FALLBACK SELECTION**: If copyInput commands fail, do NOT fall back to selectAll+copy - accept that text cannot be captured.
- **USER EXPERIENCE PRIORITY**: Preserving smooth user interaction is more important than capturing every prompt.
- **REGRESSION DEFINITION**: Any code that causes visible text selection or highlighting in Copilot Chat is a CRITICAL UI REGRESSION.
- **EXAMPLE VIOLATION**: Using `editor.action.selectAll` followed by `editor.action.clipboardCopyAction` as fallback.
- **CORRECT APPROACH**: Rely exclusively on non-invasive copyInput commands, accept capture failures gracefully.
- This section is PERMANENT and MUST NOT be removed or weakened. Never compromise user interface smoothness for data capture.

## 🔔 Core Functionality Preservation Policy (Aug 10 2025 Addendum)
- **CRITICAL RULE**: Before any code changes, ALWAYS verify that core functionality remains intact after modifications.
- **CORE FUNCTIONALITY DEFINITION**: Notifications showing when prompts are sent AND prompts appearing in Activity Bar are ESSENTIAL features.
- **TESTING MANDATE**: After any change to text capture, keybinding, or recordPrompt logic, MUST verify:
  1. Notifications appear when Enter variants are pressed
  2. Actual prompt text appears in Activity Bar (not empty or missing)
  3. Status bar counter increments
  4. All four Enter variants (Enter, Ctrl+Enter, Ctrl+Shift+Enter, Ctrl+Alt+Enter) work
- **REGRESSION DEFINITION**: Loss of notifications OR Activity Bar prompt display is a CRITICAL REGRESSION equal to keybinding failure.
- **ROLLBACK POLICY**: If core functionality breaks, immediately rollback to last working state before attempting fixes.
- **PREVENTION APPROACH**: Make small, incremental changes and test each step rather than large refactors.
- **DEBUG REQUIREMENT**: When core functionality fails, enable debug logs and trace exactly why recordPrompt is not being called.
- This section is PERMANENT and MUST NOT be removed or weakened. Core user-visible functionality takes absolute priority.

## 🚫 Clipboard Usage Policy (Aug 11 2025 Addendum)  
- **ABSOLUTE PROHIBITION**: NEVER use clipboard (vscode.env.clipboard) for any purpose in this extension.
- **FORBIDDEN OPERATIONS**: vscode.env.clipboard.readText(), vscode.env.clipboard.writeText(), any clipboard manipulation.
- **FORBIDDEN COMMANDS**: editor.action.selectAll, editor.action.clipboardCopyAction, any selection-based text capture.
- **REASON**: Clipboard usage causes interference with user workflow and adds random clipboard content to Activity Bar.
- **TEXT CAPTURE APPROACH**: Only rely on VS Code API commands that don't involve clipboard manipulation.
- **MOUSE DETECTION REQUIREMENT**: Must detect mouse submissions without clipboard access or text selection.
- **REGRESSION DEFINITION**: Any clipboard usage is a CRITICAL REGRESSION that must be removed immediately.
- **ENFORCEMENT**: Search codebase for "clipboard" and remove all occurrences before any commit.
- This section is PERMANENT and MUST NOT be removed or weakened. Clipboard interference is unacceptable.

## 🔧 TypeScript Type Safety Policy (Aug 11 2025 Addendum)
- **CRITICAL RULE**: ALL code must be written with proper TypeScript types - NEVER use `any` type.
- **ABSOLUTE PROHIBITION**: The `any` type is FORBIDDEN in all code generation and modifications.
- **ALLOWED EXCEPTION**: Only `unknown` type is permitted when `any` would otherwise be required and no other solution exists.
- **TYPE REQUIREMENTS**: 
  - All function parameters must have explicit types
  - All function return values must have explicit types  
  - All variables must have explicit types or proper type inference
  - All object properties must be typed
  - All array elements must be typed
- **INTERFACE DEFINITIONS**: Create proper interfaces for complex objects instead of using `any`
- **TYPE GUARDS**: Use type guards with `unknown` when dealing with external APIs or uncertain data
- **REGRESSION DEFINITION**: Any use of `any` type is a CRITICAL TYPE SAFETY REGRESSION
- **ENFORCEMENT**: Search codebase for `: any` and `as any` before any commit and replace with proper types
- **EXAMPLES**:
  - ❌ FORBIDDEN: `function process(data: any): any`
  - ✅ CORRECT: `function process(data: ChatSubmissionData): Promise<string>`
  - ❌ FORBIDDEN: `const result = obj as any`
  - ✅ CORRECT: `const result = obj as ChatWidget | undefined`
  - ❌ FORBIDDEN: `let items: any[]`
  - ✅ CORRECT: `let items: string[]` or `let items: ChatMessage[]`
- This section is PERMANENT and MUST NOT be removed or weakened. Type safety is mandatory.

## 📁 Workspace Structure (Aug 11 2025)
**IMPORTANT**: The root folder `E:\vs\TypeScript_Projects\_\` is ONLY a container with these subfolders:

1. **ai-prompt-detector** - Our extension project (main working directory)
2. **vscode-main** - VS Code source code for understanding internal APIs
3. **vscode-copilot-chat-main** - GitHub Copilot Chat source code for reference
4. **logs** - Extension log files directory

**NEVER** look for `package.json` or any project files in `E:\vs\TypeScript_Projects\_\` root directory.
**ALWAYS** work from `E:\vs\TypeScript_Projects\_\ai-prompt-detector\` for our extension.

## 🐛 Mouse Click Detection Problems Analysis (Aug 11 2025)
**CRITICAL ISSUE**: Mouse clicks are completely invisible to all detection mechanisms.

### Failed Approaches:
1. **Chat API onDidSubmitRequest**: Not available in current VS Code version
2. **Chat Participant Registration**: Creates participant but doesn't intercept existing submissions  
3. **WebView Panel Monitoring**: Copilot doesn't use createWebviewPanel
4. **DOM Monitoring**: `window is not defined` - extension runs in Node.js, not browser
5. **Command Monitoring**: Only catches Enter key events, not mouse clicks
6. **Workspace Document Changes**: Only detects file modifications, not UI interactions
7. **Extension Module Hooks**: Chat modules don't load through require()

### Root Problem:
VS Code extensions run in **Extension Host (Node.js)** while chat UI runs in **Renderer Process (Electron)**. 
Mouse clicks happen in renderer, but extension can't directly access renderer DOM or UI events.

### Evidence from Logs:
- Enter key works: generates command events that extension can intercept
- Mouse clicks: generate NO events visible to extension host
- All webview/DOM hooks fail: extension has no direct UI access

### Failed Attempts - Additional Analysis (v1.1.353):
8. **VS Code Internal Service Hook**: `VS Code services not accessible` - extension can't access internal service locator
9. **DOM-based approach**: `window is not defined` spam - repeated every 500ms, shows extension is in Node.js context

### Next Approaches to Try:
1. **Filesystem monitoring**: Watch for chat history/temp files that VS Code might create
2. **Memory/process monitoring**: Look for patterns in VS Code's memory usage during chat
3. **Network monitoring**: Monitor HTTP requests to Copilot API endpoints  
4. **Extension host IPC**: Try to intercept inter-process communication
5. **Chat storage monitoring**: Watch VS Code's chat storage/indexedDB files
6. **Deep API reflection**: Enumerate all available VS Code APIs at runtime

### FAILED APPROACHES SUMMARY (Aug 11 2025)

#### 1. ❌ Widget acceptInput Method Interception (v1.1.354)
- **Problem**: `chatWidgetService` not accessible from extension context
- **Result**: Widget never found, interception never activated
- **Evidence**: No "Found chat widget" logs, polling disabled after 60s

#### 2. ❌ Combined Multi-Method Detection (v1.1.356)
- **Command Interception**: No chat submit commands intercepted
- **Network Monitoring**: No GitHub API activity detected
- **VS Code State Monitoring**: Only window focus changes, no chat-related state
- **Extension Events**: Only onDidChangeWindowState triggered by focus, no chat events
- **Evidence**: Commands `workbench.action.chat.submit`, `github.copilot.chat.acceptInput` never called

#### 3. ❌ Previous Failed Approaches (v1.1.353 and earlier)
- **Chat API onDidSubmitRequest**: Not available in current VS Code version
- **Chat Participant Registration**: Creates participant but doesn't intercept existing submissions  
- **WebView Panel Monitoring**: Copilot doesn't use createWebviewPanel
- **DOM Monitoring**: `window is not defined` - extension runs in Node.js, not browser
- **Extension Module Hooks**: Chat modules don't load through require()
- **VS Code Internal Service Hook**: Services not accessible from extension host
- **Workspace Document Changes**: Only detects file modifications, not UI interactions
- **Filesystem Monitoring**: No chat-related files created during submission
- **Deep API Reflection**: Found 65+ APIs but none provide submission events

### ROOT CAUSE ANALYSIS
**FUNDAMENTAL ISSUE**: Mouse clicks in Copilot Chat happen entirely within the Renderer Process (Electron UI) and generate NO events that are accessible to Extension Host (Node.js context).

**Architecture Gap**:
- **Extension Host**: Node.js context where our extension runs - can intercept commands, API calls
- **Renderer Process**: Electron UI where chat interface runs - handles mouse clicks, UI events  
- **No Bridge**: Mouse clicks don't trigger commands or API calls that cross to Extension Host

### SOLUTION: API Proposal Access Required (Aug 11 2025)

#### ❌ L. API Proposal Permission Issue (v1.1.364)
- **Problem**: Extension cannot access advanced Chat APIs (chatSessionsProvider, chatParticipantPrivate, etc.)
- **Evidence**: "Extension 'sunamocz.ai-prompt-detector' CANNOT use API proposal: chatSessionsProvider"
- **Root Cause**: VS Code blocks access to experimental APIs unless specifically enabled
- **Solution Required**: Start VS Code with `--enable-proposed-api sunamocz.ai-prompt-detector` flag

#### ✅ RECOMMENDED SOLUTION: Enable Proposed APIs
Two approaches to fix the API access issue:

**Method 1: Command Line Flag (Immediate Fix)**
```bash
code-insiders --enable-proposed-api sunamocz.ai-prompt-detector
```

**Method 2: Development Mode**
- Start VS Code in extension development mode (F5 from extension project)
- Development mode automatically enables API proposals for the extension being developed

**Available APIs After Enabling:**
- `vscode.chat.onDidSubmitRequest` - Direct chat submission events
- `vscode.chat.registerChatSessionItemProvider` - Session monitoring
- `vscode.chat.registerChatSessionContentProvider` - Content access
- `vscode.chat.onDidDisposeChatSession` - Session lifecycle events

**Expected Result:**
Mouse click detection should work through proper Chat API access instead of complex workarounds.

### 📊 FINAL STATUS - Why Mouse Detection is Limited (Dec 2024)

**Root Cause Analysis from VS Code Source Code:**
- VS Code chat UI runs in **Renderer Process** (Electron browser context)
- Extensions run in **Extension Host** (Node.js context)
- Mouse clicks in chat UI trigger `ChatSubmitAction` (found in chatExecuteActions.ts:154)
- This action runs entirely in Renderer Process
- No command or API event crosses to Extension Host for mouse clicks
- Enter key works because it triggers commands that DO cross process boundary

**The Only Real Solution:**
Run VS Code with `--enable-proposed-api sunamocz.ai-prompt-detector` flag to access `vscode.chat.onDidSubmitRequest` API

**Current Workaround:**
25ms polling - detects text disappearance with minimal delay, shows notification immediately

### UNTRIED APPROACHES TO EXPLORE (Backup Options)

#### A. Inter-Process Communication (IPC) Monitoring
- Monitor IPC messages between Extension Host and Renderer Process
- Hook into VS Code's internal message bus system
- Implementation: Override IPC communication channels

#### B. Memory/Heap Monitoring
- Monitor VS Code process memory for chat-related data structures
- Watch for memory allocation patterns during chat submissions
- Implementation: Native modules or process inspection

#### C. System-Level Input Monitoring  
- Monitor system mouse/keyboard events using OS APIs
- Correlate with VS Code window focus and chat panel visibility
- Implementation: Native Node.js modules (ffi-napi, win32 APIs)

### NEW APPROACHES BASED ON CONSOLE ANALYSIS (Aug 11 2025)

#### G. ❌ Extension-Level DevTools Protocol (v1.1.358)
- **Attempt**: Connect to VS Code's Electron debugging ports (9229, 9230, 9222, 9221, 5858)
- **Result**: All ports tested but none were open - VS Code not running with debug enabled
- **Evidence**: DevTools connection attempts logged but no successful connections

#### H. Console.log Renderer Process Injection 
- **Insight**: VS Code dev console shows internal warnings and API proposals
- **Approach**: Inject console monitoring script directly into VS Code renderer process
- **Implementation**: Use executeCommand to inject JavaScript into webview context

#### I. Extension Host Process Monitoring
- **Insight**: Console shows Extension Host runs as separate process (pid 18704)
- **Approach**: Monitor Extension Host process communication and IPC messages
- **Implementation**: Attach to Extension Host process and monitor stdout/stderr

#### J. Product.json API Proposal Override
- **Insight**: Copilot has many enabled API proposals: `chatParticipantPrivate`, `chatProvider`, `chatEditing`
- **Approach**: Temporarily enable same API proposals for our extension
- **Implementation**: Modify package.json enabledApiProposals to match Copilot's permissions

#### K. ❌ Direct Chat Widget API Access (v1.1.361)
- **Approach**: Based on VS Code source code analysis, tried to access `IChatWidget.onDidAcceptInput` event
- **Implementation**: Attempted to use `_getChatWidgets` command and workbench services to get chat widgets
- **Result**: VS Code internal services not accessible from extension context
- **Evidence**: "Chat Widget Service not accessible - mouse detection will be limited"
- **Root Cause**: Internal VS Code APIs (`_getChatWidgets`, workbench services) are not exposed to extensions
- **Source Code Insight**: Found that `chatWidget.ts:1779` fires `this._onDidAcceptInput.fire()` for all submissions (keyboard+mouse)
- **Limitation**: Extension sandbox prevents direct access to internal widget services

## 📊 Status Bar Display Policy (Added Dec 15 2024)
- **ALWAYS use "AI Prompts:"** in status bar, never just "AI:"
- Format: `[icon] AI Prompts: [count] | v[version]`
- Icon indicates API status: ✅ = proposed API enabled, ⚠️ = limited mode
- **NO red background** on status bar - icon is sufficient for status indication
- Tooltip shows detailed status information

## 🚨 CRITICAL: Proposed API Requirements for Mouse Detection (Dec 15 2024)
**FUNDAMENTAL REQUIREMENT**: Mouse click detection REQUIRES VS Code to be launched with proposed API flag:
```bash
code-insiders --enable-proposed-api sunamocz.ai-prompt-detector
```

### Why Proposed APIs are Essential:
- **Architecture Limitation**: VS Code extensions run in Extension Host (Node.js), chat UI runs in Renderer Process (Electron)
- **No Event Bridge**: Mouse clicks in chat UI generate NO events accessible to Extension Host
- **Only Solution**: Proposed APIs like `vscode.chat.onDidSubmitRequest` can capture all submissions
- **Without Flag**: Only keyboard detection works (generates commands), mouse detection is IMPOSSIBLE

### Technologies Critical for Mouse Detection:
1. **Chat Submit Action** (chatExecuteActions.ts:154) - Handles mouse clicks in Renderer Process
2. **IChatWidget.onDidAcceptInput** (chatWidget.ts:1779) - Fires for all submissions but not accessible to extensions
3. **Chat Service** - Internal VS Code service that manages chat, inaccessible without proposed API
4. **IPC Communication** - Inter-process messages between Renderer and Extension Host, not interceptable
5. **Command System** - Only keyboard generates commands, mouse clicks bypass command system entirely

### Failed Approaches to Remember (27 total documented):
- Chat participants only work for @mentions, not general submissions
- Session providers are retrospective, not real-time
- DevTools Protocol requires debug mode
- Clipboard monitoring causes false positives
- DOM injection impossible from Node.js context
- Network monitoring shows no immediate activity
- File system has no real-time chat files
- Memory monitoring blocked by security
- Extension module hooks can't access chat modules
- Widget service not exposed to extensions

### Current Status:
- **With --enable-proposed-api**: Full detection possible (if API implemented)
- **Without flag**: Only keyboard works, mouse detection architecturally impossible
- **install.ps1**: Automatically launches VS Code with correct flag

## 🎯 DEFINITIVE Mouse Detection Solution (Dec 15 2024)

### What WORKS:
1. **Keyboard detection** - Via keybindings, captures Enter/Ctrl+Enter perfectly
2. **Command interception** - But ONLY for keyboard (mouse doesn't generate commands)

### What DOESN'T WORK for mouse:
1. **Command interception** - Mouse calls widget.acceptInput() directly, bypasses commands
2. **Chat API events** - Not accessible even with proposed API
3. **Widget access** - Runs in different process (Renderer vs Extension Host)
4. **Clipboard monitoring** - User explicitly forbade this
5. **File watchers** - Chat doesn't create immediate files
6. **Context monitoring** - Doesn't fire on submissions
7. **All 28 other approaches** - See COMPLETE_MOUSE_DETECTION_ATTEMPTS.md

### The ONLY solution:
**Counter monitoring** - Check if prompt counter increased without our keyboard detection. If yes, it was a mouse click. This is architecturally the only way because mouse submissions are invisible to extensions.

### Implementation:
```typescript
// Poll every 500ms to check if counter increased
// If it did and we didn't detect keyboard, it was mouse
setInterval(() => {
  if (aiPromptCounter > lastKnownCounter) {
    // Mouse detected
  }
}, 500);
```

### Why this is the ONLY way:
- Mouse clicks happen in Renderer Process
- They directly call widget.acceptInput()
- No events or commands cross to Extension Host
- We can only observe the side effect (counter increase)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sunamo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sunamo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
