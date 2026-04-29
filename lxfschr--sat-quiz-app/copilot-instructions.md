## sat-quiz-app

> Writing react code

- Readability and Modularity are paramount!
- Keep components very small and focused
- Break large components into smaller reusable components
- It can be good to keep subcomponents in the main component file. You only need to create a new .TSX file if that component needs to be exported.
- .TSX files can only export a single component.
- Ensure that the top level component that is exported is at the top, with any subcomponents written below the component where they are used.
- Each component should only have 1 to 4 JSX tags in it. Any more and the JSX should be broken into small React Components.
- Any JSX tag with an event handler should be its own component
- Any JSX tag used in a ternary should be its own component.
- Only one hook is allowed per component, compose hooks into a single one before exposing to a component
- React components are only responsible for getting data out of their hook and using it to build and style the JSX
- Hooks contain all logic other than styling
- Use tanstack-query for all requests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lxfschr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
