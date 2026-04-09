## agritechblog

> **Problem:** Auto-save can overwrite user's active edits when server data refetches after save operations.

# AgriTechBlog - AI Agent Development Rules

## Auto-Save and State Management

### Critical: Prevent Auto-Save Rollback
**Problem:** Auto-save can overwrite user's active edits when server data refetches after save operations.

**Solution Pattern (REQUIRED):**
Always use `isDirty` ref pattern in editor components:

```typescript
const isDirty = useRef(false);

// Mark dirty on ANY user input
const handleChange = (newValue) => {
  setValue(newValue);
  isDirty.current = true;
};

// Block updates when user is editing
useEffect(() => {
  if (isDirty.current) return; // Skip server updates
  if (serverData) setValue(serverData);
}, [serverData]);

// Reset after successful save
const handleSave = async () => {
  await saveToServer();
  isDirty.current = false;
};
```

**Applied in:**
- `/client/src/components/simple-markdown-editor.tsx` (all input fields)
- Any future editor components

### Critical: Markdown ↔ HTML Conversion
**Problem:** Tiptap editor outputs HTML but blog system stores Markdown. Without conversion, formatting is lost on reload.

**Solution Pattern (REQUIRED):**
All WYSIWYG editors MUST implement bidirectional conversion:

```typescript
import TurndownService from 'turndown';
import { marked } from 'marked';

// Initialize converters
const turndownService = new TurndownService({
  headingStyle: 'atx',
  codeBlockStyle: 'fenced',
  fence: '```',
});

marked.setOptions({
  gfm: true,
  breaks: true,
});

// On save: HTML → Markdown
onUpdate: ({ editor }) => {
  const html = editor.getHTML();
  const markdown = turndownService.turndown(html);
  onChange(markdown); // Save as markdown
};

// On load: Markdown → HTML
useEffect(() => {
  if (editor && content) {
    const html = marked(content) as string;
    editor.commands.setContent(html, false); // false = don't trigger onUpdate
  }
}, [content, editor]);
```

**Critical Rules:**
1. **Always save as Markdown** - Database stores `# Heading` not `<h1>Heading</h1>`
2. **Convert on load** - Parse Markdown → HTML for editor display
3. **Convert on save** - Parse HTML → Markdown for storage
4. **Use `false` flag** - `editor.commands.setContent(html, false)` prevents infinite loops
5. **Never mix formats** - Content is EITHER Markdown (storage) OR HTML (display)

**Applied in:**
- `/client/src/components/notion-editor.tsx`

### Editor Component Rules

1. **Never overwrite local state** from server after user starts typing
2. **Reset dirty flag** only after confirmed successful save
3. **Apply to ALL inputs:** title, content, excerpt, tags, images
4. **Auto-save timing:** 3s debounce + 30s interval (already configured)
5. **Preserve format on reload:** Use Markdown ↔ HTML converters
6. **Dependencies:** `turndown` (HTML→MD), `marked` (MD→HTML)

## React State Management Best Practices

### useEffect Dependencies
- Empty `[]` = run once on mount
- `[data]` = run when data changes
- Always check dirty flag before applying server updates

### useRef vs useState
- `useRef`: For values that don't trigger re-renders (flags, timers)
- `useState`: For values that update the UI

## File Organization

### Key Editor Files
- **Main editor:** `/client/src/components/simple-markdown-editor.tsx`
- **Notion editor component:** `/client/src/components/notion-editor.tsx`
- **Page controller:** `/client/src/pages/create-post.tsx`
- **Auto-save logic:** In editor component (handleAutoSave)
- **Editor styling:** `/client/src/components/notion-editor.css`

### Editor Features
- **Notion-like WYSIWYG editor:** Uses Tiptap for real-time formatting as you type
- **Single unified view:** No split panes - see formatted content directly
- **Markdown storage:** Converts HTML ↔ Markdown automatically
  - On save: HTML → Markdown (stored in database)
  - On load: Markdown → HTML (displayed formatted)
6. **Don't save HTML in database** - Always convert to Markdown before storing
7. **Don't load Markdown directly into Tiptap** - Convert to HTML first
8. **Don't trigger onUpdate when loading** - Use `setContent(html, false)` flag
9. **Don't install packages without --legacy-peer-deps** - Tiptap has peer dep conflicts
10. **Don't skip format conversion** - Every WYSIWYG editor needs both converters

