## protocol-pro

> Each component should be in its own file, with a clear and descriptive name.

markdown
# Your rule content

Each component should be in its own file, with a clear and descriptive name.
Components should be functional and use hooks for state and side effects.
Use StyleSheet for styling components in React Native. Each component should have its own StyleSheet object defined within its file.
Organize components by feature. For example, all components related to the user profile should be in a 'profile' folder.
Keep state local to components unless it needs to be shared across multiple components. Use props to pass data down and callbacks to pass data up.
Components should not directly access or modify the state or DOM of other components. Use props and callbacks for communication.
When creating a new component, generate a new file named after the component, use a functional component with hooks, and include a stylesheet module for styling.
When creating a new component, also generate a test file with basic tests for the component's functionality.
When writing component code, use props for configurable values instead of hardcoding them.
Use Prettier to format the code. Run Prettier before committing changes.
Components should handle errors gracefully. Use try-catch blocks for asynchronous operations and display user-friendly error messages.
Ensure components are accessible. Use semantic HTML elements, provide alt text for images, and make sure the app is navigable with a keyboard.
Design components to be reusable across different parts of the app. Avoid tying components to specific contexts or data sources.
Favor composition over inheritance when building components. Use child components and props to customize behavior.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kumarpriyanshu09)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kumarpriyanshu09)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
