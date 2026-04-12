## workshop-component-library

> When generating components from Figma MCP:

# CLAUDE.md - Project Rules

## Component Generation Rules

### STRICT: Do NOT add extra elements
When generating components from Figma MCP:
- Generate ONLY the elements visible in the Figma screenshot
- Do NOT add placeholder content, demo data, or example usage beyond what Figma shows
- Do NOT add wrapper divs unless they exist in the Figma design
- Do NOT add icons, images, or decorative elements not in the original design
- If something is unclear, ASK instead of assuming

### Component Structure
- React components go in: `components/react/`
- HTML showcase pages go in: `showcase/`
- When adding a new React component, always update the barrel export in `components/react/index.ts`
- Each React component should be a single, focused file
- Export only the component itself, not demo/example code
- Props should match EXACTLY what Figma variants show
- Do not invent additional props or variants beyond what exists in Figma

### Naming Conventions
- Use PascalCase for React component files: `Button.tsx`, `IconButton.tsx`
- Match Figma component names when possible

### Design Tokens
- Reference `tokens.json` for the correct color values, spacing, and typography
- Use the exact hex values from tokens.json (e.g., `#4573D2` for blue-500, `#CE3F42` for red-500)
- Do NOT invent colors — if a color isn't in tokens.json, check the Figma design first

### When Using Figma MCP
1. Always fetch `get_design_context` or `get_screenshot` before implementing — never guess
2. Extract ONLY what's in the design
3. If the design shows text like "Text" or "Title", keep it exactly as shown
4. Do NOT add hover states, focus states, or animations unless they are explicitly shown in Figma variants OR specifically requested by the user
5. Do NOT add interactive behavior unless requested
6. Ask for clarification if the design is ambiguous

### Showcase Pages
- Showcase HTML pages use Tailwind CDN (`<script src="https://cdn.tailwindcss.com"></script>`)
- Use vanilla JavaScript for interactivity (no frameworks)
- Follow the existing section pattern:
  ```html
  <section class="bg-white border border-[#DDDDDD] rounded-[4px] p-6 mb-6">
    <h2 class="text-xs font-bold text-[#595959] uppercase tracking-wide mb-4">Component Name</h2>
    <!-- component content exactly as in Figma -->
  </section>
  ```
- Font: Helvetica, Arial, sans-serif
- Always verify output in the preview server before considering a task done

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cristinaiftode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cristinaiftode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
