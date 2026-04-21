## lulu-s-playground

> when implement demo pages in `src/pages/playground`


# Playground Structure

Group related demos under a project folder. Each project exports its demos via `index.ts`.

```
playground/
├── index.ts                      # Aggregate all project exports
├── ProjectName/
│   ├── index.ts                  # Export: { ProjectNamePages }
│   └── DemoName/
│       ├── index.tsx             # Page component (PageProps)
│       ├── styles.module.scss
│       ├── components/           # Follow component-structure.mdc
│       └── data/                 # Optional: mock data
└── AnotherProject/
    └── ...
```

**Key Points:**
- Project `index.ts` exports an object: `export const ProjectNamePages = { DemoPage1, DemoPage2 }`
- Top-level `index.ts` merges all projects: `export const PlaygroundPages = { ...Project1Pages, ...Project2Pages }`
- Components follow standard structure: `_component.tsx`, `_styles.module.scss`, `index.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WenchuanLiliZhao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
