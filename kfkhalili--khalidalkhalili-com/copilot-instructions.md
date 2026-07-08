## digital-garden-project

> Digital Garden project structure, component slots, and embeddable widgets


# Digital Garden Project Context

This repo is an **Eleventy** site used with the [Obsidian Digital Garden](https://github.com/oleeskild/obsidian-digital-garden) plugin. Note content is published from Obsidian into `src/site/notes/`; do not edit those files (see rule **do-not-edit-plugin-managed**).

## Custom components (two patterns)

**1. Slot components (every page or by namespace)**  
Path: `src/site/_includes/components/user/<namespace>/<slot>/`  
- Namespaces: `index` (home only), `notes` (note pages only), `common` (all pages).  
- Slots: `head`, `header`, `beforeContent`, `afterContent`, `footer`.  
- Add a `.njk` file in the right slot folder (e.g. `common/afterContent/my-widget.njk`); it is included automatically. Dynamics are built in `_data/dynamics.js` from this directory tree.

**2. Embeddable widgets (only where the author puts them in a note)**  
To add a widget that appears only where the user writes a code fence in a note:  
- Put the widget HTML/JS in `src/site/_includes/snippets/<name>.njk`.  
- In `.eleventy.js`: add a **fence rule** (in the markdown-it `fence` handler) that returns a placeholder, e.g. `<div data-dg-embed="<name>"></div>` when the fence info is your widget name.  
- In `.eleventy.js`: add a **transform** that finds `[data-dg-embed="<name>"]`, reads the snippet file, and sets `placeholder.innerHTML = snippetHtml`.  
- In Obsidian the user writes ` ```<name>``` ` in the note; the transform injects the snippet at that spot.

Example: the Technical Debt Simulator is `snippets/tech-debt-sim.njk`, fence `tech-debt-sim`, transform `embed-tech-debt-sim`.

## Build and layout

- Input: `src/site`. Output: `dist`. Notes use layout `layouts/note.njk` (or `index.njk` for garden entry).  
- To change how note content is rendered, edit layouts, includes, or filters/transforms in `.eleventy.js`; never edit files under `src/site/notes/`.

---
> Source: [kfkhalili/khalidalkhalili.com](https://github.com/kfkhalili/khalidalkhalili.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
