## mem-rebuild-pl

> Authoritative visual DNA and implementation specs for the Digital Twilight design system.

# Digital Twilight Design System Codemap

Authoritative visual DNA and implementation specs for the Digital Twilight design system.

## Visual DNA (The Twilight Spectrum)

- **Primary Noir**: `bg-[#020617]` (Slate 950) - The foundation of every view.
- **Accents**:
  - **Electric Cyan**: `#00F2FF` - Primary interaction and status color.
  - **Signal Orange**: `#FFA500` - Secondary warnings and warm CTAs.
- **Glassmorphism**: High-intensity blurring with minimal opacity for maximum depth.

## Typography Protocol

- **Headlines (H1-H6)**:
  - **Fonts**: `Space Grotesk`, `Clash Display`.
  - **Weight**: `900` (Black/Heavy).
  - **Styling**: `tracking-tighter` (-0.05em), `leading-[0.9]`.
  - **CSS**: `font-family: "Space Grotesk", "Clash Display", sans-serif !important; font-weight: 900 !important;`
- **Technical Metadata**:
  - **Font**: `Geist Mono`.
  - **Styling**: `uppercase`, `tracking-[0.2em]`, `text-[10px]`.
- **Body Content**:
  - **Font**: `Geist Sans`.
  - **Styling**: `text-white/70`, `leading-relaxed`.

## Component Primitives

### The Unified Card Token

Every container/card MUST implement this specific glassmorphic stack:

- **Tailwind**: `bg-white/5 backdrop-blur-xl border border-white/10 rounded-2xl`
- **Hover Enhancement**: `hover:border-white/20 hover:bg-white/10 transition-all duration-300`

### Interaction Layer

- **Cursor**: Always use `InvertedCursor` at the root.
- **Progress**: Always use `ScrollProgressIndicator` at the root.

## Implementation Enforcement
1. **Strict Noir**: No standard grey backgrounds. Use slate-950 or pure black.
2. **Backdrop Intensity**: Standard `backdrop-blur` is insufficient; always use `backdrop-blur-xl` or higher.
3. **Border Fidelity**: Borders must be semi-transparent white (`border-white/10`) to catch light without creating harsh edges.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darlingmarketing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
