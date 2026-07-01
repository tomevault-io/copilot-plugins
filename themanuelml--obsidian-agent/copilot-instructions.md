## obsidian-agent

> This is an Obsidian plugin that aims to create an agent sidebar that interacts with your vault. The goal is to connect multiple tools to your vault and be able to use the tools with any model you want.

# Description
This is an Obsidian plugin that aims to create an agent sidebar that interacts with your vault. The goal is to connect multiple tools to your vault and be able to use the tools with any model you want. 

## Stack 
- Typescript
- React
- CSS

## Rules
All styles must be classes defined in `styles.css`. For styles, use obsidian built-in [CSS variables](https://docs.obsidian.md/Reference/CSS+variables/CSS+variables).

All interfaces and types must be declared in the `types/` directory. Component props should also be declared in that folder.

- Before and after any change review the plugin guidelines.
- Do not use any deprecated object.
- Lean more to interfaces rather than types.
- When developing, think long-term for scalability and maintainability.
- Add comments for future developers, English only.

All errors should be catch, if it is needed you will need to show the a clean message to the user through an Obsidian `Notice`.

## Resources
- [Official sample plugin repository](https://github.com/obsidianmd/obsidian-sample-plugin)
- [Obsidian developer documentation](https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- [Specific plugin guidelines](https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines)
- [Reference langchain JS docs](https://reference.langchain.com/javascript/langchain)

---
> Source: [TheManuelML/obsidian-agent](https://github.com/TheManuelML/obsidian-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
