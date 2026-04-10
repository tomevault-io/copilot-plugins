## ncssm-hu4010

> This repository contains HTML presentations for the **Applications in Entrepreneurship** course at NCSSM (North Carolina School of Science and Mathematics). The course is based on Bill Aulet's Disciplined Entrepreneurship framework.

# Claude Instructions for NCSSM Entrepreneurship Course

## Project Overview

This repository contains HTML presentations for the **Applications in Entrepreneurship** course at NCSSM (North Carolina School of Science and Mathematics). The course is based on Bill Aulet's Disciplined Entrepreneurship framework.

**Delivery Context:** Presentations are shown on ~80" TVs in classrooms AND streamed over Zoom to the Morganton campus. All design decisions must account for both viewing contexts.

---

## Critical Specification Files

### 1. `master-presentation-spec.md` (READ FIRST)

**Purpose:** The master design specification that governs ALL presentations.

**When to read:**
- Before creating or modifying ANY presentation
- When making styling decisions
- When implementing D3 visualizations
- When doing code review

**Key sections:**
- **Color Palette** - NCSSM brand colors, theme colors, CSS variables
- **Typography** - Font families, type scale, font loading
- **Minimum Font Sizes** - CRITICAL for classroom readability (see below)
- **Layout System** - Slide types and their purposes
- **Shared Resources** - How to use shared/styles.css and shared/presentation.js
- **Keyboard Navigation** - Standard shortcuts (T, A, S, F, 0-9, arrows)
- **DE Framework Theme Colors** - Color coding for the 6 entrepreneurship themes

### 2. `SYLLABUS.md`

**Purpose:** Course schedule, learning objectives, and class topics.

**When to read:**
- To understand the overall course structure
- To see which DE steps map to which classes
- For context about student deliverables

### 3. Class-Specific Specs (`class-N-topic/presentation-spec-*.md`)

**Purpose:** Detailed specifications for each individual presentation.

**When to read:**
- Before modifying a specific class presentation
- To understand the intended slide content and flow
- To see which D3 visualizations are expected

### 4. Lesson Plans (`class-N-topic/lesson-plan-*.md`)

**Purpose:** Teaching notes and activity instructions for instructors.

**When to read:**
- To understand the pedagogical intent
- For context about in-class exercises
- When the presentation content seems unclear

---

## Minimum Font Size Requirements (CRITICAL)

**These presentations must be readable from the back of a classroom AND over Zoom.**

| Element Type | Minimum Size | Notes |
|--------------|--------------|-------|
| Titles/Headlines | `2.5rem` (40px) | Slide titles, section headers |
| Primary Content | `1.5rem` (24px) | Main text, bullet points |
| Secondary Content | `1.25rem` (20px) | Supporting text |
| Labels & Captions | `1.25rem` (20px) | **Chart labels, D3 text** |
| Absolute Minimum | `1rem` (16px) | Only non-essential metadata |

### Code Review Rules

```javascript
// REJECT - Too small for classroom
.attr('font-size', '12px')
.attr('font-size', '14px')
.attr('font-size', '0.8rem')

// ACCEPT - Classroom readable
.attr('font-size', '1rem')      // Minimum
.attr('font-size', '1.25rem')   // Preferred for labels
.attr('font-size', '1.5rem')    // Body text
```

**Always use `rem` units, never small `px` values for D3 visualizations.**

---

## Shared Resources

### `shared/styles.css`

Common CSS for all presentations including:
- Slide type layouts (`.slide-title`, `.slide-statement`, `.slide-content`, `.slide-visual`, `.slide-quote`, `.slide-list`, `.slide-two-part`, `.slide-comparison`)
- TOC and Appendix overlay styles
- Progress bar and navigation UI
- Animation keyframes
- Responsive breakpoints

**All slides use flexbox centering. Content should be centered both vertically and horizontally with balanced whitespace.**

### `shared/presentation.js`

Navigation and interaction logic including:
- Keyboard navigation (arrows, T for TOC, A for Appendix, S for Syllabus, F for fullscreen)
- TOC overlay (auto-generated from slide data attributes)
- Appendix overlay navigation
- Slide animations via `animateSlide()` function
- Hash navigation for direct linking to slides

**Custom D3 visualizations should override `window.Presentation.goToSlide` to trigger drawing on slide entry.**

---

## Slide Types and When to Use Them

| Slide Type | Class | Use For |
|------------|-------|---------|
| `slide-title` | Opening slide | Course/class title, branding |
| `slide-statement` | Key message | Single powerful statement, quotes |
| `slide-content` | Bullet points | Lists, agenda items, key points |
| `slide-visual` | Visualizations | D3 charts, diagrams, grids |
| `slide-quote` | Attributed quotes | Expert quotes with attribution |
| `slide-list` | Lists with wrapper | Lists inside a `.slide-content` div |
| `slide-two-part` | Primary + secondary | Definitions, explanations |
| `slide-comparison` | Two columns | Before/after, pros/cons |

---

## D3 Visualization Guidelines

1. **Always read `master-presentation-spec.md`** before creating visualizations
2. **Use theme colors** from the spec (CSS custom properties or hex values)
3. **Font sizes must be `1.25rem` minimum** for all labels and text
4. **Use `rem` units** instead of `px` for scalability
5. **Make visualizations fully responsive** (see Responsive D3 section in master spec)
6. **Override `goToSlide`** to trigger visualization drawing:

```javascript
const originalGoToSlide = window.Presentation.goToSlide;
window.Presentation.goToSlide = function(n) {
    originalGoToSlide(n);
    if (n === 10) setTimeout(drawMyVisualization, 300);
};
```

### Responsive D3 Pattern (CRITICAL)

