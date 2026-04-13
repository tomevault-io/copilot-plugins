## obsidian-simple-icons

> There are two other pre-existing plugins that have implemented most of the features this plugin is intended to. When building new features or debugging existing features you should review the code bases of https://github.com/gfxholo/iconic and https://github.com/FlorianWoelki/obsidian-iconize for potential solutions

There are two other pre-existing plugins that have implemented most of the features this plugin is intended to. When building new features or debugging existing features you should review the code bases of https://github.com/gfxholo/iconic and https://github.com/FlorianWoelki/obsidian-iconize for potential solutions

**Best Practices**

- Update the README.md with all feature updates and improvements so that it matches the current state and functionality of the plugin. Provide clear documentation of how the feature should be configured and how it will impact the UI
- Components should avoid overlapping purpose and should strive to be single purpose
- Files should ideally be no more than 300 lines. As files grow close to or beyond this length it should be discussed how to best decompose it into smaller files
- Classes and methods should include jsdocs describing their purpose and expected behavior

**Styling**

- limit the use of css as much as possible to better match the default look and feel of the Obsidian UI
- always use Obsidian's built in CSS variables whenever possible
- do not apply inline css apply classes and write styles to a styles.css file in the same directory of the component

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamJpRowan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
