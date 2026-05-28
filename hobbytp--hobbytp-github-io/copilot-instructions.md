## hobbytp-github-io

> - **NEVER** modify `layouts/_default/baseof.html` - PaperMod standard only

# Hugo Blog Architecture Rules - CRITICAL

## 🚨 ARCHITECTURE PROTECTION

### ✅ MANDATORY: Use PaperMod Standard Architecture
- **NEVER** modify `layouts/_default/baseof.html` - PaperMod standard only
- **NEVER** modify `layouts/_default/list.html` - PaperMod standard only
- **NEVER** run `scripts/toggle-spa-mode.sh` - DISABLED for stability
- **NEVER** add Tailwind CDN alongside PaperMod CSS

### ✅ CSS Rules
- ONLY modify `assets/css/custom.css` (keep under 500 lines)
- NEVER add external CSS via CDN
- Use CSS custom properties for theming
- NO unsupported CSS selectors like `:contains()`

### ✅ Navigation Rules
- Use Hugo menu system from `config.toml` only
- NO JavaScript SPA routing
- NO AJAX page loading with `innerHTML`

### ✅ Testing Requirements
- ALWAYS run `make build` before committing
- NEVER commit broken builds
- Test all page types: home, article, search, categories

## 🎯 APPROVED WORKFLOW
1. Read `ARCHITECTURE.md` first
2. Make changes following PaperMod patterns
3. Test with `make build`
4. Verify `make dev` works
5. Commit only working changes

## ❌ FORBIDDEN CHANGES
- SPA mode activation scripts
- Tailwind CDN integration
- Complex JavaScript routing
- Hugo core template modification
- Large custom CSS files

## 🔥 IF YOU BREAK ARCHITECTURE
1. Restore from git immediately
2. Read `docs/architecture-fixes.md`
3. Test thoroughly before re-committing

**THIS BLOG USES PAPERMOD STANDARD ARCHITECTURE - DO NOT CHANGE IT!**

---
> Source: [hobbytp/hobbytp.github.io](https://github.com/hobbytp/hobbytp.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
