## elsamultiskillagent

> Behavioral guidelines to reduce common LLM coding mistakes.

# CLAUDE.md — Chalea Clawskill Agent
# Template Family: OpenClaw Skill Catalog
# Generated for: N's project pipeline

---

## BLOCK 1 — Behavioral Contract (Karpathy)

Behavioral guidelines to reduce common LLM coding mistakes.
**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First
**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

## BLOCK 2 — Project Identity

**Project name:** Chalea Clawskill Agent
**Domain:** chaleaclawskill.site
**Derived from:** elsamultiskillagent (leluhur)
**Zauth target:** ≥ 70
**Stack:** Vite + React + Tailwind + TypeScript + Supabase + Radix UI

---

## BLOCK 3 — UI Spec Reference

**UI reference project:** Arwen (arwen-intelligence-k1)
**Theme directive:** Clean, white-dominant, developer aesthetic — bento + glass (light & sharp)

### Arwen UI Spec (extracted)

**Color palette:**
- Page background: `#f8fafc` (html base), `transparent` (body/main)
- Foreground / text: `#09090b` (near-black)
- Card background: `rgba(255,255,255,0.7)` default → `rgba(255,255,255,0.85)` hover
- Border: `rgba(0,0,0,0.06)` default → `rgba(0,0,0,0.12)` hover
- Primary accent (sky): `#0ea5e9` (sky-500) — used as subtle glow only, 8% opacity
- Warm accent: `#d4a574` — halftone decoration, dividers, hover borders
- Muted text: `#71717a`

**Typography:**
- Sans: `IBM Plex Sans` (var `--font-ibm-plex-sans`)
- Mono: `JetBrains Mono` (var `--font-jetbrains-mono`)
- Letter spacing: `-0.01em` global body
- Buttons: uppercase, `letter-spacing: 0.05em`, `font-weight: 700`

**Border radius (sharp system):**
- Buttons: `2px`
- Cards: `4px`
- Larger containers: `6–8px`
- No rounded-full anywhere

**Card / glass system:**
- `.bento-card`: white glass, `backdrop-filter: blur(20px)`, subtle shadow
- Hover: translateY(-1px), shadow lift
- `.glass-nav`: `blur(16px)`, white 60% opacity

**Background decorations:**
- Hero: `radial-gradient` sky glow at center, 8% opacity
- Dot grid: `18px` spacing, black 7% or warm amber 15%
- SVG grid lines: black, 3% opacity, `32×32` pattern
- Halftone divider: amber gradient line

**Page structure (from page.tsx):**
- `ParticleField` — animated dot matrix, full page background (z-index base)
- `Nav` — glass nav, sticky
- `Hero` — main above-fold section
- `Showcase` — scroll-triggered, viewport margin 200px
- `Install` — scroll-triggered
- `Footer`
- All sections: `motion.div` with `hidden → visible` variants (`opacity 0→1`, `y 30→0`, `duration 0.8`)

**Animation system:**
- Page load: `opacity 0 → 1`, `translateY(4px → 0)`, `blur(4px → 0)`, `0.4s cubic-bezier(0.16,1,0.3,1)`
- Scroll sections: `y: 30 → 0`, `duration 0.8s`, `easeOut`
- Card hover: `0.2s cubic-bezier(0.2,0,0,1)`
- Button transition: `0.15s ease`

**Component patterns:**
- All sections loaded via `dynamic()` with `ssr: false`
- Scrollbar: 4px width, transparent track, black 10% thumb

---

**Mascot placement (K standard pattern):**
- Hero section: 2-column layout
- Column 1: CTA (headline + subtext + button)
- Column 2: mascot image
- Mascot file: public\mascot.jpeg

**Mascot → accent color rule:**
- Dominant color dari mascot image → jadikan accent color project
- Jika bertentangan dengan white theme: gunakan sebagai accent only (hover, border, glow)
- White base tetap dominan, mascot color sebagai karakter

---

## BLOCK 4 — Path Constraints (CRITICAL)

**DO NOT scan, read, or iterate over files under:**
```
/public/skills/
```

This directory contains 1,708+ skill folders. It is static data — treat as a black box.
Reference skills only via `/public/skills.json` at root level.

**skills.json structure (reference only):**
```json
{
  "id": 1606,
  "name": "gcal-pro",
  "slug": "gcal-pro",
  "description": "...",
  "category": "Calendar & Scheduling",
  "author": "...",
  "github_url": "https://github.com/.../public/skills/...",
  "install_command": "npx clawhub@latest install gcal-pro",
  "featured": false,
  "popular": false
}
```

If a task requires skill content, fetch via `github_url` only when explicitly instructed. Never bulk-read the skills directory.

---

## BLOCK 5 — Scope Lock

**Full requirements:** see `REQUIREMENTS.md` at repo root (= ELSA_Upgrade_Summary.md yang K kirim)

**Core requirement (non-negotiable):**
Tambah skills dari `https://github.com/VoltAgent/awesome-openclaw-skills` ke dalam project.
Ini harus selesai — semua fitur lain adalah bonus.

**Priority order — work top to bottom, stop when time runs out:**

| Priority | Feature | Effort |
|----------|---------|--------|
| 0 | **Sync skills dari awesome-openclaw-skills** | **wajib** |
| 1 | One-click install command | < 1 hari |
| 2 | Requirements display | 1–2 hari |
| 3 | Skill preview pane | 2–3 hari |
| 4 | Security scan badges | 2–3 hari |
| 5 | Semantic / vector search | 3–5 hari |
| 6 | Bundle builder / loadout | 3–4 hari |
| 7 | Community ratings + reviews | 3–5 hari |
| — | Live ClawHub API sync | optional |
| — | Submit skill flow | optional |

**Rule:** jika 1 hari tidak cukup untuk semua, deliver sampai mana yang selesai clean. Jangan kirim fitur setengah jadi.

**Demo video scope:**
[TO FILL — konfirmasi ke K setelah dev selesai, bukan sebelum]

---

## BLOCK 6 — Zauth Divergence Strategy

Goal: hit ≥ 70. Previous project (Arwen) hit 72 — use same approach.

Divergence tactics to apply:
- Custom hooks with project-specific names (prefix: `useChalea`)
- Utility functions named to project context
- Component naming convention: `Chalea[ComponentName]`
- Do NOT copy component structure 1:1 from Elsa/Arwen without renaming + restructuring

Do NOT add dummy/disconnected code. All divergence must be logically coherent in project context.

---

## BLOCK 7 — Dev Checklist

Before starting any task, confirm:
- [x] Domain filled in BLOCK 2 → chaleaclawskill.site
- [x] Arwen UI spec filled in BLOCK 3
- [ ] Mascot file received and path noted in BLOCK 3
- [ ] `REQUIREMENTS.md` (= ELSA_Upgrade_Summary.md) copied to repo root
- [ ] `/public/skills/` path constraint understood

During dev:
- [ ] Do not scan `/public/skills/`
- [ ] Reference skills only via `skills.json`
- [ ] Apply Chalea-prefixed naming for custom hooks and components

---
> Source: [Demerzels-lab/elsamultiskillagent](https://github.com/Demerzels-lab/elsamultiskillagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
