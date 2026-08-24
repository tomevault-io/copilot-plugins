## website

> Follow these guidelines when creating or editing documentation pages.

# Website CLAUDE.md

## Writing Documentation

Follow these guidelines when creating or editing documentation pages.

### Core Philosophy

Action-Oriented: users visit docs to solve a problem, not to read.

1. **Result First**: Tell the user *what* they will achieve in the first sentence.
2. **No Fluff**: Minimize conceptual explanations.
3. **Declarative over Procedural**: Don't say "Step 1: Do this". Show the configuration and say "Apply this".

### Formatting Rules

#### Steps
Avoid numbered headers like `## 1. Step Name`.
- Use standard markdown headers (`##` or `###`).
- Group code and commands logically.

#### Code Blocks
All multi-line code blocks **MUST** include a filename attribute.

Correct:
````
```yaml {filename="config.yaml"}
apiVersion: v1
kind: Pod
```
````

#### Headings
- Titles should be verbs (e.g., "Run Tests" not "Tests").
- Keep hierarchy flat.
- No Prerequisites Sections: mention dependencies inline or assume the user has the basics (Cluster, CLI).

### Tone
- **Direct**: "Run this command" (Imperative).
- **Confident**: Avoid "please", "maybe", or "we recommend".

---
> Source: [kuberik/website](https://github.com/kuberik/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