## Package Installation Rules

**Always use `--legacy-peer-deps` flag:**
```bash
npm install <package> --legacy-peer-deps
```

**Why:** Tiptap packages have peer dependency conflicts with React versions. Using the flag prevents installation failures.

**Required packages for Notion-like editor:**
- `@tiptap/react` - Core editor
- `@tiptap/starter-kit` - Basic extensions
- `@tiptap/extension-placeholder` - Placeholder text
- `@tiptap/extension-image` - Image support
- `@tiptap/extension-link` - Link support
- `turndown` - HTML to Markdown converter
- `marked` - Markdown to HTML parser
- `react-is` - Required by recharts (analytics)
  - Uses `turndown` for HTML→MD and `marked` for MD→HTML
- **Markdown shortcuts work instantly:**
  - Type `#` + space → Heading 1
  - Type `##` + space → Heading 2
  - Type `###` + space → Heading 3
  - Type `**bold**` → **bold** text
  - Type `*italic*` → *italic* text
  - Type `-` + space → Bullet list
  - Type `1.` + space → Numbered list
  - Type ``` for code blocks
- **Auto-save protection:** isDirty flag prevents content rollback
- **Keyboard shortcuts:** `Cmd+Enter` to save (Mac) / `Ctrl+Enter` (Windows)
- **Format preservation:** Content stays formatted after auto-save and reload

### When Modifying Editors
1. Check for isDirty pattern
2. Test auto-save doesn't overwrite edits
3. Test manual save resets dirty flag
4. Verify server refetch doesn't cause rollback

## Testing Checklist for Editor Changes

- [ ] Type content, wait 3 seconds, verify no rollback
- [ ] Type content, manual save, verify saves correctly
- [ ] Edit existing post, verify loads correctly
- [ ] Switch between posts, verify no data mixing
- [ ] Check browser console for save/load debug logs

## Architect`ure Patterns

### Component State Flow
```
User Input → Mark Dirty → Auto-save (debounced) → Reset Dirty
     ↓
Server Data → Check Dirty → Skip if Dirty → Apply if Clean
```

### When to Use Each Hook
- `useState`: UI state (content, title, save status)
- `useRef`: Non-rendered state (isDirty, timers, last saved values)
- `useEffect`: Side effects (auto-save, data sync, cleanup)
- `useCallback`: Memoized functions (auto-save, data getters)

## Common Pitfalls to Avoid

1. **Don't sync server data unconditionally** - Always check isDirty first
2. **Don't forget to reset isDirty** - Reset after both auto-save and manual save
3. **Don't use useState for isDirty** - Use useRef to avoid unnecessary re-renders
4. **Don't create race conditions** - Block saves when already saving
5. **Don't lose user work** - isDirty protection is critical

## Debug Tips

### Debug Flow Visualizer
- **Hidden by default** to avoid clutter during development
- **Toggle with:** `Ctrl+Shift+D` (on admin/editor pages only)
- **Manual toggle:** Click the "🔍 Show Debug" button (bottom-right corner)
- **Preference persists** in localStorage across sessions

### Check These When Editor Issues Occur
1. Is `isDirty.current` being set on all input changes?
2. Is `isDirty.current` being reset after successful saves?
3. Are useEffect dependencies correct?
4. Check browser console for debug logs
5. Verify auto-save timing (3s debounce, 30s interval)

### Common Symptoms and Causes
- **Content disappears after typing:** Missing isDirty check in useEffect
- **Auto-save not triggering:** Check debounce timing and hasContentChanged
- **Changes not saving:** Verify mutation is working and not blocked
- **Old content appearing:** Server refetch overwriting without isDirty protection

## Image Upload (Cloudinary)

### Architecture
- **Server:** `server/config/cloudinary.config.ts` — Cloudinary SDK configuration
- **API endpoint:** `POST /api/admin/upload-image` — multer + Cloudinary streaming upload
- **API endpoint:** `GET /api/admin/cloudinary-status` — check if Cloudinary is configured
- **Component:** `client/src/components/image-upload.tsx` — drag-and-drop + URL paste fallback

