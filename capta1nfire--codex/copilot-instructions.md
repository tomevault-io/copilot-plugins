## codex

> Please refer to CLAUDE.md for comprehensive project guidelines and workflows.

# CODEX Project Cursor Rules

## 🎯 Primary Context
Please refer to CLAUDE.md for comprehensive project guidelines and workflows.
This file provides minimal Cursor-specific adaptations to avoid duplication.

## 🚀 Quick Start
- **Start all services**: `./pm2-start.sh`
- **Frontend**: http://localhost:3000
- **Backend**: http://localhost:3004
- **Rust Generator**: http://localhost:3002

## 📁 Essential Files to Read
1. `CLAUDE.md` - Complete development guide with workflows
2. `.nav.md` - Quick navigation and file paths
3. `CONTEXT_SUMMARY.md` - Current project state
4. `docs/CODEX_DESIGN_SYSTEM.md` - UI/UX guidelines

## 🛠️ Project Standards
- **Framework**: Next.js 14 (App Router) + Express + Rust
- **Language**: TypeScript with strict mode
- **Styling**: Tailwind CSS + Design System v2.0
- **Components**: Functional React components only
- **State**: React hooks and Context API
- **Process Manager**: PM2 (never use npm run dev directly)
- **Testing**: Jest + React Testing Library

## 💻 Code Style
```typescript
// ✅ Good
export const Component: React.FC<Props> = ({ prop }) => {
  const [state, setState] = useState<Type>(initial);
  return <div className="p-4 bg-blue-600">Content</div>;
};

// ❌ Avoid
class Component extends React.Component { }
const component = (props: any) => { }
```

## 🎨 Design System Tokens
- **Primary**: Blue-600 to Blue-700
- **Text**: Slate color scale
- **Spacing**: 4px base (p-1 = 4px)
- **Shadows**: shadow-sm, shadow-md
- **Animations**: transition-all duration-200

## 🚨 Critical Rules
- **NEVER** modify CODEX.md without permission
- **NEVER** use npm run dev (use PM2 instead)
- **ALWAYS** follow existing patterns
- **ALWAYS** run tests before committing
- **ALWAYS** use TypeScript strict types

## 📍 Common Tasks

### Generate Barcode
```bash
# QR Code (v2)
curl -X POST http://localhost:3004/api/v2/qr \
  -H "Content-Type: application/json" \
  -d '{"data": "test", "options": {"size": 400}}'
```

### Add New Feature
1. Check `.nav.md` for file locations
2. Follow existing patterns
3. Write tests first (TDD)
4. Update documentation

### Debug Issues
1. Check PM2 logs: `pm2 logs`
2. Verify services: `pm2 status`
3. Test endpoints with curl
4. Check `docs/TROUBLESHOOTING.md`

## 🔗 File References
@CLAUDE.md - Complete AI agent guide with all workflows
@.nav.md - Project navigation shortcuts
@CONTEXT_SUMMARY.md - Project state and transfer guide
@docs/CODEX_DESIGN_SYSTEM.md - Full design system documentation
@docs/TROUBLESHOOTING.md - Common issues and solutions

## 📝 When Using Cursor AI
1. This file provides quick context for Cursor
2. For detailed workflows, refer to CLAUDE.md
3. For navigation, use .nav.md
4. Follow the Think-Plan-Execute workflow from CLAUDE.md
5. Use visual iteration for UI changes

## ⚡ FOCUS Methodology (MANDATORY)
- **80% Code, 20% Documentation** - Write code, not endless docs
- **NO new .md files** without explicit permission
- **Update existing docs only**: CHANGELOG.md, TROUBLESHOOTING.md
- **Before creating any file**: Can this go in existing docs?
- **Every 30 min**: Run `./scripts/validate-focus.sh`

## ⚠️ Important Notes
- This is a companion file to CLAUDE.md
- Avoid duplicating content between files
- Update this file only for Cursor-specific needs
- Primary documentation lives in CLAUDE.md

---
*Last updated: June 14, 2025 - Created to complement CLAUDE.md for Cursor IDE*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/capta1nfire) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