All D3 visualizations must be responsive - no hardcoded pixel values for positions or sizes:

```javascript
// GOOD - Responsive pattern
const width = 600;
const height = 400;
const scale = Math.min(width, height);

const svg = d3.select(container)
    .append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet');

// Positions as proportions
const center = { x: width * 0.5, y: height * 0.5 };
const nodeRadius = scale * 0.08;  // 8% of scale

// BAD - Hardcoded pixels
const center = { x: 300, y: 200 };
const nodeRadius = 50;
```

See `master-presentation-spec.md` for complete responsive D3 guidelines.

---

## File Structure

```
managua/
├── CLAUDE.md                    # This file
├── master-presentation-spec.md  # Master design spec (READ FIRST)
├── SYLLABUS.md                  # Course schedule
├── index.html                   # Course homepage/syllabus
├── shared/
│   ├── styles.css              # Common CSS
│   └── presentation.js         # Navigation logic
└── class-{n}-{topic}/
    ├── presentation.html       # The actual presentation
    ├── presentation-spec-*.md  # Class-specific spec
    └── lesson-plan-*.md        # Teaching notes
```

---

## Common Tasks

### Modifying a Presentation

1. Read `master-presentation-spec.md` for design guidelines
2. Read the class-specific `presentation-spec-*.md`
3. Make changes following the minimum font size rules
4. Test that content is centered and readable
5. Verify D3 visualizations use `1.25rem`+ font sizes

### Adding a New Visualization

1. Check `master-presentation-spec.md` for color palette and animation timings
2. Use theme colors from CSS variables or the documented hex values
3. Set all text to `1.25rem` minimum
4. Use the `goToSlide` override pattern to trigger drawing
5. Test on an actual large display if possible

### Code Review Checklist

- [ ] No font sizes below `1rem` (16px)
- [ ] D3 labels use `1.25rem` or larger
- [ ] Using `rem` units, not small `px` values
- [ ] Slides are centered (flexbox layout working)
- [ ] Colors match the theme palette
- [ ] Keyboard navigation works (T, A, S, arrows)

---

## DE Framework Theme Colors

The course follows Bill Aulet's 6 themes, color-coded as:

| Theme | Question | Color | Variable |
|-------|----------|-------|----------|
| 1 | Who is your customer? | NCSSM Blue `#356093` | `--theme-1-customer` |
| 2 | What can you do for them? | Purple `#7c3aed` | `--theme-2-value` |
| 3 | How do they acquire? | Emerald `#059669` | `--theme-3-acquire` |
| 4 | How do you make money? | Gold `#d4a028` | `--theme-4-money` |
| 5 | How do you design & build? | Red `#dc2626` | `--theme-5-build` |
| 6 | How do you scale? | Cyan `#0891b2` | `--theme-6-scale` |

Each class presentation uses its theme color as `--theme-color` for accents.

---

## RPIV Curriculum Development System

This project includes a complete RPIV (Research-Plan-Implement-Verify/Validate) skills system for systematic curriculum development.

### Quick Start

```bash
/artifact-status     # Check what exists and what's missing
/rpiv-workflow       # Get guided workflow recommendations
```

### Available Skills

#### Meta-Skills
| Skill | Purpose |
|-------|---------|
| `/skill-builder` | Create new Claude Code skills |
| `/skill-architect` | Design skill sets for workflows |

#### Course-Level Skills
| Skill | Purpose | Output |
|-------|---------|--------|
| `/course-init` | Scaffold new course project | Project structure |
| `/course-research` | Research pedagogical approaches | `curriculum-design-research.md` |
| `/course-plan` | Define teaching philosophy | `course-plan.md` |
| `/syllabus-build` | Create session schedule | `SYLLABUS.md` |
| `/master-spec` | Establish design system | `master-presentation-spec.md` |

#### Class-Level Skills (Per-Topic RPIV Loop)
| Skill | Purpose | Gate |
|-------|---------|------|
| `/class-research {topic}` | Research topic deeply | - |
| `/lesson-plan {topic}` | Plan instruction | Gate 1 |
| `/presentation-spec {topic}` | Specify slides | Gate 2 |
| `/presentation-build {topic}` | Build presentation | Gate 3 |
| `/presentation-review {topic}` | Validate & verify | Gates 4-5 |
| `/narration-build {topic}` | Generate voice narration | - |

#### Support Skills
| Skill | Purpose |
|-------|---------|
| `/artifact-status` | Check artifact completeness |
| `/gate-check {scope}` | Run validation gates |

### Quality Gates

| Gate | Question | On Fail |
|------|----------|---------|
| Gate 1 | Objectives align with course plan? | Fix lesson plan |
| Gate 2 | Spec achieves objectives? | Fix presentation spec |
| Gate 3 | Implementation matches spec? | Fix code |
| Gate 4 | Built the thing right? (Verification) | Fix CODE |
| Gate 5 | Built the right thing? (Validation) | Fix SPEC first |

### Workflow Order

```
Course Foundation:
  course-init → course-research → course-plan → syllabus-build → master-spec

Per-Class Loop:
  class-research → lesson-plan (G1) → presentation-spec (G2) →
  presentation-build (G3) → presentation-review (G4-5) → narration-build
```

### Configuration

RPIV settings are in `.claude/rpiv-config.json`. Key settings:
- Minimum font sizes
- Required research sources
- Gate strictness
- Narration provider

### Skill Files Location

All skills are in `.claude/skills/{skill-name}/`:
- `SKILL.md` - Main skill instructions
- `templates/` - Output templates
- `gate-N-checklist.md` - Gate checklists (where applicable)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caryden)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/caryden)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
