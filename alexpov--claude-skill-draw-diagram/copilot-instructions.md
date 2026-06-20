## claude-skill-draw-diagram

> >


# Draw Diagram Skill

Generate production-quality editable diagrams in **.drawio** (Draw.io / diagrams.net) or **.excalidraw** (Excalidraw) format.

## Quick Start

1. **Determine format** — Default to `.drawio` unless user requests Excalidraw. Draw.io is preferred for cloud/infra diagrams (has official AWS, Azure, GCP, K8s icon libraries). Excalidraw is preferred for hand-drawn-style sketches, whiteboard-style brainstorming, or when the user explicitly asks.
2. **Read the appropriate reference** before generating:
   - For `.drawio` → Read `references/drawio-format.md`
   - For `.excalidraw` → Read `references/excalidraw-format.md`
3. **Search for icon shapes** — Run `python scripts/find_icon.py <term>` to find the correct shape style string for cloud provider icons.
4. **Review examples** — Check `references/examples/` for complete working diagrams similar to the request.
5. **Apply the design tests** — Before generating, verify your plan passes the Isomorphism Test and Education Test (see Design Philosophy below).
6. **Generate the file** following the format spec exactly. For 10+ components, build section-by-section (see Building Large Diagrams).
7. **Run the render-view-fix loop** — This is mandatory, not optional. Validate structure, render to PNG, audit, fix, repeat (see Step 5 in Workflow).
8. **Run the Pre-Delivery Checklist** — Every item must pass before delivering.
9. **Save** — If the user specifies a save location, use that. Otherwise, save to the workspace root `.diagrams/` folder (create it if it doesn't exist).
10. **Export (optional)** — If user wants PNG/SVG/PDF: `python scripts/export_drawio.py <file.drawio> --format png` (embeds editable diagram in the exported file).
11. **List diagrams** — If the user asks to "show diagrams", "list diagrams", or "what diagrams do we have", check the `.diagrams/` folder in the workspace root.

## Format Selection Guide

| Use Case | Recommended Format |
|---|---|
| Cloud architecture (AWS/Azure/GCP) | `.drawio` |
| Kubernetes / networking | `.drawio` |
| System design / microservices | `.drawio` |
| UML / ER diagrams | `.drawio` |
| Quick whiteboard sketch | `.excalidraw` |
| Hand-drawn aesthetic | `.excalidraw` |
| User says "excalidraw" | `.excalidraw` |
| User says "draw.io" / "drawio" | `.drawio` |

## Workflow

### Step 1: Understand Requirements
- What system/architecture to diagram?
- Which cloud provider(s)? (AWS, Azure, GCP, multi-cloud)
- Level of detail? (high-level overview vs detailed with subnets/ports)
- Any specific components to include?

### Step 2: Look Up Icons & Review Examples
- Run `python scripts/find_icon.py <term> --provider aws` to find correct shape strings
- Check `references/examples/` for a similar diagram to use as a structural starting point
- Plan layout on a mental grid before writing XML/JSON

### Step 3: Plan Layout
- Identify all components/nodes
- Determine groupings (VPCs, resource groups, namespaces, zones)
- Plan edge connections and data flow direction
- Choose left-to-right or top-to-bottom flow
- Place connected elements near each other to minimize edge crossings

### Step 4: Generate
- Read the appropriate reference file (`references/drawio-format.md` or `references/excalidraw-format.md`)
- Generate valid XML/JSON
- Use proper cloud provider icon shapes where available
- Use containers/groups for logical boundaries
- Label all edges with protocols/actions where relevant
- Apply consistent color coding

### Step 5: Render-View-Fix Loop (Required)

This is the most important step. AI-generated diagrams almost always have visual problems on the first pass — overlapping labels, edges through boxes, unbalanced spacing. You MUST validate your own output before delivering.

**Minimum (always do this):**
1. Run `python scripts/render_diagram.py <file> --validate-only` — catches structural errors (duplicate IDs, broken edge refs, missing parent cells).

**Full visual validation (required for Excalidraw, strongly recommended for Draw.io):**
1. **Render** — Run `python scripts/render_diagram.py <file>` to produce a PNG screenshot.
2. **View** — Read the PNG to actually see the rendered diagram.
3. **Audit against vision** — Compare the rendered result to what you designed in Steps 3-4. Ask:
   - Does the visual structure match the conceptual structure you planned?
   - Does the eye flow through the diagram in the intended order?
   - Are containers/groups clearly distinguishable from their children?
   - Are all labels readable and not overlapping neighboring elements?
4. **Check for defects** — Look for these specific problems:
   - Edges crossing through unrelated boxes
   - Labels rendered as literal HTML tags (missing `html=1;`)
   - Text overflowing its container
   - Overlapping elements or cramped spacing
   - Orphaned elements with no connections
5. **Fix** — Edit the file to resolve any issues found.
6. **Repeat** — Re-render and re-check until the diagram passes both the vision check and the defect check.

If Playwright is not available, fall back to structural validation + the Pre-Delivery Checklist (below).

### Step 6: Save & Deliver
- Save to user-specified location, or default to workspace `.diagrams/` folder
- Present to user with instructions on how to open

### Step 7: Export (Optional)
If the user wants an image/PDF:
- **PNG/SVG/PDF via draw.io desktop CLI**: `python scripts/export_drawio.py <file.drawio> --format png`
- Exported files use `--embed-diagram` by default — the PNG/SVG/PDF contains the full diagram XML embedded inside it. This means the exported file is both a viewable image AND an editable diagram. Users can open the `.drawio.png` in draw.io to recover and edit the full diagram. The intermediate `.drawio` source can be deleted after export since the exported file is self-contained.

### Building Large Diagrams (10+ components)

For comprehensive diagrams, you MUST build the file one section at a time. Do NOT attempt to generate the entire file in a single pass. This is a hard constraint — coding agents have token output limits (~32K for Claude Code), and a complex diagram easily exceeds that in one shot. Even when it doesn't, generating everything at once leads to worse layout quality.

1. Create the base file with the structural wrapper and the first section of elements (e.g., the container boundaries and tier 1 nodes).
2. Add one section per edit — each section gets its own dedicated pass.
3. After each section, run structural validation to catch errors early.
4. Run the full render-view-fix loop only after all sections are complete.

## Scripts Reference

| Script | Purpose | Usage |
|---|---|---|
| `scripts/find_icon.py` | Search AWS/Azure/GCP/K8s icon shape names | `python scripts/find_icon.py lambda --provider aws` |
| `scripts/render_diagram.py` | Validate structure + render to PNG | `python scripts/render_diagram.py arch.drawio --validate-only` |
| `scripts/export_drawio.py` | Export via draw.io desktop CLI | `python scripts/export_drawio.py arch.drawio --format png` |

### Render Pipeline Setup

The render script uses Playwright to open diagrams in a headless browser and screenshot them. This lets the agent see its own output and self-correct in the render-view-fix loop.

**Required for Excalidraw diagrams. Strongly recommended for Draw.io.**

```powershell
& .venv\Scripts\pip.exe install playwright
& .venv\Scripts\playwright.exe install chromium
```

If Playwright cannot be installed (e.g., CI environment, restricted sandbox), you MUST still run `--validate-only` mode and manually verify every item in the Pre-Delivery Checklist.

## Opening Instructions (include with delivery)

**Draw.io files (.drawio):**
- Open at [app.diagrams.net](https://app.diagrams.net)
- VS Code: Install "Draw.io Integration" extension
- Desktop: Download draw.io desktop app

**Excalidraw files (.excalidraw):**
- Open at [excalidraw.com](https://excalidraw.com) (drag & drop)
- VS Code: Install "Excalidraw" extension
- Obsidian: Native support with Excalidraw plugin

## Design Philosophy

Diagrams should **argue visually**, not just display information. A diagram is not formatted text — it is a visual argument that shows relationships, causality, and flow that words alone cannot express.

### The Isomorphism Test
Before generating, ask: **if you removed all text labels, would the structure alone communicate the concept?** Fan-outs should represent one-to-many relationships. Convergence should represent aggregation. Timelines should represent sequences. The shape of the diagram should mirror the shape of the concept. If every component is an identical box in a grid, the diagram is displaying, not arguing.

### The Education Test
Ask: **could someone learn something concrete from this diagram that they couldn't learn from a bullet list?** If the answer is no, the diagram isn't earning its complexity. Add spatial relationships, grouping, flow direction, or layering that reveals structure a list cannot.

### Applying This in Practice
- For multi-concept diagrams, each major concept should use a different visual pattern — not uniform cards.
- Before writing XML/JSON, mentally trace how the eye moves through the diagram. There should be a clear visual story with a beginning (entry point/user), middle (processing/logic), and end (output/storage).
- Use container nesting depth to show logical hierarchy — a service inside a subnet inside a VPC communicates three levels of scope at a glance.

## Layout Best Practices (Critical)

These rules prevent the most common AI-generated diagram problems: overlapping labels, edge crossings through boxes, and unreadable text.

### Spacing & Density
- **Prefer spacious layouts over compact** — clarity over density. Extra whitespace is always better than overlapping.
- **Size containers generously** — leave 50–80px margin around child elements inside swimlanes/groups. For containers with 3+ children stacked vertically, add 100px extra height beyond the sum of child heights.
- **Use a large page size** — `pageWidth="1600" pageHeight="1000"` or bigger. Don't try to fit everything in 1100x850. Increase `pageHeight` when vertical content grows (e.g., expanded containers push elements below the fold).
- **Space rows/columns 80-120px apart** — elements should never be closer than 60px.
- **Vertical gaps between siblings inside a container must be 75-80px minimum.** A common AI mistake is spacing children only 40-50px apart, which looks cramped and leaves no room for edge labels between them. Calculate: next child y >= previous child (y + height + 75).
- **When expanding a container, cascade the shift.** Increasing a container's height pushes everything below it — cost boxes, annotations, legends, edge waypoints. Update ALL downstream y-coordinates by the same delta. Also shift elements to the right if the container width grew (graph groups, annotations, legends).

### Labels & Text
- **Prefer rounded boxes with inline text over icon shapes with external labels.** Icon shapes (e.g., `mxgraph.azure.azure_functions`) are small (50x50) and place labels below the icon using `verticalLabelPosition=bottom`, which causes multi-line labels to overlap neighboring elements. Instead, use `rounded=1;` rectangles (170x70 or larger) with text inside.
- **Avoid HTML tags in labels** — `<b>`, `<i>`, `<font>` tags often render as literal text, especially in swimlane containers. Use `fontStyle=1` for bold, `fontStyle=2` for italic, and `&#xa;` for line breaks instead.
- **Always add `html=1;` to swimlane/container styles** — without this, even `&#xa;` newlines and any HTML in the container title won't render correctly.
- **Keep labels short** — 3–4 lines max inside a box. Move extra detail to a subtitle or separate text element.

### Edge Routing & Crossings
- **Plan element placement around edges first.** Before placing elements, identify which nodes connect to each other and arrange them so edges don't cross through unrelated boxes.
- **Place auxiliary elements near their targets.** If element A only connects to element B, put A adjacent to B — not on the opposite side of the diagram forcing the edge to cross other elements.
- **Use multi-row/staggered layouts** instead of one crowded row. Stagger elements vertically to create clear horizontal/vertical edge paths between rows.
- **Route edges through empty space.** Use explicit `<Array as="points">` waypoints to guide orthogonal edges around boxes when auto-routing fails.
- **Separate parallel edges by 15-20px** — when two edges go to the same region, offset them so labels don't overlap.
- **Opposite-side exit for diverging edges.** When two nodes in the same container both connect outward to different external targets in the same direction (e.g., both connect upward), route them from opposite sides — one exits left, one exits right. The auto-router treats each edge independently and will create X-crossings.
- **Pin entry/exit points on busy nodes.** When multiple edges arrive at or depart from the same box, use `entryX`/`entryY`/`exitX`/`exitY` (0-1 range) to assign each edge a distinct connection point. Don't rely on auto-snapping — it picks the nearest side per-edge, ignoring other edges.
- **Waypoints are mandatory for container-crossing edges.** Edges that cross swimlane/group boundaries must use explicit `<Array as="points">` waypoints. Auto-routing doesn't account for container headers or borders and often routes through them.
- **Edge labels need clear air.** When placing waypoints, ensure the mid-segment where the label renders falls in empty space — not overlapping a box, container header, or another label. Check by estimating the label center at the midpoint of the longest segment.

### Z-Ordering
- **Declare edges before shapes in XML** — mxGraph renders elements in document order. Edges declared first appear behind boxes, preventing visual clutter where lines draw over shapes.
- **Declare containers before their children** — parent mxCell elements must appear before child elements that reference them.

### Annotation Placement (Cloud Diagrams)

Cloud icon shapes (e.g., `mxgraph.aws4.*`) automatically place their label below the icon using `verticalLabelPosition=bottom`. If you add annotations or descriptions below as well, they WILL overlap the next row of elements.

**Rule 1: Main flow on a horizontal line.** Place the primary data path (request flow) on a single horizontal row at the same y-coordinate:
```
[Users] → [CloudFront] → [ALB] → [ECS] → [RDS]
  y=300      y=300        y=300    y=300    y=300
```

**Rule 2: Auxiliary services above or below.** Services that connect TO a main-flow element (not part of the primary path) go on a separate row above or below:
```
                [IAM]   [CloudWatch]          ← auxiliary row (y=150)
                  ↓         ↓
[Users] → [CloudFront] → [ALB] → [ECS] → [RDS]  ← main flow (y=300)
                                    ↓
                                  [SQS] → [Lambda] ← auxiliary row (y=450)
```

**Rule 3: Annotations go RIGHT of icons, not below.**
```
WRONG — annotation below icon overlaps next row:
  [CloudFront]
  "Cache Policies"     ← collides with [ALB] label below

RIGHT — annotation to the right:
  [CloudFront]  ← "Cache Policies"
```

**Rule 4: When using icon shapes with external labels**, leave 80-100px of vertical space between rows to accommodate the below-icon label plus clearance. When using rounded boxes with inline text, 60-80px between rows is sufficient.

## Pre-Delivery Checklist

Run through this checklist before delivering any diagram. Every item must pass.

### Structural (both formats)
- [ ] File parses without errors (valid XML for .drawio, valid JSON for .excalidraw)
- [ ] All element IDs are unique — no duplicates
- [ ] All edge/arrow source and target references point to existing elements
- [ ] No orphaned elements (every non-root element has a valid parent/container)

### Visual Layout (both formats)
- [ ] No elements overlap — shapes, labels, and containers all have clear boundaries
- [ ] Edges do not cross through unrelated boxes — reroute with waypoints if needed
- [ ] Parallel edges are offset by 15-20px so labels don't stack
- [ ] Edges entering/exiting the same node use distinct `entryX`/`entryY` or `exitX`/`exitY` values
- [ ] Edges crossing container boundaries have explicit waypoints
- [ ] Diverging edges from sibling nodes exit on opposite sides (no X-crossings)
- [ ] Edge labels don't overlap boxes or container headers
- [ ] Containers have 30px+ margin between their boundary and internal elements
- [ ] Spacing between rows/columns is 80px+ (never less than 60px)

### Labels & Text (.drawio specific)
- [ ] No raw HTML tags in labels — use `fontStyle=1` for bold, `&#xa;` for newlines
- [ ] All swimlane/container styles include `html=1;`
- [ ] Font sizes are readable — 11px minimum for any text
- [ ] Labels are 3-4 lines max inside any single box

### Cloud Icons (.drawio specific)
- [ ] Icon shapes use explicit width/height (50-60px) — not auto-sized
- [ ] AWS service names use official names and correct abbreviations
- [ ] Icons use latest version prefixes (`mxgraph.aws4.*`, `mxgraph.azure.*`, `mxgraph.gcp2.*`)
- [ ] Annotations are placed to the RIGHT of icon shapes, not below (see Annotation Placement)

### Z-Ordering (.drawio specific)
- [ ] Edges are declared before shapes in XML (renders behind boxes)
- [ ] Container elements are declared before their children

### Excalidraw Specific
- [ ] All bound text elements cross-reference their container (`boundElements` ↔ `containerId`)
- [ ] Arrow bindings are bidirectional (arrow refs shape AND shape refs arrow)
- [ ] Every element has a unique `seed` value
- [ ] `roughness`, `strokeWidth`, and `fillStyle` are consistent across similar element types

### Export (if applicable)
- [ ] Exported file uses `--embed-diagram` so the editable diagram is recoverable
- [ ] PNG/SVG renders correctly when opened (visually verified)

## Important Notes

- **Always use uncompressed XML** for .drawio files — never Base64/deflate encode
- **IDs must be unique** across the entire diagram
- **AI tends to resize cloud icons** — always use explicit width/height (typically 50x50 or 60x60 for icons)
- **Use containers** (parent relationships) to group services inside VPCs/regions/zones
- **Color-code consistently** — e.g., blue for compute, green for storage, orange for networking
- For cloud diagrams, prefer the `mxgraph.aws4.*`, `mxgraph.azure.*`, `mxgraph.gcp2.*` shape prefixes
- **Never use double hyphens (`--`) inside XML comments** — this is illegal per the XML spec and causes parse errors

## Copilot Compatibility Notes

This skill works with both Claude Code and GitHub Copilot (agent mode, CLI, and coding agent). Both read skills from `.github/skills/` and `.claude/skills/`. A few notes:

- **Copilot may not auto-trigger skills as aggressively** as Claude Code. If Copilot doesn't pick up the skill, invoke it explicitly with `/draw-diagram` in the chat.
- **Script execution**: Copilot agent mode in VS Code has terminal tool controls. The Python scripts require Python 3.10+ and optionally Playwright. If the agent can't run scripts, it should still follow the SKILL.md instructions and generate valid XML/JSON directly.
- **Skill name must be lowercase with hyphens** (no underscores, no uppercase). `draw-diagram` is compliant.

---
> Source: [alexpov/claude-skill-draw-diagram](https://github.com/alexpov/claude-skill-draw-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
