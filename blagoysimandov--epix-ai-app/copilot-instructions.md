## epix-ai-app

> Epix AI is a health-genomics mobile application built as a PWA (Vite + React + TypeScript + Tailwind CSS) that translates a user's genetic and epigenetic data into personalised, real-time health intelligence across 9 core scenarios: a genomic risk report with biological age tracking and disease domain drill-downs (Cardio, Metabolic, Neuro, Onco); an aging velocity dashboard with time-series visualisation; a camera-based food and medication scanner that triggers pharmacogenomic and nutrigenomic alerts with safe alternatives; an activity dashboard that contextualises step deficits against cardiovascular genetic risk; a travel health advisory that cross-references destination environments with genetic predispositions; a blood test upload and AI interpretation flow with genetically-adjusted reference ranges and longitudinal biomarker tracking; a real-time workout screen with gene-based heart rate zone guidance and post-workout recovery estimation; an alcohol metabolism warning triggered on drink detection; and a 

# AI Assistant Instructions for epix-ai

## Project Overview
Epix AI is a health-genomics mobile application built as a PWA (Vite + React + TypeScript + Tailwind CSS) that translates a user's genetic and epigenetic data into personalised, real-time health intelligence across 9 core scenarios: a genomic risk report with biological age tracking and disease domain drill-downs (Cardio, Metabolic, Neuro, Onco); an aging velocity dashboard with time-series visualisation; a camera-based food and medication scanner that triggers pharmacogenomic and nutrigenomic alerts with safe alternatives; an activity dashboard that contextualises step deficits against cardiovascular genetic risk; a travel health advisory that cross-references destination environments with genetic predispositions; a blood test upload and AI interpretation flow with genetically-adjusted reference ranges and longitudinal biomarker tracking; a real-time workout screen with gene-based heart rate zone guidance and post-workout recovery estimation; an alcohol metabolism warning triggered on drink detection; and a sleep recovery dashboard linking circadian gene variants to aging speed — all screens use hardcoded mock data, a consistent design system (custom Tailwind tokens + CSS variables), a mobile-first 390px layout with a persistent bottom tab bar, and React Router v6 for navigation.

## Design System Workflow
When creating, updating, or modifying components within the `src/components/design-system` directory:
1. You MUST export the component from `src/components/design-system/index.ts`.
2. You MUST update the design system inventory route at `src/routes/design.tsx` to showcase the component and document its usage. This ensures `/design` remains a complete, live inventory of all available UI elements.
3. Use the `@design-system` alias when importing components from the design system.
4. You MUST update the "Design System Components" section in both `GEMINI.md` and `CLAUDE.md` with a short explanation of the new or modified component.

## Design System Components
The following components are part of the core design system:
- **Badge**: A small, color-coded pill (`green`, `yellow`, `red`, `default`, `outline`) used to display status, tags, or small notification counts.
- **StatusDot**: A small circular indicator (`green`, `yellow`, `red`, `default`) with a subtle pulse/glow effect used for inline risk or status indicators next to text.
- **BarChartOverview**: A pre-styled responsive Bar Chart widget displaying comparative metrics using the Recharts library and standard design tokens.
- **LineChartTrend**: A pre-styled responsive Line Chart widget displaying trends over time using the Recharts library and standard design tokens.
- **ColorCard**: A specialized presentation card used purely within the design system inventory route to display and preview theme color tokens and their hex/CSS variables.
- **HalfCircleGauge**: An enhanced dual-arc SVG semicircle gauge that visualises two concurrent 0–100 levels with gradient strokes and a center value callout. `primaryColor` drives the outer arc (Risk); `secondaryColor` (defaults to `var(--teal)`) drives the inner arc (Activation). Used for displaying genetic risk vs. activation levels on domain cards.
- **MetricChart**: A compact, single-metric area chart (using Recharts) for 7-day trend data. Accepts `data: Array<{ day: string; value: number }>`, a `color` CSS string, and an optional `unit` label. Used in physical activity and environment detail pages.
- **AgingTrajectoryChart**: A dark-navy trajectory chart for displaying biological age against a diagonal “Standard aging” baseline over a multi-week period. Accepts `data: TrajectoryPoint[]` (`{ week, bioAge, baseline }`). Automatically computes and displays the delta in the header, color-codes biological-age segments when they move above (rose) or below (teal) the standard line, and highlights the latest point. Used on the home screen.
- **DiseaseItem**: A rich collapsible card for displaying a condition's genetic propensity. Collapsed state shows a status-colored dot, disease name, genetic marker chip, and two mini horizontal progress bars for genetic risk and activation. Expands via smooth CSS `grid-template-rows` animation to reveal full description text, a `GeneticRiskChart`, an `ActivationGauge`, and a CTA button.
- **GeneticRiskChart**: An SVG horizontal gradient track (green→amber→rose) visualizing where the user's genetic risk falls in the population. Features a bell-curve population overlay, a dashed average marker, and a glowing diamond pin with "YOU" label at the user's percentile. Labels: Low / Average / High.
- **ActivationGauge**: A beautiful semi-circle gauge featuring a glowing arc, decorative tick ring, and a central value display to visualize epigenetic activation levels (0-100%).
- **PulsingAlertIcon**: A high-visibility warning icon featuring two concentric pulsing/fading rings and a glassmorphic center container. Used for genetic and health alerts.
- **GeneticProfileChip**: A glassmorphic data chip used to display genetic context, including Gene, SNP, and Genotype, with a DNA icon. Used in alert and report screens.
- **AlertBanner**: A slim, high-visibility warning bar used to surface urgent environmental or health alerts. Features a red/amber gradient and a pulsing icon.
- **BottomSheet**: A slide-up detailed information modal with a blurred background overlay and scrollable content area. Includes a handle for visual affordance and a header with a close button.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blagoySimandov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/blagoySimandov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
