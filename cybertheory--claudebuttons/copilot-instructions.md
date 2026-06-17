## claudebuttons

> Add "Run on Claude Code" and "Run on Cowork" buttons to any web project.

# claudebuttons — Integration Skill

Add "Run on Claude Code" and "Run on Cowork" buttons to any web project.

## When to Use

Use this skill when you need to:
- Add a "Run on Claude Code" button to a website, README, or documentation page
- Add a "Run on Cowork" button that provides setup and run instructions
- Create deploy-style buttons that trigger Claude CLI commands or Cowork skills
- Integrate Claude-powered workflows into an existing web interface

## How to Add Buttons

### Step 1: Install

```bash
npm install claudebuttons
```

Or add via CDN script tag:

```html
<script src="https://unpkg.com/claudebuttons"></script>
```

### Step 2: Use the Web Components

```html
<!-- Claude Code button (copies CLI command to clipboard) -->
<claude-code-button
  command="/your-skill-name --flags"
  theme="branded"
  size="md"
></claude-code-button>

<!-- Cowork button (guides through skill setup) -->
<cowork-button
  command="/your-skill-name"
  skill-url="https://your-domain.com/skill-package.zip"
  theme="branded"
  size="md"
></cowork-button>
```

### Step 3: Customize

**Themes:** `branded` (default, terracotta), `dark`, `light`, `system` (auto-detects OS preference)

**Sizes:** `sm` (32px), `md` (40px, default), `lg` (48px)

**Popup:** Set `popup="false"` to skip the dialog and copy directly to clipboard.

## Framework Integration

The buttons are Web Components and work in any framework:

- **React:** `import { ClaudeCodeButton } from 'claudebuttons/react'`
- **Vue/Svelte/Angular:** Just `import 'claudebuttons'` and use the HTML tags
- **Vanilla HTML:** Load via `<script>` tag, no build step needed

## Button Behavior

### Claude Code Button
1. User clicks button
2. Popup shows the full `claude -p "/command"` command
3. User clicks "Copy" to copy to clipboard
4. User pastes into terminal

### Cowork Button
1. User clicks button
2. Popup shows setup instructions (with optional skill download)
3. User installs skill in Cowork via Customize → Skills
4. User runs the slash command in the Cowork task window

## Available Exports

```ts
// Web Components (auto-registered)
import 'claudebuttons';

// Programmatic API
import { createClaudeCodeButton, createCoworkButton, showPopup } from 'claudebuttons';

// React wrappers
import { ClaudeCodeButton, CoworkButton } from 'claudebuttons/react';

// Icons and theme utilities
import { CLAUDE_CODE_ICON, COWORK_ICON, themes, resolveTheme } from 'claudebuttons';
```

---
> Source: [cybertheory/claudebuttons](https://github.com/cybertheory/claudebuttons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