### Environment Variables Required
```
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

### Rules
1. Images are uploaded via `FormData` (no JSON body)
2. Server uses `multer.memoryStorage()` — files never touch disk
3. Max file size: 10 MB
4. Allowed types: JPEG, PNG, GIF, WebP, SVG
5. All uploads go to `agritech-blog/` folder in Cloudinary
6. If Cloudinary is not configured, endpoint returns 503 and the component falls back to URL-only mode

## Editor Whitespace & Backspace Bug

### Problem
Tiptap's default behavior can cause backspace to delete both a character AND adjacent space (e.g., `hello world` → backspace after 'o' → `hellworld`).

### Root Cause
- HTML contenteditable converts consecutive spaces to `&nbsp;` 
- Mismatched Tiptap extension versions (Typography v3 with core v2)
- Default `white-space` CSS doesn't preserve spaces properly

### Fixes Applied
1. **Removed `@tiptap/extension-typography`** — v3.19.0 was incompatible with v2.14.0 core
2. **Added `white-space: pre-wrap`** in `notion-editor.css` — preserves all spaces
3. **Configured `hardBreak: false`** in StarterKit — prevents hard breaks from interfering
4. **Added `preformattedCode: true`** in TurndownService — better Markdown conversion

### Testing Checklist
- Type `hello world` → backspace after 'o' → should delete only 'o', not the space
- Multiple consecutive spaces should be preserved
- Spaces in bold/italic text should work correctly
- Enter key should create new paragraphs (not `<br>` tags)

## Automated Error Fixing

### Common Issues & Auto-Fixes

#### 1. Git Lock File Error
**Symptom:**
```
fatal: Unable to create '.git/index.lock': File exists.
Another git process seems to be running in this repository
```

**Auto-fix:**
```bash
npm run fix:git
```

**Manual fix:**
```bash
rm -f .git/index.lock .git/*.lock
```

**Root cause:** Git process was interrupted (Ctrl+C during commit, VS Code still open, system crash)

**Prevention:** The deploy script now auto-removes lock files before committing.

---

#### 2. CSS Syntax Warning During Build
**Symptom:**
```
[WARNING] Expected identifier but found "-" [css-syntax-error]
<stdin>:4381:2:
  4381 │   -: |\s;
```

**Status:** ✅ **Safe to ignore**

**Explanation:** Vite's CSS minifier occasionally misinterprets valid Tailwind utilities. This warning doesn't affect functionality or production builds.

---

#### 3. TypeScript Compilation Errors
**Check:**
```bash
npm run check
```

**Common fixes:**
- Missing imports → Add import statement
- Type mismatch → Update type annotations
- Unused variables → Remove or prefix with `_`

---

#### 4. Node Modules Corruption
**Auto-fix:**
```bash
npm run fix:deps
```

This deletes `node_modules`, `package-lock.json`, and reinstalls with `--legacy-peer-deps`.

---

#### 5. Port Already in Use
**Auto-fix:**
```bash
npm run predev
```

Kills processes on ports 3000, 5000, and 5173.

---

#### 6. GitHub Repository Rule Violations
**Symptom:**
```
! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs
```

**Cause:** Branch protection rules prevent direct pushes to `main`.

**Solution 1 - Feature Branch (Recommended):**
```bash
npm run fix:github
# Choose option 1, then create a Pull Request
```

**Solution 2 - Disable Branch Protection:**
```bash
npm run fix:github
# Choose option 2, follow steps to disable rules
```

**Solution 3 - Manual Feature Branch:**
```bash
git checkout -b feature/my-updates
git push origin feature/my-updates
# Then create PR on GitHub
```

**Solution 4 - Force Push (Repo Owner Only):**
```bash
git push origin main --force
# WARNING: Overwrites remote history!
```

**Check Rules:**
- Settings: https://github.com/asamountain/AgriTechBlog/settings/branches
- Rulesets: https://github.com/asamountain/AgriTechBlog/settings/rules

---

### Quick Commands

| Problem | Command |
|---------|---------|
| Git locked | `npm run fix:git` |
| Dependencies broken | `npm run fix:deps` |
| Build failing | `npm run fix:build` |
| Everything broken | `npm run fix:all` |
| Pre-deployment check | `npm run auto-fix` |
| Port conflicts | `npm run predev` |
| GitHub rules blocked | `npm run fix:github` |

### Auto-Fix Script

**What `npm run auto-fix` does:**
1. ✓ Removes Git lock files
2. ✓ Runs Git garbage collection
3. ✓ Verifies node_modules integrity
4. ✓ Checks TypeScript compilation
5. ✓ Validates CSS (informational)
6. ✓ Cleans build artifacts
7. ✓ Reports disk space
8. ✓ Scans for common issues

### For AI Agents

**CRITICAL RULES:**
1. ✅ Run `npm run auto-fix` before any deployment
2. ✅ Use `npm install --legacy-peer-deps` (ALWAYS)
3. ✅ Git errors → Auto-fix first, manual second
4. ✅ CSS build warnings → Ignore (Tailwind/Vite quirks)
5. ✅ TypeScript errors → Must fix before deployment
6. ✅ Test locally before deploying (`npm run dev`)

**NEVER:**
❌ Use `npm install` without `--legacy-peer-deps`
❌ Ignore TypeScript errors during deployment
❌ Delete `node_modules` without reinstalling
❌ **Commit API keys, tokens, passwords, or secrets**
❌ **Use hardcoded fallback credentials in code**

## Secret Management & Security

### CRITICAL: Never Commit Secrets

**Prohibited patterns:**
- API keys: `sk-`, `gsk_`, `AIza`, `pk_`, `Bearer`, etc.
- Tokens: OAuth tokens, JWTs, session secrets
- Passwords: Database credentials, admin passwords
- Private keys: SSH keys, certificates, encryption keys

**Required practices:**
1. ✅ **Always use environment variables** for secrets
2. ✅ **Fail-fast if required secrets are missing** (no fallback defaults)
3. ✅ **Document required env vars in `.env.example`**
4. ✅ **Keep `.env` in `.gitignore`**
5. ✅ **Rotate immediately if a secret is exposed**

### Example: Secure Configuration Pattern

**❌ WRONG - Hardcoded fallback:**
```javascript
const apiKey = process.env.API_KEY || 'sk-prod-abc123...';
```

**✅ CORRECT - Required from environment:**
```javascript
if (!process.env.API_KEY) {
  throw new Error('API_KEY environment variable is required');
}
const apiKey = process.env.API_KEY;
```

### If You Accidentally Commit a Secret

**Immediate actions:**
1. **Revoke the exposed secret** in the provider's console
2. **Generate a new secret** and store securely
3. **Remove from code:** Use required environment variables
4. **Purge from Git history:**
   ```bash
   npm run fix:secrets
   ```
5. **Force push cleaned history:**
   ```bash
   git push origin main --force
   ```
6. **Notify collaborators** to re-clone repository
7. **Enable GitHub Secret Scanning:**
   https://github.com/asamountain/AgriTechBlog/settings/security_analysis

### GitHub Push Protection

If you see:
```
remote: - GITHUB PUSH PROTECTION
remote:   Push cannot contain secrets
```

**Fix:**
1. Remove the secret from your code
2. Run: `npm run fix:secrets` (interactive cleanup)
3. Revoke the exposed secret in the provider console
4. Generate and use a new secret
5. Force push the cleaned history

**Files to check:**
- [test-polisher.js](test-polisher.js) - Groq API key
- `.env` files (should never be committed)
- Config files with credentials
- Scripts with hardcoded tokens

### Prevention: Pre-Commit Hook

Create `.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Scan for common secret patterns
if git diff --cached | grep -E '(sk-|gsk_|AIza|pk_test|pk_live|Bearer [A-Za-z0-9])'; then
    echo "❌ Potential secret detected in staged files!"
    echo "Remove secrets before committing."
    exit 1
fi
```

Make it executable: `chmod +x .git/hooks/pre-commit`

### Required Environment Variables

All secrets must be in `.env` (never committed):

```bash
# Required for production
MONGODB_URI=mongodb+srv://...
SESSION_SECRET=random-string-at-least-32-chars
POLISHER_API_KEY=gsk_...          # Groq
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...

# Optional but recommended
NOTION_TOKEN=secret_...
GOOGLE_CLIENT_SECRET=...
GITHUB_CLIENT_SECRET=...
```

See [env.example](env.example) for complete list.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asamountain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/asamountain)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
