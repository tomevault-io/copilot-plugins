## claudegram

> **ALWAYS update `docs/index.html` when adding features, commands, or contributors.** This is a required step in every PR that changes functionality. Do not commit feature/command changes without the corresponding website update.

# Claudegram Development Guidelines

## Website Maintenance (MANDATORY)

**ALWAYS update `docs/index.html` when adding features, commands, or contributors.** This is a required step in every PR that changes functionality. Do not commit feature/command changes without the corresponding website update.

### New Features
Add a feature card to the features grid. Use the `data-category` attribute for tab filtering (valid values: `core`, `voice`, `media`, `session`):
```html
<div class="feature-card" data-category="core">
  <div class="feature-icon">[emoji]</div>
  <h3>Feature Name</h3>
  <p>Brief description of what the feature does.</p>
</div>
```

### New Commands
Add a command row to the appropriate category section in the commands grid:
```html
<div class="command-row">
  <code class="command-code">/command &lt;args&gt;</code>
  <span class="command-desc">Description of what the command does</span>
</div>
```

### New Contributors
When a new contributor makes significant contributions (check `git shortlog -sn`), add them to the contributors section:
```html
<a href="https://github.com/username" class="contributor-card" target="_blank">
  <img src="https://github.com/username.png" alt="username" class="contributor-avatar">
  <span class="contributor-name">Display Name</span>
  <span class="contributor-role">Contributor</span>
</a>
```

### Pre-commit Check
Before committing, verify that `docs/index.html` reflects ALL:
- Feature cards for every user-facing feature
- Command rows for every bot command in `src/bot/handlers/command.handler.ts`
- Contributor cards for every contributor with 3+ commits

## Code Style

- TypeScript for all source files
- Functional patterns preferred
- Use existing utilities from `src/utils/` (download, sanitize, file-type)
- Validate external input (URLs, file content) using existing helpers

## Security Checklist

Before committing changes that handle external input:
- [ ] URL protocol validation using `isValidProtocol()` from `src/utils/download.ts`
- [ ] Path sanitization using `sanitizePath()` from `src/utils/sanitize.ts`
- [ ] Error sanitization using `sanitizeError()` before logging
- [ ] File content validation using `isValidImageFile()` for images
- [ ] No tokens or secrets in process arguments (use stdin for curl)

---
> Source: [NachoSEO/claudegram](https://github.com/NachoSEO/claudegram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
