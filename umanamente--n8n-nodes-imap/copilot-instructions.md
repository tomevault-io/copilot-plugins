## n8n-nodes-imap

> This file provides quick guidance for AI agents and GitHub Copilot working on the n8n-nodes-imap project.

# AI Agents & Copilot Guide

This file provides quick guidance for AI agents and GitHub Copilot working on the n8n-nodes-imap project.

## 📚 First, Read the Contributing Guide

**Before making any changes, always check `CONTRIBUTING.md` for complete setup instructions, project structure, and development guidelines.**

## 🎯 Quick Reference for AI Agents

### Project Structure
- **Node entry point**: `nodes/Imap/Imap.node.ts`
- **Operations**: `nodes/Imap/operations/{resource}/functions/`
- **Tests**: `test/UnitTests/`, `test/WithGreenmail/`, `test/WithImapflowMock/`
- **Credentials**: `credentials/ImapCredentials.credentials.ts`

### Key Commands
```bash
npm install          # Install dependencies
npm run build        # Build the project
npm test            # Run all tests
npm run test:watch  # Run tests in watch mode
```

### Adding New Operations
1. Create function in `operations/{resource}/functions/`
2. Register in `operations/{resource}/OperationsList.ts`
3. Add comprehensive tests
4. Update documentation if needed

### Commit Convention
Use Angular convention:
- `feat(scope): description` - New features
- `fix(scope): description` - Bug fixes
- `docs: description` - Documentation changes
- `test(scope): description` - Test additions/updates

### Important Notes
- ❌ **Never** update version numbers or `CHANGELOG.md` manually
- ✅ **Always** include tests for new functionality
- ✅ **Follow** existing code patterns and structure
- ✅ **Check** `CONTRIBUTING.md` for complete guidelines

### Resources
- Email operations: Handle email reading, searching, downloading
- Mailbox operations: Handle mailbox management, folder operations

---
> Source: [umanamente/n8n-nodes-imap](https://github.com/umanamente/n8n-nodes-imap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
