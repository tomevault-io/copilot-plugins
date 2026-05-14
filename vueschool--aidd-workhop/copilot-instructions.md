## z-components

> * When creating a brand new stand alone component (that's not a child/tightly coupled component), use the `nlx hygen component new --name [ComponentNameHere]` from package.json*

* When creating a brand new stand alone component (that's not a child/tightly coupled component), use the `nlx hygen component new --name [ComponentNameHere]` from package.json* 
* ALWAYS read the created files after running the hygen command.
* When creating template refs always name them with the `El` suffix. For example:
  * `const textareaEl = useTemplateRef('textareaEl')`  
  * Always use `useTemplateRef('[ref identifier here]')` to define template refs (not `ref()`)
  * ShadCN has components like `Textarea` that seem like native components but are not. Remember to account for that.

---
> Source: [vueschool/aidd-workhop](https://github.com/vueschool/aidd-workhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
