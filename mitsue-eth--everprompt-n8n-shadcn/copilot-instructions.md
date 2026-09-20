## ui

> User Interface of Everprompt


User Interface Instructions — EverPrompt

Overview

The UI must be minimal, scalable, and DB-ready.
Default view: black canvas, centered Prompt Editor Card, and a right-edge Arc of labels (colored dots + tiny captions).

Clicking a label opens a Label Sheet side panel with prompts for that label. Selecting a prompt loads it into the editor.

All components accept data props and must be able to scale with any number of labels (including zero). In the future, data will be fetched from an external DB.

**Key Design Principles:**

- **Minimalism First**: Distraction-free prompt crafting experience
- **n8n Community Focus**: UI optimized for automation developers
- **Extensibility**: Plugin-ready component architecture
- **Performance**: Fast, responsive, and accessible
- **shadcn/ui Foundation**: Built on shadcn/ui components for consistency and customization

⸻

Screens & States

Start View
• Prompt Editor Card
• Large textarea with autosave + debounce.
• Top-right toolbar: Saved • Copy.
• Right Arc (Labels)
• SVG arc with colored dots.
• Each dot = one label.
• Tiny caption = first word of label name.
• Arc auto-scales to number of labels (3–12 typical).
• If labels > visibleCount → show “+ more” dot to open Label Sheet with all labels.

Label Sheet
• Opens as a right-side Sheet.
• Header: label name + colored bullet + prompt count.
• Search bar + sort options (Updated, Created, Title).
• List of prompts:
• Title (1 line).
• Preview (2 lines).
• Optional meta chips (future).
• Row click → load prompt into editor + close sheet.
• “All labels” and “Unlabeled” are valid label filters.

Empty States
• No labels → arc hidden, show hint “Add your first label”.
• Label has no prompts → Sheet shows “No prompts yet”.
• Unlabeled only → arc shows a single gray dot labeled “Unlabeled”.

⸻

Components

<PromptEditorCard />

Props:
• value: string
• saving: boolean
• onChange(value: string): void
• onCopy(): void

Implementation:
• Built on shadcn/ui Card component
• Uses shadcn/ui Textarea for content editing
• shadcn/ui Button components for toolbar actions
• Responsive design with shadcn/ui responsive utilities

<ArcLabels />

Props:
• labels: Array<{ id: string; name: string; color: string; lastUsedAt?: number }>
• visibleCount?: number
• onLabelClick(labelId: string): void
• geometry?: { radius?: number; arcDeg?: number; cx?: number; cy?: number }
Behavior:
• Sort labels by lastUsedAt desc, fallback name.
• Captions = first word of name.
• If too many labels, add “+ more” dot (labelId="**all**").

<LabelSheet />

Props:
• open: boolean
• label: { id: string; name: string; color?: string } | { id: "**all**", name: "All labels" }
• prompts: Array<PromptSummary>
• onOpenChange(open: boolean): void
• onSelectPrompt(promptId: string): void
• onSearch(term: string): void

Implementation:
• Built on shadcn/ui Sheet component
• Uses shadcn/ui Input for search functionality
• shadcn/ui Button components for actions
• shadcn/ui Badge components for labels
• Responsive design with shadcn/ui responsive utilities
PromptSummary:

{
id: string
title: string
preview: string
labelIds: string[]
updatedAt: number
createdAt: number
meta?: Record<string, any> // extensible
}

⸻

Event Flow 1. User clicks a dot in <ArcLabels />. 2. App sets LabelSheet.open = true and passes labelId. 3. Prompts for that label are loaded and displayed in <LabelSheet />. 4. User clicks a prompt row → prompt loads into <PromptEditorCard />, sheet closes.

⸻

Accessibility
• Each dot is a button with aria-label="Open label {name}".
• Sheet rows keyboard-navigable; Enter opens.
• Esc closes the sheet.

⸻

Responsiveness
• Desktop ≥1280px: arc full radius, captions always visible.
• Tablet 768–1279px: smaller arc, captions on hover/focus.
• Mobile <768px: show only a few dots + “+ more” dot → opens Sheet full screen.

⸻

Visual Style
• Background: pure black (#000).
• Arc stroke: white @ 7% opacity.
• Dot: label color, inner opaque + outer glow (15%).
• Caption: white @ 70% opacity, font size 10–11px.

⸻

Acceptance (for v0)
• Start View renders with editor + arc, no runtime errors.
• Arc adapts to labels[] length.
• Clicking label opens Label Sheet with mock prompts.
• Clicking prompt loads into editor + closes sheet.
• “+ more” dot opens All Labels view.
• Works with no labels or only Unlabeled.

⸻

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
