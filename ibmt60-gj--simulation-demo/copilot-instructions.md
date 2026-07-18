## simulation-demo

> This project is "Jing's 矩阵乐园", a personal interactive learning website for understanding matrix computation through spatial and geometric meaning.

# Agent Workflow Notes

## Project understanding

This project is "Jing's 矩阵乐园", a personal interactive learning website for understanding matrix computation through spatial and geometric meaning.

It is not only a source-code project. It also maintains ready-to-publish and portable deliverables:

- Source pages for the homepage and interactive dashboards.
- Cloudflare Pages output in `dist-cloudflare/`.
- Uploadable zip packages for Cloudflare.
- A portable folder and zip package that can be copied to another computer and opened locally.
- Deployment helper files for Wrangler and Cloudflare Pages.

The core product is a learning site with a homepage plus interactive dashboards about Moore-Penrose pseudoinverse, role vectors `r` and dual basis vectors `u`, and SVD-based geometric intuition for pseudoinverse.

The project scope also includes designing new interactive dashboards for additional and more complex matrix-related concepts, computations, and operations. Future boards may cover broader linear algebra, numerical linear algebra, matrix decompositions, optimization, projections, eigenspaces, conditioning, regularization, transformations, and applied matrix workflows.

When working in this repository, preserve this dual purpose:

1. Improve and maintain the learning website.
2. Design and add new matrix-concept dashboards when requested.
3. Keep the publishable and portable artifacts consistent when the user explicitly asks to package or publish.

When designing a new dashboard:

- Start from the mathematical concept and the user's learning goal, not from the UI layout.
- Explain the spatial or operational meaning before exposing formulas.
- Prefer interactive controls that reveal how matrix behavior changes under parameter changes.
- Connect computation steps to visual states so the user can see why the calculation works.
- Keep formulas, geometry, and procedural computation synchronized.
- Use stable, well-tested numerical methods or libraries when the concept requires nontrivial computation.
- Make each dashboard usable as a standalone learning object while still linking back to the homepage.

## Design and publishing workflow

When the user asks for homepage designs, visual options, drafts, prototypes, or alternatives, treat the work as an exploration stage.

- Do not modify `index.html` unless the user explicitly chooses a design and asks to apply it.
- Do not modify `deploy-cloudflare.sh`, `wrangler.toml`, or deployment configuration for unconfirmed drafts.
- Do not update `dist-cloudflare`, Cloudflare upload zips, or portable packages for unconfirmed drafts.
- Keep draft or preview files isolated from the publishing path.
- Only update deployment output or zip packages after the user explicitly says to publish, package, deploy, replace the homepage, or use the selected version.

Use these stages:

1. Draft: create lightweight preview material only.
2. Review: wait for the user to choose or reject the direction.
3. Apply: update formal site files only after confirmation.
4. Package or deploy: update `dist-cloudflare` and zip files only after explicit publishing or packaging confirmation.

Core rule: unconfirmed design options are previews, not release artifacts.

## Web design lessons

Before changing a page for visual reasons, diagnose the design problem first. Name whether the issue is layout balance, visual hierarchy, whitespace rhythm, component affordance, responsive behavior, or content density. Do not jump straight to changing padding, margins, or font sizes.

For homepage and dashboard design:

- Start from the information hierarchy: title, explanation, primary actions, secondary concepts, then supporting visuals.
- Keep layout relationships explicit. If two columns should align, decide whether the correct fix is equal height, shorter media, less content, or a different composition.
- Avoid fixing balance by stretching empty space. Prefer reducing the oversized side, changing the grid ratio, or simplifying content.
- Visual assets and interactive canvases should support the learning goal, not dominate the page unless the page concept explicitly calls for that.
- Buttons and board links should clearly look actionable. Use stable dimensions, clear labels, hover states, and enough spacing.
- Do not create nested card-heavy layouts. Use cards for repeated items or tools, and keep page sections clean.
- For Chinese text, check line length, wrapping, and weight. Large headings can easily overpower the rest of the page.
- Avoid one-off local tweaks that only solve the current screenshot. Prefer a layout rule that remains stable across common browser widths.

When offering multiple design options:

- Explain the design intent and tradeoffs before building.
- Keep options visually and structurally distinct enough to compare.
- Build previews only after the user wants to see them, and keep previews out of the release path.
- Do not package or publish options before the user chooses one.

Before calling a design task complete:

- Inspect the page visually when the environment allows it.
- Check at least desktop, laptop-width, and mobile-width layouts.
- Confirm that text does not overflow or collide with controls.
- Confirm that canvases or other visuals are not blank, distorted, or stealing focus from the main task.
- If browser preview is blocked by the environment, say so clearly and avoid pretending the layout was visually verified.

---
> Source: [ibmt60-gj/simulation-demo](https://github.com/ibmt60-gj/simulation-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
